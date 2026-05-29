> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: land-and-deploy
description: 合并 PR、等待 CI、验证部署、执行金丝雀检查——完整的落地流水线
argument-hint: "[--skip-checks] [--env staging|production]"
effort: high
disable-model-invocation: true
---

# 落地与部署

完整的落地流水线：合并 PR、等待 CI、验证部署、执行健康检查。

从 `/ship` 结束的地方继续。`/ship` 负责创建 PR，本命令负责合并并验证生产环境。

**默认非交互式运行。** 用户说"落地它"——就落地它。只在关键就绪门控和硬性阻塞问题时停止。

## 执行步骤

### 步骤 1：预检

```bash
# 验证 GitHub CLI 已认证
gh auth status

# 从当前分支检测 PR（或使用提供的参数）
gh pr view --json number,state,title,url,mergeStateStatus,mergeable,baseRefName,headRefName
```

**停止条件：**
- GitHub CLI 未认证 → "请先运行 `gh auth login`"
- 不存在 PR → "该分支未找到 PR，请先运行 `/ship`。"
- PR 已合并 → "PR 已经合并。"
- PR 已关闭 → "PR 已关闭，请先重新打开。"

---

### 步骤 2：CI 状态检查

```bash
# 检查当前 CI 状态
gh pr checks --json name,state,status,conclusion

# 检查合并冲突
gh pr view --json mergeable -q .mergeable
```

**停止条件：**
- 必需检查 FAILING（失败）→ 显示失败的检查，停止
- `mergeable` 为 `CONFLICTING` → "PR 存在合并冲突，请解决后推送再落地。"
- 必需检查 PENDING（待定）→ 继续步骤 3（等待 CI）
- 所有检查通过 → 跳至步骤 3.5（就绪门控）

---

### 步骤 3：等待 CI（如有待定任务）

```bash
# 监视 CI 检查，超时时间 15 分钟
gh pr checks --watch --fail-fast
```

- CI 通过 → 继续步骤 3.5
- CI 失败 → 停止，显示失败信息
- 超时（15 分钟）→ "CI 已运行 15 分钟，请手动排查。"

记录 CI 等待时长，用于部署报告。

---

### 步骤 3.5：合并前就绪门控

**这是不可逆合并前唯一的关键确认环节。** 收集所有证据，然后获取明确批准。

#### 审查新鲜度检查

```bash
# 自上次审查以来该分支有多少次提交？
git log --oneline $(git merge-base HEAD origin/main)..HEAD | wc -l

# 审查完成后有什么变更？
git log --oneline -10
```

新鲜度阈值：
- 自审查后 0–3 次提交 → 当前（绿色）
- 4+ 次提交且涉及代码改动 → 过期（黄色——审查可能未反映当前代码）
- 未找到审查记录 → 未执行（黄色）

#### 测试结果

```bash
# 立即运行测试——仅限快速测试
npm test 2>/dev/null || pnpm test 2>/dev/null || \
  pytest --tb=short -q 2>/dev/null || \
  go test ./... 2>/dev/null

# 检查退出码
echo "Tests exit code: $?"
```

测试失败 = 阻塞。不能在测试失败时合并。

#### 文档检查

```bash
# 该分支是否更新了 CHANGELOG 和文档？
git diff --name-only $(git merge-base HEAD origin/main)...HEAD -- \
  README.md CHANGELOG.md ARCHITECTURE.md CONTRIBUTING.md CLAUDE.md VERSION
```

若 CHANGELOG.md 和 VERSION 未被修改，且 diff 包含新功能 → 警告。

#### 就绪报告

展示摘要并请求明确确认：

