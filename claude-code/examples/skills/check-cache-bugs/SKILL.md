> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: check-cache-bugs
description: "审计 Claude Code 配置中的缓存漏洞（CC#40524）——哨兵字符串、--resume/--continue、归因头 + ArkNill B3/B4/B5"
effort: low
disable-model-invocation: true
---

# 检查缓存漏洞（CC#40524）

审计您的 Claude Code 配置，检查 2026 年 3–4 月发现的缓存及费用漏洞。

**耗时**：约 30 秒 | **范围**：版本、配置文件、CLAUDE.md、技能、钩子、Shell 配置文件、所有 claude 二进制文件

> **关于缓存污染的说明**：本技能会将包含 `cch=` 字符串的内容加载到当前会话的消息数组中。为获得最干净的结果，请在全新会话开始时运行此命令，或通过 `claude -p "$(cat .claude/commands/check-cache-bugs.md)"` 以单次打印模式调用。

**参考**：`anthropics/claude-code#40524` | **发现者**：`@jmarianski` + `@whiletrue0x` | **扩展者**：`@ArkNill`（ArkNill/claude-code-cache-analysis，2026 年 4 月）

---

## 背景

### 修复状态（截至 v2.1.92）

| 漏洞 | 影响版本 | 状态 |
|-----|------------------|--------|
| 漏洞 1 — cch 哨兵字符串（独立二进制文件） | v2.1.36–v2.1.90 | **已在 v2.1.91 修复** |
| 漏洞 2 — --resume 时的 deferred_tools_delta | v2.1.69–v2.1.89 | **已在 v2.1.90 修复** |
| 漏洞 3 — 归因头每会话哈希 | v2.1.69+ | **活跃**（环境变量临时解决方案） |
| B4 — 微压缩 / 静默上下文剥除 | 所有版本，直至 v2.1.92 | **活跃**（GrowthBook 服务端控制） |
| B5 — 工具结果预算上限 200K | 所有版本，直至 v2.1.92 | **活跃**（MCP 工具豁免） |

### 原始三个漏洞（CC#40524）

- **漏洞 1**（v2.1.91 已修复）：Bun 的原生 HTTP 层在 `JSON.stringify` 之后、TLS 加密之前，对
  `cch=00000` 证明占位符进行了等长字节替换。仅当 `messages[]` 内容中字面出现
  `cch=00000` 时才会触发。已确认关闭——npm 版本与独立二进制版本在 v2.1.91+ 上现已等效。

- **漏洞 2**（v2.1.90 已修复）：会话 JSONL 写入器在写入磁盘前会剥除 `deferred_tools_delta` 附件
  记录。在 `--resume` 时，这些记录缺失——延迟工具层没有先前历史，会从头重新声明所有工具，导致
  每条消息位置偏移，彻底破坏消息级缓存前缀。每次恢复会将 87–118K 个 token 重建为
  `cache_creation`。Anthropic 内部追踪编号为 inc-4747。

- **漏洞 3**（活跃，v2.1.69+）：Claude Code 将计费头作为系统提示词的第一块注入。该头包含一个
  3 字符的 SHA-256 哈希，由第一条用户消息的第 [4, 7, 20] 个字符 + CC 版本派生——每个会话、
  每个子智能体、每次附带查询均唯一。由于缓存基于前缀匹配，这导致每次调用时约 12K 系统提示词
  token 发生冷缺失。一个包含 5 个子智能体的会话 = 6 次冷缺失。实测数据：使用环境变量解决方案
  后，缓存命中率从 48% 提升至 99.98%。

### 额外漏洞（ArkNill，2026 年 4 月）

- **B4 — 微压缩**（所有版本）：三种机制在发送至 API 前静默将工具结果替换为
  `[Old tool result content cleared]`。由服务端 GrowthBook 标志控制，绕过 `DISABLE_AUTO_COMPACT`。
  `/export` 命令显示的是原始上下文，而非模型实际接收到的内容。在一次会话中测量到 327 次清除事件。

- **B5 — 工具结果预算上限**（所有版本）：`applyToolResultBudget()` 在聚合字符数超过 200K 时截断
  工具结果。内置工具（Read、Bash、Grep、Glob、Edit）均受影响。MCP 工具可通过
  `_meta["anthropic/maxResultSizeChars"]`（v2.1.91+）进行覆盖。

