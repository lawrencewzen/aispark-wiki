> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "RTK Token 优化模板"
description: "使用 RTK CLI 代理最小化 token 消耗的 CLAUDE.md 配置"
tags: [claude-md, template, performance]
---

# RTK Token 优化

**背景**：使用 RTK（Rust Token Killer）最小化命令输出产生的 token 消耗。

## 需要优化的命令

以下高输出量命令始终使用 RTK 包装器：

### Git 操作（平均减少 92.3%）
- `rtk git log` 替代 `git log`
- `rtk git status` 替代 `git status`
- `rtk git diff` 替代 `git diff`

### 文件操作（平均减少 69.4%）
- `rtk find "*.md" .` 替代 `find . -name "*.md"`
- `rtk read <file>` 替代 `cat <file>`（适用于超过 10K 行的大文件）
- `rtk ls .` 替代 `ls -la`
- `rtk grep "pattern"` 替代 `grep -r "pattern"`

### JS/TS 技术栈（减少 70-90%）
- `rtk vitest run` 替代 `pnpm test`
- `rtk pnpm list` 替代 `pnpm list`
- `rtk pnpm outdated` 替代 `pnpm outdated`
- `rtk prisma migrate status` 替代 `pnpm prisma migrate status`

### Rust 工具链（减少 80-90%）
- `rtk cargo test` 替代 `cargo test`
- `rtk cargo build` 替代 `cargo build`
- `rtk cargo clippy` 替代 `cargo clippy`

### Python（减少 90%）
- `rtk python pytest` 替代 `pytest`

### Go（减少 90%）
- `rtk go test` 替代 `go test`

### GitHub CLI（减少 79-87%）
- `rtk gh pr view <num>` 替代 `gh pr view <num>`
- `rtk gh pr checks <num>` 替代 `gh pr checks <num>`

## Token 节省目标

**基准**：每次 30 分钟会话约 15 万 token
**使用 RTK 后**：约 4.5 万 token（减少 70%）

## 安装

```bash
# Homebrew (macOS/Linux)
brew install rtk-ai/tap/rtk

# Cargo (all platforms)
cargo install rtk

# Hook-first install
rtk init
```

## 验证

检查 RTK 是否可用：
```bash
rtk --version  # Should show: rtk 0.16.0+
```

## 不使用 RTK 的场景

- 快速探索（1-2 条命令）：额外开销得不偿失
- 已使用 Grep/Read 等工具（Claude 原生工具已做优化）
- 输出量极小（不足 100 个字符）：收益微乎其微

## 自动化

通过钩子自动使用 RTK（参见 `.claude/hooks/bash/rtk-wrapper.sh`），或使用 `rtk init` 进行钩子优先安装。