```
╔══════════════════════════════════════════════════════════╗
║              合并前就绪报告                               ║
╠══════════════════════════════════════════════════════════╣
║  PR: #NNN — [标题]                                        ║
║  分支：feature-branch → main                              ║
║                                                          ║
║  审查                                                    ║
║    审查状态：当前 / 过期（N 次提交）/ 未执行               ║
║                                                          ║
║  测试                                                    ║
║    快速测试：通过 / 失败（阻塞）                           ║
║                                                          ║
║  文档                                                    ║
║    CHANGELOG：已更新 / 未更新（警告）                      ║
║    VERSION：已升级 / 未升级（警告）                        ║
║                                                          ║
║  警告：N 个  |  阻塞：N 个                                ║
╚══════════════════════════════════════════════════════════╝

选项：
  A) 合并——所有检查绿色
  B) 暂不合并——先处理警告
  C) 强行合并——我了解风险
```

若用户选择 B，列出需要处理的具体事项并停止。

---

### 步骤 4：合并 PR

```bash
# 合并（从仓库设置自动检测合并方式，合并后删除分支）
gh pr merge --auto --delete-branch

# 若未启用自动合并，使用备用方案
# gh pr merge --squash --delete-branch
```

记录合并提交的 SHA 和时间戳。

若因权限错误合并失败 → "你没有合并权限，请让维护者来合并。"

若合并队列已激活，轮询直至合并完成：

```bash
# 每 30 秒轮询一次，30 分钟后超时
gh pr view --json state -q .state
```

---

### 步骤 5：平台检测

检测本项目的部署方式，以便我们知道需要验证什么。

```bash
# 从配置文件检测平台
[ -f fly.toml ]         && echo "PLATFORM: fly"
[ -f render.yaml ]      && echo "PLATFORM: render"
[ -f vercel.json ] || [ -d .vercel ] && echo "PLATFORM: vercel"
[ -f netlify.toml ]     && echo "PLATFORM: netlify"
[ -f Procfile ]         && echo "PLATFORM: heroku"
[ -f railway.toml ]     && echo "PLATFORM: railway"

# 检测 GitHub Actions 部署工作流
for f in .github/workflows/*.yml .github/workflows/*.yaml; do
  [ -f "$f" ] && grep -qiE "deploy|release|production|cd" "$f" 2>/dev/null && echo "DEPLOY_WORKFLOW: $f"
done

# 分类 diff 范围（前端 / 后端 / 文档 / 配置）
git diff --name-only $(git merge-base HEAD~1 origin/main)...HEAD | \
  awk '{
    if (/\.(css|scss|tsx|jsx|html|svg)$/ || /components|pages|public\//) f=1;
    if (/api\/|server\/|backend\/|\.(go|py|rb|java)$/) b=1;
    if (/README|CHANGELOG|docs\/|\.(md)$/) d=1;
    if (/\.env|config\/|\.toml$|\.yaml$/) c=1;
  } END {
    if (f) print "SCOPE_FRONTEND=true";
    if (b) print "SCOPE_BACKEND=true";
    if (d) print "SCOPE_DOCS=true";
    if (c) print "SCOPE_CONFIG=true";
  }'
```

**决策树：**
- 仅文档 diff → 跳过部署验证，直接进入步骤 8
- 无部署工作流且未提供 URL → 询问用户该项目是否有 Web 部署
- 否则 → 继续步骤 6

---

### 步骤 6：等待部署

**GitHub Actions 部署工作流：**

```bash
# 找到由合并提交触发的运行
gh run list --branch main --limit 10 --json databaseId,headSha,status,conclusion,workflowName

# 轮询直至完成（30 秒间隔，20 分钟超时）
gh run view <run-id> --json status,conclusion
```

**平台专属策略：**

| 平台 | 检测方式 | 等待策略 |
|------|----------|----------|
| Vercel / Netlify | 推送后自动部署 | 等待 60 秒传播，然后检查 |
| Fly.io | 存在 `fly.toml` | `fly status --app <app>` — 检查 `started` 状态 |
| Render | 存在 `render.yaml` | 轮询生产 URL 直至返回 200 |
| Heroku | 存在 `Procfile` | `heroku releases --app <app> -n 1` |
| Railway | 存在 `railway.toml` | 轮询生产 URL |
| 仅 GitHub Actions | `.github/workflows/` 含部署步骤 | 轮询 `gh run view` |

若部署失败 → 提供排查日志或创建回滚提交的选项。

记录部署时长，用于报告。