费用影响摘要：漏洞 3 单独影响下每次会话启动/子智能体费用增加 2–5 倍；包含 10+ 技能和 3–4 次
恢复的情况下，漏洞 2 每轮增加 10–20 倍（两项估算在各自场景下均准确）。

---

## 使用说明

您是一名审计员。按顺序运行所有阶段，收集每项结果，并生成最终报告。不得跳过阶段或提前停止。

---

### 阶段 1 — Claude Code 版本及安装方式

```bash
# 版本检查
claude --version

# 所有已安装的 claude 二进制文件
which -a claude 2>/dev/null

# 检查当前二进制文件是独立版还是 npm 版
file $(which claude) 2>/dev/null
ls -la $(which claude) 2>/dev/null
```

- 版本 < 2.1.90 → 标记为**漏洞 2 风险**（resume/DTD；v2.1.90 已修复）
- 版本 < 2.1.91 且为独立二进制文件 → 标记为**漏洞 1 机制**（哨兵字符串；v2.1.91 已修复）
- 版本 >= 2.1.69 → 标记为**漏洞 3 风险**（归因头；尚无修复——检查阶段 4）
- 二进制文件为 Mach-O / ELF 可执行文件 → 独立版本（与 v2.1.36–v2.1.90 的漏洞 1 相关）
- 二进制文件为指向 `node_modules` 的符号链接或包含 `cli.js` → npm/npx

---

### 阶段 2 — 哨兵字符串扫描（漏洞 1）

在所有静态配置文件中搜索字面字符串 `cch=`。排除 `.jsonl` 文件（临时对话历史）和本命令文件本身。

```bash
# 全局配置文件
grep -r "cch=" \
  ~/.claude/CLAUDE.md \
  ~/.claude/MEMORY.md \
  ~/.claude/TONE.md \
  ~/.claude/FLAGS.md \
  ~/.claude/RULES.md \
  ~/.claude/RTK.md \
  ~/.claude/ANTI_AI.md \
  2>/dev/null

# 全局技能、命令、智能体、钩子（排除本命令文件）
grep -rl "cch=" ~/.claude/skills/ 2>/dev/null
grep -rl "cch=" ~/.claude/commands/ --exclude="check-cache-bugs.md" 2>/dev/null
grep -rl "cch=" ~/.claude/agents/ 2>/dev/null
grep -rl "cch=" ~/.claude/hooks/ 2>/dev/null

# 项目级配置
grep -r "cch=" CLAUDE.md .claude/CLAUDE.md .claude/MEMORY.md 2>/dev/null
grep -rl "cch=" .claude/skills/ 2>/dev/null
grep -rl "cch=" .claude/commands/ --exclude="check-cache-bugs.md" 2>/dev/null
grep -rl "cch=" .claude/agents/ 2>/dev/null
grep -rl "cch=" .claude/hooks/ 2>/dev/null

# 更广泛扫描：跨项目所有 CLAUDE.md 文件
find ~ -name "CLAUDE.md" \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  2>/dev/null | xargs grep -l "cch=" 2>/dev/null
```

若在 `.jsonl` 文件以外发现匹配且版本 < v2.1.91，则标记为**漏洞 1 风险**。
v2.1.91+ 版本：哨兵机制已修复——无论配置内容如何，均无风险。

---

### 阶段 3 — 恢复/继续使用情况（漏洞 2）

```bash
# settings.json
grep -i -- "--resume\|--continue" ~/.claude/settings.json 2>/dev/null
grep -i -- "--resume\|--continue" .claude/settings.json 2>/dev/null

# 钩子
grep -rn -- "--resume\|--continue" ~/.claude/hooks/ 2>/dev/null
grep -rn -- "--resume\|--continue" .claude/hooks/ 2>/dev/null

# 命令和技能
grep -rn -- "--resume\|--continue" ~/.claude/commands/ 2>/dev/null
grep -rn -- "--resume\|--continue" .claude/commands/ 2>/dev/null

# Shell 配置文件（别名、函数）
grep -n -- "--resume\|--continue" ~/.zshrc ~/.bashrc ~/.bash_profile ~/.zprofile 2>/dev/null

# 项目脚本
find . -name "*.sh" -o -name "Makefile" 2>/dev/null | \
  xargs grep -l -- "--resume\|--continue" 2>/dev/null
```

