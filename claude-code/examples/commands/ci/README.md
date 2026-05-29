> 📚 **AI Spark Wiki** · Claude Code 知识库

# CI 命令

用于 CI/CD 工作流的斜杠命令。自动检测技术栈（Python/Node/Rust），同时支持 GitLab CI 和 GitHub Actions。

## 命令列表

| 命令 | 说明 |
|---------|-------------|
| `/ci:all` | 完整流水线：测试 + 类型检查 + 推送 + 流水线 URL。提 PR 前唯一需要运行的命令。 |
| `/ci:tests` | 运行测试套件。自动检测 pytest、vitest、cargo test。 |
| `/ci:pipeline` | 推送当前分支并返回流水线追踪 URL。 |
| `/ci:status` | 显示当前分支的流水线状态。 |

## 使用模式

```
# 每次提 PR 前：
/ci:all

# 仅运行测试：
/ci:tests
/ci:tests tests/test_billing.py   # 指定文件

# 推送后检查流水线：
/ci:status
/ci:status 42   # 指定 MR/PR 编号

# 跳过测试直接推送（例如仅修改文档）：
/ci:all --skip-tests
```

## 技术栈支持

| 技术栈 | 测试命令 | 类型检查 |
|-------|-------------|------------|
| Python + uv | `uv run pytest --tb=short -q` | mypy（可选） |
| Node + pnpm | `pnpm vitest run` | `pnpm tsc --noEmit` |
| Node + npm | `npm test` | `npx tsc --noEmit` |
| Rust | `cargo test --quiet` | `cargo clippy` |

## CI 平台支持

所有命令同时支持 **GitLab CI**（通过 `glab` CLI）和 **GitHub Actions**（通过 `gh` CLI）。
如果两个 CLI 均未安装，命令将回退为仅显示流水线 URL。
