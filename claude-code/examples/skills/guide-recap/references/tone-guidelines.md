> 📚 **AI Spark Wiki** · Claude Code 知识库

# 语调规范

基于 CHANGELOG 条目生成社交内容的规则。核心原则：**以价值吸引受众，而非噱头**。

## 应做 / 禁做 清单

### 应做

- 使用 CHANGELOG 中的具体数字（`227 -> 257`、`+522 lines`、`4 new diagrams`）
- 说明用户现在能做什么（`测试你的知识`、`新增视觉指南……`）
- 提出真实的问题（`视觉学习者？`、`你是如何 review PR 的？`）
- 注明有名有姓的来源（`基于 Pat Cullen 的工作流`、`来自 Addy Osmani 的研究`）
- 使用精准的动作动词（`added`、`integrated`、`documented`、`evaluated`）
- 按名称引用具体模式（`Permutation Frameworks`、`Split-Role Agents`）

### 禁做

- 使用炒作词汇：`game-changer`、`revolutionary`、`incredible`、`amazing`、`must-have`
- 制造 FOMO：`你落后了`、`别被落下`、`所有人都在用这个`
- 制造虚假紧迫感：`立即行动`、`限时`、`赶在截止前`
- 使用标题党：`这一个技巧`、`你不会相信`、`隐藏功能`
- 捏造数据：`快 10 倍`、`节省数小时`、`提升 300% 生产力`
- LinkedIn 帖子超过 3-4 个表情符号，推文超过 2 个
- 过度承诺：`你唯一需要的指南`、`完全掌握`

## 语言规则

### 法语（FR）

| 格式 | 语体 | 示例 |
|------|------|------|
| LinkedIn | 正式（Vouvoiement） | `Vous utilisez Claude Code au quotidien ?` |
| Newsletter | 正式（Vouvoiement） | `Vous trouverez dans cette version...` |
| Twitter/X | 非正式（Tutoiement） | `Tu connais les Permutation Frameworks ?` |
| Slack | 非正式（Tutoiement） | `Nouvelle version dispo, check ca` |

### 英语（EN）

- 所有格式均用第二人称（`you`）直接称呼
- 使用美式英语拼写（`optimize`、`analyze`，而非 `optimise`、`analyse`）
- 不使用英式英语习语或拼写

## 表情符号预算

| 格式 | 最多表情符号数 | 放置位置 |
|------|-------------|---------|
| LinkedIn | 3-4 | 开头（0-1）、要点（每项 1 个，最多 3 个）、CTA（0-1） |
| Twitter | 2 | 开头（1）、关键点（1） |
| Newsletter | 2-3 | 仅用于章节标题 |
| Slack | 4-6 | 状态标记、强调 |

推荐使用：`+`、`->`、技术符号优先于装饰性符号。
避免：火焰、火箭、爆炸、100、脑洞大开（营销老梗）。

## CTA 规则

| 格式 | CTA 风格 | 链接目标 |
|------|---------|---------|
| LinkedIn | 软性提问或价值陈述 | 落地页 URL |
| Twitter | 简短行动呼吁或链接 | GitHub 仓库 |
| Newsletter | 带上下文的明确链接 | 落地页 URL |
| Slack | 直接链接 | GitHub 仓库 |

禁止使用：`点击此处`、`看看这个`、`链接在 bio` 等模式。

## 输出前质量检查清单

在输出任何社交内容前，请逐项确认：

1. [ ] 所有数字均来自实际的 CHANGELOG 条目
2. [ ] 无炒作词汇（对照禁做清单检查）
3. [ ] 表情符号数量在预算内
4. [ ] 法语语体与格式匹配（vous/tu）
5. [ ] 英语使用美式拼写
6. [ ] CTA 链接指向正确目标
7. [ ] 使用有名有姓来源时已注明出处
8. [ ] 无捏造指标或百分比
