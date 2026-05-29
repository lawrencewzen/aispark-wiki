> 📚 **AI Spark Wiki** · Claude Code 知识库

# 上下文工程模板

上下文工程是一种有意识地设计 Claude 在会话开始时所接收信息的实践——将你的 CLAUDE.md 及辅助文件视为生产系统，而非一次性配置。这些模板提供了构建、衡量和维护该系统所需的一切。

## 文件说明

| 文件 | 描述 |
|------|------|
| `profile-template.yaml` | 用于按人员组装上下文的开发者配置文件 |
| `skeleton-template.md` | 带逐章节说明的 CLAUDE.md 骨架模板（含注释） |
| `assembler.ts` | 从 profile + modules 构建 CLAUDE.md 的 TypeScript 脚本 |
| `eval-questions.yaml` | 20 个自评问题，用于审核你的 CLAUDE.md |
| `canary-check.sh` | 行为回归测试脚本（结构验证） |
| `ci-drift-check.yml` | 用于每周检测上下文漂移的 GitHub Actions 工作流 |
| `context-budget-calculator.sh` | 测量你的上下文配置的常驻 token 成本 |
| `rules/knowledge-feeding.md` | 会话后主动更新上下文的规则模板 |
| `rules/update-loop-retro.md` | 用于记录会话学习成果的复盘模板 |

## 快速开始

**新项目——3 步获得可用的 CLAUDE.md：**

```bash
# 1. 复制骨架并填入你的项目详情
cp examples/context-engineering/skeleton-template.md CLAUDE.md

# 2. 检查上下文预算（保持在 10K token 以内）
bash examples/context-engineering/context-budget-calculator.sh .

# 3. 运行金丝雀检查以验证结构
bash examples/context-engineering/canary-check.sh .
```

**已有项目——审核并改进：**

```bash
# 运行结构检查
bash examples/context-engineering/canary-check.sh .

# 然后使用 eval-questions.yaml 手动为你的 CLAUDE.md 打分
# 目标：16+ / 20
```

**团队配置——按开发者个人化：**

```bash
# 安装 assembler 所需依赖
npm install js-yaml @types/js-yaml ts-node typescript

# 复制配置文件模板并自定义
cp examples/context-engineering/profile-template.yaml .claude/profiles/yourname.yaml
# 编辑 .claude/profiles/yourname.yaml，填入你的技术栈和偏好

# 组装你的 CLAUDE.md
ts-node examples/context-engineering/assembler.ts \
  --profile .claude/profiles/yourname.yaml \
  --modules .claude/modules \
  --output CLAUDE.md
```

**持续维护：**

```bash
# 每周：运行金丝雀检查
bash examples/context-engineering/canary-check.sh .

# 每次会话后：使用复盘模板
# 参见 rules/update-loop-retro.md

# 添加 CI 工作流以实现自动化漂移检测
cp examples/context-engineering/ci-drift-check.yml .github/workflows/context-drift.yml
```

## 指南章节

完整方法论与原则：`guide/core/context-engineering.md`

这里的模板是操作层——指南解释了每个设计决策背后的理由。
