> 📚 **AI Spark Wiki** · Claude Code 知识库

# PR 工作流插件

自动化的拉取请求审查与验证系统。

## 安装

```bash
bash install.sh
```

## 组件

- **code-reviewer 智能体** — 自动化代码质量与安全检查
- **/review-pr 命令** — 分析 PR 并提供详细反馈
- **/pr 命令** — 快速 PR 准备工作流
- **pre-pr-check 钩子** — 在创建 PR 前验证变更

## 快速开始

```bash
# 审查现有 PR
/review-pr 123

# 准备新 PR
/pr

# 检查将要提交的内容
/validate-changes
```

## 功能特性

✓ 代码质量分析
✓ 安全扫描
✓ 测试覆盖率验证
✓ 合规检查
✓ 自动化建议
✓ 团队通知

## 参见

- `guide/workflows/code-review.md` — 完整审查工作流文档
- `/security-check` — 针对安全的 PR 审查