---

### 步骤 7：生产健康检查

使用步骤 5 中的 diff 范围决定检查深度：

| Diff 范围 | 金丝雀检查深度 |
|-----------|--------------|
| 仅文档 | 已在步骤 5 跳过 |
| 仅配置 | 仅 HTTP 200 冒烟测试 |
| 仅后端 | 状态 + 响应时间检查 |
| 前端（任意） | 完整：状态 + 响应时间 + 内容检查 |
| 混合 | 完整检查 |

**完整健康检查序列：**

```bash
# 1. 页面加载（200 状态）
curl -sf -o /dev/null -w "%{http_code}" "${PROD_URL}" 2>/dev/null

# 2. 响应时间检查
curl -sf -o /dev/null -w "%{time_total}" "${PROD_URL}" 2>/dev/null

# 3. 健康端点（如存在）
curl -sf "${PROD_URL}/health" 2>/dev/null || \
curl -sf "${PROD_URL}/api/health" 2>/dev/null

# 4. 内容检查——页面非空
curl -sf "${PROD_URL}" 2>/dev/null | wc -c
```

通过标准：
- HTTP 200 状态
- 响应时间低于 10 秒
- 页面有内容（>500 字节）
- 健康端点返回 200（如已配置）

若任何检查失败 → 提供回滚选项：

```
部署后健康检查发现问题：
  [发现——具体描述]

选项：
  A) 排查——这可能是正常现象（缓存预热、最终一致性）
  B) 回滚——回滚合并提交
  C) 继续——我将手动监控
```

---

### 步骤 8：回滚（如需要）

```bash
# 拉取最新基础分支
git fetch origin main

# 创建回滚提交
git checkout main
git revert <merge-commit-sha> --no-edit
git push origin main
```

若有冲突 → "回滚存在冲突，请手动运行 `git revert <sha>` 解决。"
若有分支保护 → "创建回滚 PR：`gh pr create --title 'revert: <标题>'`"

---

### 步骤 9：部署报告

```
落地与部署报告
═════════════════════════════════════════
PR：          #NNN — [标题]
分支：        feature-branch → main
合并时间：    [时间戳]（squash / merge）
合并 SHA：    [短 SHA]

耗时：
  CI 等待：   [X 分 Y 秒 / 已跳过]
  部署：      [X 分 Y 秒 / 未检测到工作流]
  健康检查：  [X 秒 / 已跳过]
  总计：      [端到端时长]

CI：          通过 / 失败 / 已跳过
部署：        通过 / 失败 / 无工作流
生产环境：    健康 / 降级 / 已跳过 / 已回滚
  状态：      [HTTP 状态码]
  响应：      [X 毫秒]

结论：已部署并验证 / 已部署（未验证）/ 已回滚
═════════════════════════════════════════
```

---

### 步骤 10：后续建议

部署报告完成后，建议相关后续步骤：

- 若生产 URL 已验证："运行 `/canary <url>` 进行 10 分钟的扩展监控。"
- 若有新功能已上线："运行 `/document-release` 更新项目文档。"

---

## 重要规则

- **永远不要强制推送。** 使用 `gh pr merge`——这是安全的。
- **永远不要跳过 CI。** 检查失败 = 停止。
- **单次生产检查。** 如需扩展监控，使用 `/canary`。
- **回滚始终是一个选项。** 在每个失败节点，提供回滚作为退出方案。
- **合并后删除功能分支**（通过 `--delete-branch`）。
- **目标**：用户输入 `/land-and-deploy`，接下来看到的就是部署报告。

## 用法

```
/land-and-deploy                                    # 自动检测 PR，不使用金丝雀 URL
/land-and-deploy https://app.example.com            # 自动检测 PR + 验证该 URL
/land-and-deploy 123                                # 指定 PR 编号
/land-and-deploy 123 https://app.example.com        # PR 编号 + 验证 URL
```

## 相关命令

- `/ship` — 先运行此命令创建 PR
- `/canary` — 部署后扩展监控循环
- `/review-pr` — 落地前审查 PR

$ARGUMENTS