- 版本 >= v2.1.90：漏洞 2 已**修复**——将命中结果标记为仅供参考（无实际费用风险）
- 版本 < v2.1.90 且钩子/设置中有命中 → 标记为**漏洞 2 自动化**（持续暴露）
- 版本 < v2.1.90 且命令/技能/脚本中有命中 → 标记为**漏洞 2 手动**（调用时暴露）

---

### 阶段 4 — 归因头检查（漏洞 3）

检查计费头环境变量是否已被禁用。

```bash
# 全局设置
grep -i "CLAUDE_CODE_ATTRIBUTION_HEADER\|ENABLE_TOOL_SEARCH" \
  ~/.claude/settings.json 2>/dev/null

# 项目设置
grep -i "CLAUDE_CODE_ATTRIBUTION_HEADER\|ENABLE_TOOL_SEARCH" \
  .claude/settings.json 2>/dev/null

# Shell 配置文件
grep -i "CLAUDE_CODE_ATTRIBUTION_HEADER" \
  ~/.zshrc ~/.bashrc ~/.bash_profile ~/.zprofile 2>/dev/null
```

- `CLAUDE_CODE_ATTRIBUTION_HEADER` 未设置为 `false` 且版本 >= 2.1.69 → 标记为**漏洞 3 活跃**
- 已设置为 `false` → **漏洞 3 已缓解**

---

### 阶段 5 — 多二进制文件检查

```bash
for b in $(which -a claude 2>/dev/null | sort -u); do
  echo "=== $b ==="
  $b --version 2>/dev/null || echo "unavailable"
  file $b 2>/dev/null
  ls -la $b 2>/dev/null
done
```

将所有版本 < v2.1.91 的独立二进制文件标记为漏洞 1 风险。
将所有版本 < v2.1.90 的二进制文件标记为漏洞 2 风险。
将所有版本 >= v2.1.69 的二进制文件标记为漏洞 3 风险（除非归因头环境变量已设置）。
记录可能被误调用的过期二进制文件。

---

## 输出格式

