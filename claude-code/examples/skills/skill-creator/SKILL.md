> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: skill-creator
description: "为新的 Claude Code skill 搭建脚手架，包含 SKILL.md、frontmatter 和捆绑资源。适用于创建自定义 skill、在团队中统一 skill 结构，或将 skill 打包分发。"
allowed-tools: Write Bash
effort: low
---

# Skill 创建器

生成具有正确目录结构、YAML frontmatter 和可选捆绑资源的新 Claude Code skill。

## 何时使用

- 为项目创建新的自定义 skill
- 在团队中统一 skill 结构
- 生成包含脚本、参考资料和资源文件的 skill 模板
- 将 skill 打包以供分发

## Skill 目录结构

```
skill-name/
├── SKILL.md          # 必需：包含 YAML frontmatter 的主 skill 文件
├── scripts/          # 可选：确定性任务的可执行代码
├── references/       # 可选：按需加载的参考文档
└── assets/           # 可选：模板、图片、样板文件（不自动加载到上下文）
```

## 工作流程

### 1. 创建 Skill

```
在 ~/.claude/skills/ 中创建一个名为 "my-skill-name" 的新 skill
```

或指定具体用途：

```
创建一个从 git 提交生成发布说明的 skill，
包含 CHANGELOG.md 和 Slack 公告的模板
```

或通过初始化脚本创建：

```bash
python3 ~/.claude/skills/skill-creator/scripts/init_skill.py <skill-name> --path <output-directory>
```

### 2. 生成的 SKILL.md 模板

创建的 SKILL.md 遵循以下结构：

```markdown
---
name: skill-name
description: "这个 skill 做什么。Use when [触发条件]。"
---

# Skill 名称

## 何时使用
- 触发条件 1
- 触发条件 2

## 这个 Skill 做什么
1. **第一步**：说明
2. **第二步**：说明

## 如何使用
[使用示例]

## 示例
**用户**：「示例提示词」
**输出**：[示例输出]
```

### 3. 验证 Skill

创建后，检查以下项目：

1. **Frontmatter**：`name` 为 kebab-case，1-64 个字符；`description` 为带引号的字符串，包含 "Use when" 子句
2. **内容**：包含"何时使用"章节，列有触发条件，以及至少一个使用示例
3. **结构**：SKILL.md 不超过 5000 词；references 和 assets 位于正确的子目录中
4. **测试**：用真实用例调用 skill，确认输出符合预期

### 4. 打包分发（可选）

```bash
python3 ~/.claude/skills/skill-creator/scripts/package_skill.py <path/to/skill-folder> [output-directory]
```

## 组织模式

| 模式 | 最适合 | 结构 |
|---------|----------|-----------|
| **基于工作流** | 顺序化流程 | 逐步说明 |
| **基于任务** | 多项操作 | 任务集合 |
| **参考/指南** | 标准、规范 | 规则与示例 |
| **基于能力** | 相互关联的功能 | 功能描述 |

## 示例：创建发布说明 Skill

**用户**：「创建一个能生成 3 种输出格式的发布说明 skill」

**步骤**：
1. 初始化：`init_skill.py release-notes-generator --path ~/.claude/skills/`
2. 在 `assets/` 中添加模板：`changelog-template.md`、`pr-release-template.md`、`slack-template.md`
3. 在 `references/` 中添加规则：`tech-to-product-mappings.md`
4. 完善 `SKILL.md`，添加使用说明
5. 验证：检查 frontmatter，用真实的提交范围测试
6. 打包：`package_skill.py ~/.claude/skills/release-notes-generator`

## 技巧

- 保持 SKILL.md 在 5000 词以内，以提高上下文使用效率
- 将不常变动的领域知识放在 `references/` 中
- 将模板放在 `assets/` 中，避免自动加载到上下文
- 在 description frontmatter 中始终包含 "Use when" 子句
- 打包前先用真实用例测试
