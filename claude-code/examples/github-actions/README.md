> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "GitHub Actions Workflows for Claude Code"
description: "开箱即用的 CI/CD 工作流，将 Claude Code 集成到 GitHub Actions"
tags: [ci-cd, devops, template, workflows]
---

# GitHub Actions Workflows for Claude Code

开箱即用的 GitHub Actions 工作流，将 Claude Code 集成到你的 CI/CD 流水线中。

## 前置条件

1. **安装 Claude GitHub App**：在你的组织/仓库安装（Actions 需要它来评论 PR/issue）
2. **添加 API Key 密钥**：进入仓库 Settings → Secrets and variables → Actions → New repository secret
   - 名称：`ANTHROPIC_API_KEY`
   - 值：从 [console.anthropic.com](https://console.anthropic.com) 获取的 Anthropic API 密钥
3. **复制工作流文件**：将 `.yml` 文件放入 `.github/workflows/` 目录
4. **测试**：开一个测试 PR 或 issue 观察运行效果

## 可用工作流

### 1. 代码审查 — 基于提示词（`claude-code-review.yml`）⭐ 推荐

**健壮模式**：提示词外置、防幻觉协议，支持 `/claude-review` 按需触发。

审查逻辑存放在 `.github/prompts/code-review.md`，可以随时调整评审标准，无需修改工作流 YAML。提示词在每条发现前强制执行验证步骤——Claude 必须通过 `Read`/`Grep` 确认问题后才上报。

**功能特性：**
- PR 打开/同步/标记就绪时触发，**以及** `/claude-review` 评论触发
- 提示词外置：编辑 `code-review.md` 即可为你的技术栈调整标准
- 防幻觉协议：不虚构行号，不提交未经验证的结论
- 结构化输出：`🔴 必须修复` / `🟡 建议修复` / `🟢 可跳过` 表格 + 行内评论
- `allowed_tools` 只读（对仓库无写权限）
- 支持 OAuth token（安装 Claude GitHub App 后无需 API 密钥）

**配置步骤：**
```bash
# 将两个文件复制到仓库
cp examples/github-actions/claude-code-review.yml .github/workflows/
mkdir -p .github/prompts
cp examples/github-actions/prompts/code-review.md .github/prompts/

# 添加密钥：CLAUDE_CODE_OAUTH_TOKEN（或 ANTHROPIC_API_KEY）
# 安装 Claude GitHub App：https://github.com/apps/claude
```

**自定义：**
编辑 `.github/prompts/code-review.md`，添加你的技术栈规范：
```markdown
## 技术栈说明
- TypeScript strict 模式，禁用 `any`
- React Server Components — 数据获取不使用 `useEffect`
- 所有数据库写操作必须经过 repository 层
- 新 API 路由需要集成测试
```

---

### 2. 自动 PR 审查（`claude-pr-auto-review.yml`）

**增强版本**：包含全面的审查标准和智能过滤。

PR 打开或更新时立即创建带行内评论的结构化审查报告。

**功能特性：**
- PR 打开/更新时自动进行代码审查
- 8 个关注维度：正确性、安全性、性能、可读性、可维护性、测试、最佳实践、破坏性变更
- 基于优先级的反馈：🔴 严重、🟡 重要、🟢 建议、💡 提示
- 智能文件过滤（跳过构建产物和锁文件）
- 跳过草稿 PR 以节省费用
- 附带风险评估的整体审查摘要
- 错误处理和降级通知
- 具体行的行内评论

**使用方法：**
```bash
# 复制工作流文件
cp examples/github-actions/claude-pr-auto-review.yml .github/workflows/

# 开一个 PR——Claude 会自动审查
```

**自定义：**
取消注释 `append_system_prompt` 部分以添加项目专属上下文：
```yaml
append_system_prompt: |
  项目规范：
  - 使用 TypeScript strict 模式
  - 遵循函数式编程范式
  - 所有函数必须有 JSDoc 注释
  - 测试覆盖率必须 >80%
```

---

### 2. 安全审查（`claude-security-review.yml`）

运行专项安全扫描，并将发现直接评论到 PR 上。

**功能特性：**
- 每个 PR 都进行安全专项分析
- 识别潜在漏洞
- 涵盖 OWASP Top 10
- 将发现以 PR 评论形式发布

**配置说明：**
```yaml
# 工作流文件中的可选参数：
exclude-directories: "docs,examples"    # 跳过指定目录
claudecode-timeout: "20"                # 超时时间（分钟）
claude-model: "claude-3-5-sonnet-20240620"  # 使用的模型
```

**使用方法：**
```bash
# 复制工作流文件
cp examples/github-actions/claude-security-review.yml .github/workflows/

# 此后每个 PR 都会自动扫描安全问题
```

---

### 3. Issue 分类（`claude-issue-triage.yml`）

新 issue 开启时，Claude 提出标签/严重程度建议，并发布整洁的分类评论。

**功能特性：**
- 自动 issue 分类
- 标签建议
- 严重程度评估（低、中、高、严重）
- 重复 issue 检测
- Markdown 格式的分类评论

**自动应用标签（可选）：**
若要自动应用建议的标签，编辑工作流文件，将以下内容修改：
```yaml
- name: Apply labels (optional)
  if: ${{ false }}  # 改为 true 以自动应用标签
```

**使用方法：**
```bash
# 复制工作流文件
cp examples/github-actions/claude-issue-triage.yml .github/workflows/

# 开一个新 issue——Claude 会自动分类
```

---

---

## 多模型审查配置

同时运行 Claude 和其他自动化审查工具（Gemini、Greptile、CodeRabbit），可以发现任何单一模型遗漏的问题。模式如下：各服务独立审查，然后由 Claude 综合共识。

**为什么用多模型？**
每个模型都有盲区。被 2 个以上独立审查者同时标记的问题是高置信度信号；各模型独特发现则提供了额外覆盖。

### 推荐组合（约 30 美元/月固定费用）

| 服务 | 费用 | 优势 |
|---------|------|----------|
| **Claude Code Review**（本工作流） | Anthropic 计划内含 | 深度推理，感知代码库 |
| **Gemini Code Assist** | 0 美元（Google Workspace 内含） | 独立 LLM，不同训练数据 |
| **Greptile** | 约 30 美元/月固定 | 跨文件上下文，依赖关系图 |

**备选方案**：CodeRabbit Pro（15 美元/开发者/月）提供交互式问答和时序图。

### 配置步骤

**第一步：安装 Gemini Code Assist**

1. GitHub Marketplace → 搜索 "Gemini Code Assist"
2. 安装并授权访问你的仓库
3. Gemini 将自动审查新 PR（以 `gemini-code-assist[bot]` 身份发布）
4. 通过 `.gemini/config.yaml` 进行可选配置：
   ```yaml
   code_review:
     comment_severity_threshold: MEDIUM
     max_comments_per_review: 20
   ```

**第二步：安装 Greptile**

1. 访问 [greptile.com](https://greptile.com) → 关联 GitHub 账号
2. 选择仓库——Greptile 会索引代码库（约 5 分钟）
3. 在控制台配置：目标分支、关注路径
4. 审查以 `greptile[bot]` 评论形式发布到 PR

**第三步：启用综合任务**

在 `claude-code-review.yml` 中，删除综合任务条件中的 `false &&`：

```yaml
# 修改前（已禁用）：
if: |
  false &&
  (github.event_name == 'pull_request' ...

# 修改后（已启用）：
if: |
  (github.event_name == 'pull_request' ...
```

**第四步：配置 CodeRabbit（可选）**

将本目录中的 `.coderabbit.yaml` 复制到仓库根目录。编辑 `path_instructions` 以匹配你的技术栈。

### 综合机制说明

`claude-code-review.yml` 中的 `multi-reviewer-synthesis` 任务：

1. 在 Claude 审查完成后等待 5 分钟（外部机器人通常 2-3 分钟内发布）
2. 通过 GitHub API 收集所有审查和评论
3. 若发布评论的审查者少于 2 人则静默跳过
4. Claude 识别共识（同一发现被 2 个以上审查者标记）与独特发现
5. 在 PR 上发布结构化的综合评论

### 目录中的文件

```
examples/github-actions/
├── README.md                      # 本文件
├── claude-code-review.yml         # 主审查 + 可选综合任务
├── .coderabbit.yaml               # CodeRabbit 配置（复制到仓库根目录）
├── claude-pr-auto-review.yml      # 行内提示词自动审查（备选方案）
├── claude-security-review.yml     # 安全专项扫描
├── claude-issue-triage.yml        # Issue 分类工作流
└── prompts/
    └── code-review.md             # 外置审查提示词（复制到 .github/prompts/）
```

---

## 自定义

### 模型选择
在工作流中设置 `CLAUDE_MODEL` 或 `claude-model` 参数：
```yaml
env:
  CLAUDE_MODEL: claude-3-5-sonnet-20240620
```

### 权限
每个工作流声明所需的最小权限：
- `pull-requests: write`：用于 PR 审查
- `issues: write`：用于 issue 分类
- `contents: read`：用于读取仓库内容

仅在组织有更严格策略要求时才进行调整。

### 范围过滤
使用 `paths:` 过滤器限制工作流触发条件：
```yaml
on:
  pull_request:
    paths:
      - 'src/**'
      - '!docs/**'
```

## 故障排查

**PR 上没有出现评论：**
- 确认已安装 Claude GitHub App
- 检查工作流是否有 `pull-requests: write` 权限

**应用标签时报 403：**
- 确保任务有 `issues: write` 权限
- 确认 `GITHUB_TOKEN` 有权访问该仓库

**Anthropic API 报错：**
- 确认 `ANTHROPIC_API_KEY` 已在仓库级别设置
- 检查密钥是否已过期

**YAML 语法错误：**
- 验证缩进：每层两个空格，不使用 Tab
- 使用 YAML 验证器：[yamllint.com](https://www.yamllint.com/)

## 高级用法

### 组合工作流
同时运行多个工作流实现全面自动化：
- 每个 PR 同时运行 PR 审查 + 安全审查
- 新 issue 同时运行 Issue 分类 + 自动打标签

### 自定义提示词
编辑工作流中的 `direct_prompt` 部分，定制 Claude 的关注重点：
```yaml
direct_prompt: |
  审查本 PR，重点关注：
  1. TypeScript 类型安全
  2. React 性能模式
  3. 无障碍合规性
  4. 测试覆盖率
```

### 与其他 Actions 集成
与现有工作流结合使用：
```yaml
jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: npm test

  claude-review:
    needs: tests  # 测试通过后再运行
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@main
        # ...
```

## 费用说明

这些工作流会消耗 Anthropic API 额度：
- **PR 审查**：每次约 $0.10-$0.50（取决于 diff 大小）
- **安全审查**：每次约 $0.20-$0.80
- **Issue 分类**：每个约 $0.05-$0.20

**降低费用的建议：**
- 使用 `paths:` 过滤器跳过文档/配置变更
- 设置条件：`if: github.event.pull_request.draft == false`
- 查看日志并调整模型选择

## 本目录文件说明

```
examples/github-actions/
├── README.md                        # 本文件
├── claude-code-review.yml           # 基于提示词的审查 + 可选综合任务
├── .coderabbit.yaml                 # CodeRabbit 配置（复制到仓库根目录）
├── claude-pr-auto-review.yml        # 行内提示词自动审查（备选方案）
├── claude-security-review.yml       # 安全扫描工作流
├── claude-issue-triage.yml          # Issue 分类工作流
└── prompts/
    └── code-review.md               # 外置审查提示词（复制到 .github/prompts/）
```

## 相关资源

- [Claude Code 文档](https://claude.ai/code)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Anthropic API 文档](https://docs.anthropic.com/)
- [Claude GitHub App](https://github.com/apps/claude)

## 许可

这些工作流作为示例提供，请按需自行调整。
