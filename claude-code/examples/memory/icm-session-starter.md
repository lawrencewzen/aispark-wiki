> 📚 **AI Spark Wiki** · Claude Code 知识库

# ICM 会话启动器
> 在任意 Claude Code 会话开始时粘贴此内容以激活 ICM 上下文。
> 需要先安装并配置 ICM：`brew tap rtk-ai/tap && brew install icm`
> 然后执行 `icm init --mode mcp && icm init --mode hook && icm init --mode skill`

---

# 上下文 — 本会话已激活 ICM（无限上下文记忆）

ICM 已在本机安装并配置完毕。可用它跨会话存储和检索持久化记忆，突破上下文窗口限制。

## 可用的 MCP 工具

`icm` MCP 服务器正在运行。你可以访问 22 个 `icm_*` 工具，用于存储、召回和管理持久化记忆。

**直接使用 CLI**（通过 Bash 调用）：

```bash
# 存储一条记忆
icm store --topic "<project-slug>" --content "<fact>" --importance high|medium|low|critical

# 通过语义查询召回
icm recall "<natural language query>"

# 查看统计
icm stats          # 数量、主题、平均权重
icm topics         # 列出所有主题
icm list           # 列出最近的记忆

# 管理
icm forget <id>    # 按 ID 删除
icm decay          # 应用时间衰减
icm prune          # 删除低权重条目
```

**重要（v0.5.0 语法）**：
- `--importance` 是枚举类型：`critical / high / medium / low`——不是浮点数
- 没有 `memory` 子命令——直接使用 `icm store`、`icm recall`
- 永久知识图谱：`icm memoir`（独立层，不会衰减）

## 斜杠命令

- `/recall <query>` — 搜索 ICM 记忆
- `/remember <content>` — 在 ICM 中存储一条记忆

## 记忆工作原理

两个层次：
- **记忆**（情节性）：带时间戳的条目，基于重要性进行时间衰减。
  `critical` 重要性永不衰减。`low` 随时间淡出。
- **Memoir**（语义性）：带类型关系的永久知识图谱
  （`depends_on`、`contradicts`、`superseded_by`、`part_of` 及另外 5 种）。

搜索采用混合策略：BM25 全文检索（30%）+ 向量相似度（70%）。

`PostToolUse` 钩子会自动运行——每隔 N 次工具调用，ICM 自动提取上下文并存储，无需任何显式操作。

## 本会话的建议用法

```bash
# 会话开始时——召回相关上下文
icm recall "<current feature or topic>"

# 做出关键决策时
icm store --topic "<project>" --content "<decision and rationale>" --importance high

# 记录永久性架构事实
icm memoir add-concept -m "<project>" -n "<concept>"
```

## 数据库位置

`~/Library/Application Support/dev.icm.icm/memories.db`
