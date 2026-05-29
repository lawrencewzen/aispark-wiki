> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ci-tests
description: 运行当前仓库的测试套件——自动检测 Python（pytest/uv）、Node（vitest/pnpm）或 Rust（cargo test）
argument-hint: "[文件或目录目标]"
allowed-tools: [Bash]
model: haiku
effort: low
disable-model-invocation: true
---

# /ci:tests — 运行测试

自动检测技术栈并使用正确的命令运行测试。

## 技术栈检测

```bash
if [ -f "uv.lock" ]; then
  STACK="python"
elif [ -f "pnpm-lock.yaml" ] || [ -f "package.json" ]; then
  STACK="node"
elif [ -f "Cargo.toml" ]; then
  STACK="rust"
else
  STACK="unknown"
fi
```

## 各技术栈命令

### Python（uv + pytest）

```bash
# 全部测试
uv run pytest --tb=short -q $ARGUMENTS

# 带覆盖率
uv run pytest --cov=src --cov-report=term-missing -q

# 指定文件或目录
uv run pytest $ARGUMENTS -v
```

### Node（pnpm + vitest）

```bash
# 全部测试
pnpm vitest run $ARGUMENTS

# 带覆盖率
pnpm vitest run --coverage

# 监听模式（开发用）
pnpm vitest
```

### Node（npm + jest）

```bash
npm test -- --passWithNoTests $ARGUMENTS
```

### Rust（cargo）

```bash
cargo test --quiet $ARGUMENTS 2>&1
```

## 预期输出

```
Tests — my-api (Python/pytest)
───────────────────────────────

uv run pytest --tb=short -q

[pytest output]

✅ 42 passed in 3.1s  →  Ready to push
```

失败时：
```
❌ 2 failed

FAILED tests/test_billing.py::TestInvoice::test_promo_expired
AssertionError: expected discount=0, got discount=10

→ Fix before pushing.
```

## 使用方式

```
/ci:tests
/ci:tests tests/test_orders.py
/ci:tests src/components/Button.test.tsx
```

目标：$ARGUMENTS
