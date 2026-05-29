> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: talk-pipeline
description: "编排完整的演讲准备流水线，从原始素材到修改清单，按顺序运行 6 个阶段，并在关键节点设置人工干预检查点，适用于 REX 模式或概念模式的演讲。当需要启动新的演讲流水线、从某个特定阶段恢复流水线，或运行完整的端到端准备工作流时使用。"
tags: [talk, pipeline, presentation, orchestrator]
allowed-tools: "Write, Read, AskUserQuestion, Task"
effort: medium
---

# 演讲流水线编排器

编排完整的演讲准备流水线——从原始素材到修改清单。可以运行完整流水线，也可以单独运行某一隔离阶段。

## 模式

- `--rex`：带 git/代码证明的 REX 演讲（变更日志、提交记录、可量化指标）
- `--concept`：基于文章、想法、笔记的概念演讲（跳过第 2 阶段）

## 用法

```
/talk-pipeline                          # 完整流水线，询问上下文信息
/talk-pipeline --stage=extract          # 单独运行某一隔离阶段
/talk-pipeline --rex                    # REX 模式（含 git 考古）
/talk-pipeline --concept                # 概念模式（跳过研究阶段）
/talk-pipeline --rex --slug=my-talk --event="Conf 2026" --date=2026-06-15 --duration=30
```

## 上下文收集

若未提供以下信息，通过 AskUserQuestion 询问：

```
- slug         : kebab-case 标识符（例如 my-talk-topic）
- event        : 活动名称（例如 Conf 2026、Tech Meetup）
- date         : 演讲日期（YYYY-MM-DD）
- duration     : 时长（分钟，例如 30）
- audience     : 受众画像（例如资深开发者、技术负责人、非技术人员）
- type         : --rex 或 --concept
- source_path  : 源材料路径（文章 .mdx、文字稿 .md、笔记）
- repo_path    : （仅 REX 模式）用于 git 考古的代码仓库路径
```

## 工作流

1. **收集上下文** — 通过 AskUserQuestion 获取必要元数据
2. **按模式路由** — `--rex` 或 `--concept`（概念模式跳过第 2 阶段）
3. **运行第 1 阶段** — `/talk-stage1-extract` — 始终最先执行
4. **并行运行第 2-4 阶段**（第 1 阶段确认后）
   - REX 模式：提取 → 研究 + 概念 + 定位（并行）
   - 概念模式：提取 → 概念 + 定位（并行，跳过研究）
5. **检查点** — 等待角度 + 标题选择（第 4 阶段输出）
6. **运行第 5 阶段** — `/talk-stage5-script`，使用已验证的选择
7. **运行第 6 阶段** — `/talk-stage6-revision`
8. **最终汇总** — 列出所有生成文件及其路径

## 依赖关系图

```
         extract（第 1 阶段）
               |
    ┌──────────┼──────────┐
    v          v          v
research    concepts   position
（第 2 阶段）（第 3 阶段）（第 4 阶段）
[仅 --rex]             [检查点]
    |          |          |
    └──────────┼──────────┘
               v
          script（第 5 阶段）
               |
               v
         revision（第 6 阶段）
```

## 阶段路由（--stage=X）

若提供了 `--stage` 参数，则只运行对应的 skill：

| 阶段 | 要调用的 skill |
|-------|----------------|
| extract | /talk-stage1-extract |
| research | /talk-stage2-research |
| concepts | /talk-stage3-concepts |
| position | /talk-stage4-position |
| script | /talk-stage5-script |
| revision | /talk-stage6-revision |

## 输出命名规范

```
talks/{YYYY}-{slug}-summary.md           # 提取
talks/{YYYY}-{slug}-git-archaeology.md   # 研究
talks/{YYYY}-{slug}-changelog-analysis.md
talks/{YYYY}-{slug}-timeline.md
talks/{YYYY}-{slug}-concepts.md          # 概念
talks/{YYYY}-{slug}-concepts-enriched.md
talks/{YYYY}-{slug}-angles.md            # 定位
talks/{YYYY}-{slug}-titre.md
talks/{YYYY}-{slug}-descriptions.md
talks/{YYYY}-{slug}-feedback-draft.md
talks/{YYYY}-{slug}-pitch.md             # 脚本
talks/{YYYY}-{slug}-slides.md
talks/{YYYY}-{slug}-kimi-prompt.md
talks/{YYYY}-{slug}-revision-sheets.md   # 修改
```

## 最终汇总格式

第 6 阶段完成后展示：

```
流水线完成。已生成文件：

第 1 阶段 — 提取：
  ✓ talks/{YYYY}-{slug}-summary.md

第 2 阶段 — 研究（仅 REX 模式）：
  ✓ talks/{YYYY}-{slug}-git-archaeology.md
  ✓ talks/{YYYY}-{slug}-changelog-analysis.md
  ✓ talks/{YYYY}-{slug}-timeline.md

第 3 阶段 — 概念：
  ✓ talks/{YYYY}-{slug}-concepts.md
  ✓ talks/{YYYY}-{slug}-concepts-enriched.md（如有代码仓库）

第 4 阶段 — 定位：
  ✓ talks/{YYYY}-{slug}-angles.md
  ✓ talks/{YYYY}-{slug}-titre.md
  ✓ talks/{YYYY}-{slug}-descriptions.md
  ✓ talks/{YYYY}-{slug}-feedback-draft.md

第 5 阶段 — 脚本：
  ✓ talks/{YYYY}-{slug}-pitch.md
  ✓ talks/{YYYY}-{slug}-slides.md
  ✓ talks/{YYYY}-{slug}-kimi-prompt.md   ← 复制粘贴到 kimi.com

第 6 阶段 — 修改：
  ✓ talks/{YYYY}-{slug}-revision-sheets.md

下一步：打开 kimi-prompt.md，确认没有残留的 {PLACEHOLDER}，然后粘贴到 kimi.com。
```

## 反模式

- 未获得用户明确选择的角度 + 标题，不得运行第 5 阶段
- 在 `--concept` 模式下不得运行第 2 阶段（研究）
- 上游阶段失败时，不得继续推进到下一阶段
- 不得在源材料中不存在的情况下编造指标或日期

## 验证清单

- [ ] 启动下游阶段前，所有上游文件均已存在
- [ ] 脚本阶段前已遵守第 4 阶段检查点
- [ ] 输出文件按规范命名 `talks/{YYYY}-{slug}-{stage}.md`
- [ ] 生成文件中无空占位符

## 使用技巧

- 编排器是首次使用的推荐入口
- 对于熟悉流水线的用户，直接运行单个阶段的 skill 更快
- `--stage=X` 参数适合在无需重跑完整流水线的情况下重新运行单个阶段

## 相关链接

- [第 1 阶段：提取](../stage-1-extract/SKILL.md)
- [第 2 阶段：研究](../stage-2-research/SKILL.md)
- [第 3 阶段：概念](../stage-3-concepts/SKILL.md)
- [第 4 阶段：定位](../stage-4-position/SKILL.md)
- [第 5 阶段：脚本](../stage-5-script/SKILL.md)
- [第 6 阶段：修改](../stage-6-revision/SKILL.md)
- [完整工作流指南](../../../../guide/workflows/talk-pipeline.md)