```
## Claude Code 缓存漏洞审计 — CC#40524

**日期**：[今天]
**当前 claude 版本**：[版本]
**安装方式**：[独立二进制文件 | npm/npx | 混合]

---

### 漏洞 1 — 哨兵字符串替换（独立二进制文件 v2.1.36–v2.1.90）
**状态**：[已修复 (v2.1.91+) / 安全 / 有风险 / 不适用]

**版本 >= v2.1.91**：[是 → 已修复 | 否 → 查看下方]
**机制活跃（独立二进制文件，v2.1.36–v2.1.90）**：[是 / 否]
**静态配置中发现触发哨兵字符串**：[是 — 位置 | 否]

[若已修复] 运行 v2.1.91+。哨兵机制已修补——npm 与独立版本等效。无需操作。
[若有风险 — v < 2.1.91 且独立版本且发现哨兵字符串]
→ 更新至 v2.1.91+：`npm install -g @anthropic-ai/claude-code@latest`
→ 临时方案：从标记的文件中移除 `cch=00000` 字符串，或改用 npm。
[若安全] 二进制文件中存在该机制，但静态配置中无触发字符串。仍建议更新至 v2.1.91。
[若不适用] npm/npx 安装——漏洞 1 机制不存在。

---

### 漏洞 2 — --resume / --continue 时的缓存前缀不匹配（v2.1.69–v2.1.89）
**状态**：[已修复 (v2.1.90+) / 有风险（自动化）/ 有风险（手动）/ 不适用]

**版本 >= v2.1.90**：[是 → 已修复 | 否 → 查看下方]
**发现自动化使用（钩子/设置）**：[是 — 位置 | 否]
**发现手动使用（命令/技能/脚本）**：[是 — 位置 | 否]

**根本原因**：JSONL 写入器在写入磁盘前剥除了 `deferred_tools_delta` 记录。在 `--resume` 时，
延迟工具层没有声明历史，会从头重新声明所有工具，导致每条消息位置偏移，彻底破坏消息级缓存前缀。
每次恢复将 87–118K 个 token 重建为 cache_creation。Anthropic 追踪编号 inc-4747。v2.1.90 已修复。

[若已修复] 运行 v2.1.90+。漏洞已解决。配置中发现的 `--resume` 使用在缓存层面现已安全。
[若有风险]
→ 更新至 v2.1.90+：`npm install -g @anthropic-ai/claude-code@latest`
→ 立即：在更新前避免使用 `--resume` 和 `--continue`。
[若不适用] 版本 < 2.1.69——不受影响。

---

### 漏洞 3 — 归因头每会话哈希（v2.1.69+，尚无修复）
**状态**：[活跃 / 已缓解 / 不适用]

**版本在受影响范围内（>= v2.1.69）**：[是 / 否]
**CLAUDE_CODE_ATTRIBUTION_HEADER 已设置为 false/0/no/off**：[是 → 已缓解 | 否 → 活跃]

[若活跃] 每次会话启动和每次子智能体调用都会错过系统提示词缓存（约 12K token 以
cache_creation 速率重建）。包含 5 个子智能体的会话 = 6 次冷缺失。
→ 修复方案（立即生效，无需重启）：添加到 ~/.claude/settings.json：
  {
    "env": {
      "CLAUDE_CODE_ATTRIBUTION_HEADER": "false"
    }
  }
→ 预期效果：缓存命中率 48% → ~99.98%（实测，来源：@whiletrue0x CC#40524）

[若已缓解] 头部已禁用。无需操作。
[若不适用] 版本 < 2.1.69——不受影响。

---

### B4/B5 — 上下文变异（仅供参考，所有版本）

这些漏洞由服务端 GrowthBook 标志控制，无法在客户端缓解。
无法进行配置检查——作为仅供参考信息报告。

**B4 — 微压缩**：三种机制在发送至 API 前静默将工具结果替换为 `[Old tool result content cleared]`。
`/export` 命令显示的是原始上下文，而非模型实际接收到的内容。
解决方案：定期开启新会话以重置工具结果池。

**B5 — 工具结果预算上限（聚合 200K 字符）**：内置工具（Read、Bash、Grep、Glob、Edit）
在每会话聚合工具结果超过 200K 字符时被截断。MCP 工具可通过
`_meta["anthropic/maxResultSizeChars"]`（v2.1.91+，最高 500K）部分覆盖。

---

### 多个二进制文件
[列出发现的每个二进制文件、版本、类型（独立/npm）及每个漏洞的状态]

---

### 摘要

| 漏洞 | 版本 | 状态 | 操作 |
|-----|----------|--------|--------|
| 漏洞 1 — 哨兵字符串（独立版本） | v2.1.36–v2.1.90 | [已修复 / 安全 / 有风险 / 不适用] | [更新至 v2.1.91+ 或"无"] |
| 漏洞 2 — --resume/--continue | v2.1.69–v2.1.89 | [已修复 / 有风险 / 不适用] | [更新至 v2.1.90+ 或"无"] |
| 漏洞 3 — 归因头 | v2.1.69+ | [活跃 / 已缓解 / 不适用] | [添加环境变量或"无"] |
| B4 — 微压缩 | 所有版本 | 仅供参考 | 定期开启新会话 |
| B5 — 工具结果预算 200K | 所有版本 | 仅供参考 | 大量读取使用 MCP 工具 |
| 过期二进制文件 | — | [干净 / 存在] | [移除或无] |

[若漏洞 3 活跃——始终显示此内容]
⚡ 快速解决方案：将 CLAUDE_CODE_ATTRIBUTION_HEADER=false 添加到 settings.json——立即生效，无需重启。

[若漏洞 1-3 均为安全/已缓解/已修复/不适用]
✅ 原始 CC#40524 漏洞无活跃缓存暴露。B4/B5 仅供参考——监控会话长度。

⚠️ 追踪：github.com/anthropics/claude-code/issues/40524 | github.com/ArkNill/claude-code-cache-analysis
```
