> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ship
description: 全面的部署前验证，确保发布准备就绪
argument-hint: "[--no-push] [--changelog-only] [--dry-run]"
effort: medium
disable-model-invocation: true
---

# Ship 命令 - 部署前检查清单

全面的部署前验证，确保发布准备就绪。

## 目的

在每次生产部署前运行，验证：
- 代码质量门控
- 测试覆盖率
- 安全检查
- 文档更新
- 环境就绪状态

## 部署前检查清单

### 🔴 阻断项（必须通过）

```bash
# 1. 所有测试通过
npm test 2>/dev/null || pnpm test 2>/dev/null || yarn test 2>/dev/null
echo "Exit code: $?"

# 2. 无 TypeScript/lint 错误
npm run typecheck 2>/dev/null || npx tsc --noEmit
npm run lint 2>/dev/null || npx eslint .

# 3. 构建成功
npm run build 2>/dev/null || pnpm build 2>/dev/null

# 4. 代码中无密钥
grep -rn "API_KEY=\|SECRET=\|PASSWORD=" --include="*.{ts,js,json}" . 2>/dev/null | grep -v node_modules | grep -v ".env.example"
```

### 🟠 高优先级（应当通过）

```bash
# 5. 安全审计
npm audit --audit-level=high 2>/dev/null || echo "Run manually: npm audit"

# 6. 生产代码中无 console.log
grep -rn "console\.log\|console\.debug" --include="*.{ts,js,tsx,jsx}" src/ 2>/dev/null | grep -v "// allowed" | head -10

# 7. 关键路径中无 TODO/FIXME
grep -rn "TODO\|FIXME\|XXX\|HACK" --include="*.{ts,js}" src/ 2>/dev/null | head -10

# 8. 数据库迁移已就绪
[ -d "prisma/migrations" ] && echo "Prisma migrations: $(ls prisma/migrations | wc -l) total"
[ -d "migrations" ] && echo "Migrations: $(ls migrations | wc -l) total"
```

### 🟡 推荐项（锦上添花）

```bash
# 9. 文档已更新
git diff --name-only HEAD~5 | grep -E "README|CHANGELOG|docs/" | head -10

# 10. 版本号已更新
cat package.json | jq -r '.version' 2>/dev/null || echo "Check version manually"

# 11. 环境变量已记录
[ -f ".env.example" ] && echo "✅ .env.example exists" || echo "⚠️ Missing .env.example"
```

## 输出格式

---

### 🚀 发布就绪报告

**分支**：[当前分支]
**提交**：[HEAD 短哈希]
**目标**：[生产/预发布]
**时间戳**：[日期/时间]

### 阻断项（部署前必须修复）

| 检查项 | 状态 | 详情 |
|-------|--------|---------|
| 测试 | ✅/❌ | X 通过，Y 失败 |
| TypeScript | ✅/❌ | X 个错误 |
| Lint | ✅/❌ | X 个警告，Y 个错误 |
| 构建 | ✅/❌ | 成功/失败 |
| 密钥 | ✅/❌ | X 个潜在泄露 |

### 高优先级

| 检查项 | 状态 | 操作 |
|-------|--------|--------|
| 安全审计 | ⚠️/✅ | X 个漏洞 |
| Console 日志 | ⚠️/✅ | src/ 中发现 X 处 |
| TODO | ⚠️/✅ | X 个关键 TODO |
| 迁移 | ⚠️/✅ | X 个待执行 |

### 推荐项

| 检查项 | 状态 | 备注 |
|-------|--------|------|
| 文档已更新 | ⚠️/✅ | CHANGELOG 已更新 |
| 版本号已更新 | ⚠️/✅ | 当前版本：X.Y.Z |
| 环境变量已记录 | ⚠️/✅ | .env.example 已存在 |

### 📊 汇总

```
🔴 阻断项：    X/5 通过
🟠 高优先级：  X/4 通过
🟡 推荐项：    X/3 通过
─────────────────────────
总体：         [可以发布 / 未就绪]
```

### 🎯 待办事项

1. [最紧急的修复]
2. [第二优先级]
3. [第三优先级]

---

## 特定环境检查

### 生产部署

```bash
# 验证生产环境变量
[ -f ".env.production" ] && echo "Production env exists"

# 检查调试标志
grep -rn "DEBUG=true\|NODE_ENV=development" .env* 2>/dev/null

# 验证 API 端点指向生产环境
grep -rn "localhost\|127\.0\.0\.1" --include="*.{ts,js,json}" src/ 2>/dev/null | grep -v test | head -5
```

### 预发布部署

```bash
# 预发布专项检查
[ -f ".env.staging" ] && echo "Staging env exists"

# 预发布功能开关
grep -rn "FEATURE_FLAG\|ENABLE_" .env* 2>/dev/null
```

## CI/CD 集成

添加到你的流水线中：

```yaml
# GitHub Actions 示例
ship-check:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Run ship checklist
      run: |
        npm ci
        npm test
        npm run typecheck
        npm run lint
        npm run build
        npm audit --audit-level=high
```

## 部署后验证

部署完成后，请验证：

```bash
# 1. 健康检查
curl -s https://your-app.com/health | jq .

# 2. 版本检查
curl -s https://your-app.com/version | jq .

# 3. 冒烟测试
npm run test:smoke 2>/dev/null || echo "Run smoke tests manually"
```

## 回滚准备

发布前，确保能够回滚：

```bash
# 记录当前生产标签
git describe --tags --abbrev=0

# 验证回滚流程文档是否存在
[ -f "docs/runbooks/rollback.md" ] && echo "✅ Rollback docs exist"

# 检查数据库迁移是否可逆
# Prisma: prisma migrate diff
# Rails: rails db:rollback (dry-run)
```

## 使用方式

**完整检查：**
```
/ship
```

**生产部署：**
```
/ship --production
```

**快速检查（仅阻断项）：**
```
/ship --quick
```

**指定目标环境：**
```
/ship --target=staging
```

## 使用建议

1. **早跑多跑**：不要等到部署当天才运行
2. **CI 中自动化**：让阻断项导致流水线失败
3. **团队共识**：明确定义什么是阻断项，什么是警告
4. **记录例外情况**：若跳过某项检查，请注明原因
5. **部署后持续监控**：监控确认成功后，发布才算完成

## 相关命令

- `/release-notes` - 生成变更日志和公告
- `/validate-changes` - 基于大模型的代码审查
- `/security` - 深度安全审计

$ARGUMENTS
