> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: rtk-optimizer
description: "对高冗余度的 shell 命令使用 RTK 包装，以减少 token 消耗。当运行 git log、git diff、cargo test、pytest 或其他输出冗长、浪费上下文窗口 token 的 CLI 命令时使用。"
allowed-tools: Bash
effort: low
metadata:
  version: 1.0.0
---

# RTK Optimizer Skill

**目的**：自动为高冗余度命令推荐 RTK 包装，以减少 token 消耗。

## 工作原理

1. **检测高冗余度命令**（来自用户请求）
2. **推荐 RTK 包装**（如适用）
3. **用户确认后执行 RTK 命令**
4. **追踪节省量**（跨会话）

## 支持的命令

### Git（节省 70% 以上）
- `git log` → `rtk git log`（节省 92.3%）
- `git status` → `rtk git status`（节省 76.0%）
- `find` → `rtk find`（节省 76.3%）

### 中等价值（节省 50-70%）
- `git diff` → `rtk git diff`（节省 55.9%）
- `cat <large-file>` → `rtk read <file>`（节省 62.5%）

### JS/TS 技术栈（节省 70-90%）
- `pnpm list` → `rtk pnpm list`（节省 82%）
- `pnpm test` / `vitest run` → `rtk vitest run`（节省 90%）

### Rust 工具链（节省 80-90%）
- `cargo test` → `rtk cargo test`（节省 90%）
- `cargo build` → `rtk cargo build`（节省 80%）
- `cargo clippy` → `rtk cargo clippy`（节省 80%）

### Python & Go（节省 90%）
- `pytest` → `rtk python pytest`（节省 90%）
- `go test` → `rtk go test`（节省 90%）

### GitHub CLI（节省 79-87%）
- `gh pr view` → `rtk gh pr view`（节省 87%）
- `gh pr checks` → `rtk gh pr checks`（节省 79%）

### 文件操作
- `ls` → `rtk ls`（压缩输出）
- `grep` → `rtk grep`（过滤输出）

## 激活示例

**用户**："帮我看看 git 历史记录"
**Skill**：检测到 `git log` → 推荐 `rtk git log` → 说明可节省 92.3% token

**用户**："找出所有 markdown 文件"
**Skill**：检测到 `find` → 推荐 `rtk find "*.md" .` → 说明可节省 76.3%

## 安装检查

首次使用前，验证 RTK 是否已安装：
```bash
rtk --version  # 应输出：rtk 0.16.0+
```

若未安装：
```bash
# Homebrew（macOS/Linux）
brew install rtk-ai/tap/rtk

# Cargo（全平台）
cargo install rtk
```

## 使用模式

```markdown
# 当用户请求高冗余度命令时：

1. 确认请求
2. 推荐 RTK 优化：
   "我将使用 `rtk git log` 减少约 92% 的 token 用量"
3. 执行 RTK 命令
4. 追踪节省量（可选）：
   "节省了约 13K token（基准：14K，RTK：1K）"
```

## 会话追踪

可选：追踪整个会话的累计节省量：

```bash
# 会话结束时
rtk gain  # 显示本次会话的总 token 节省量（基于 SQLite 存储）
```

## 边界情况

- **输出较小**（< 100 字符）：跳过 RTK（开销不值得）
- **已使用 Claude 工具**：Grep/Read 工具本身已经过优化
- **多条命令**：使用 RTK 包装一次批量处理，而非逐条包装

## 配置

通过 CLAUDE.md 启用：
```markdown
## Token 优化

对高冗余度命令使用 RTK（Rust Token Killer）：
- git 操作（log、status、diff）
- 包管理器（pnpm、npm）
- 构建工具（cargo、go）
- 测试框架（vitest、pytest）
- 文件查找与读取
```

## 实测数据（已验证）

基于真实场景测试：
- `git log`：13,994 字符 → 1,076 字符（节省 92.3%）
- `git status`：100 字符 → 24 字符（节省 76.0%）
- `find`：780 字符 → 185 字符（节省 76.3%）
- `git diff`：15,815 字符 → 6,982 字符（节省 55.9%）
- `read file`：163,587 字符 → 61,339 字符（节省 62.5%）

**平均节省 72.6% token**

## 局限性

- GitHub 上 446 颗星，持续维护（23 天内发布 30 个版本）
- 不适用于交互式命令
- 迭代节奏较快（注意破坏性变更）

## 使用建议

**推荐使用 RTK**：git 工作流、文件操作、测试框架、构建工具、包管理器
**跳过 RTK**：小输出、快速探索、交互式命令

## 参考资料

- RTK GitHub：https://github.com/rtk-ai/rtk
- RTK 官网：https://www.rtk-ai.app/
- 评估报告：`docs/resource-evaluations/rtk-evaluation.md`
- CLAUDE.md 模板：`examples/claude-md/rtk-optimized.md`
