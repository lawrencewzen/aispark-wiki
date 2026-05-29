> 📚 **AI Spark Wiki** · Claude Code 知识库

# Talk Pipeline Skills

6 阶段 skill 流水线，将原始素材（文章、文字稿、笔记）转化为带有 AI 生成幻灯片的完整会议演讲。

## 安装

将 `talk-pipeline/` 目录复制到项目的 `.claude/skills/` 文件夹：

```bash
cp -r examples/skills/talk-pipeline ~/.claude/skills/
```

或仅安装所需阶段（每个阶段相互独立）。

## 阶段说明

| 阶段 | Skill 文件 | 模式 | 描述 |
|------|-----------|------|------|
| 1 | `stage-1-extract/SKILL.md` | REX + Concept | 将原始素材提取为结构化摘要 |
| 2 | `stage-2-research/SKILL.md` | 仅 REX | Git 考古 + 时间线 |
| 3 | `stage-3-concepts/SKILL.md` | REX + Concept | 带评分的概念目录 |
| 4 | `stage-4-position/SKILL.md` | REX + Concept | 角度、标题、简介 + 检查点 |
| 5 | `stage-5-script/SKILL.md` | REX + Concept | 5 幕演讲稿 + 幻灯片规格 + Kimi 提示词 |
| 6 | `stage-6-revision/SKILL.md` | REX + Concept | 修订表 + Q&A 速查表 |
| — | `orchestrator/SKILL.md` | REX + Concept | 单次调用运行完整流水线 |

## 快速开始

**完整流水线（推荐）**：
```
/talk-pipeline
```
编排者会询问元数据并运行所有适用阶段。

**单独阶段**：
```
/talk-stage1-extract
/talk-stage4-position
```

**带参数**：
```
/talk-pipeline --rex --slug=my-talk --event="Conf 2026" --duration=30
/talk-pipeline --concept --slug=my-idea
```

## 输出约定

所有文件输出到项目根目录的 `talks/` 下：
```
talks/{YYYY}-{slug}-summary.md
talks/{YYYY}-{slug}-git-archaeology.md
talks/{YYYY}-{slug}-concepts.md
talks/{YYYY}-{slug}-angles.md
talks/{YYYY}-{slug}-pitch.md
talks/{YYYY}-{slug}-kimi-prompt.md
...
```

## 使用 Kimi 提示词

第 5 阶段会生成 `{slug}-kimi-prompt.md`。生成幻灯片的步骤：
1. 打开文件，确认没有残留的 `{PLACEHOLDER}`
2. 访问 [kimi.com](https://kimi.com)（免费，无需 API）
3. 复制并粘贴整个提示词
4. Kimi 将根据你的幻灯片内容生成深色主题演示文稿

## 文档

完整工作流指南：[guide/workflows/talk-pipeline.md](../../../guide/workflows/talk-pipeline.md)
