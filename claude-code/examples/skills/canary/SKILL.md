> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: canary
description: 部署后监控——在部署完成后监视生产环境并在出现回归时发出告警
argument-hint: "[--baseline]"
effort: medium
disable-model-invocation: true
---

# Canary — 部署后监控

在部署后监视线上应用。对错误和回归发出告警。与部署前的基线进行对比。

**两种模式：**
- `--baseline` — 在部署**之前**捕获当前状态
- *（默认）* — 在部署**之后**进行监控并与基线对比

## 执行说明

### 第 1 阶段：初始化

解析用户参数并检测部署上下文。

```bash
# Detect current branch and recent deploy commit
git branch --show-current
git log --oneline -5

# Auto-detect platform from config files
[ -f fly.toml ]         && echo "PLATFORM: fly"
[ -f render.yaml ]      && echo "PLATFORM: render"
[ -f vercel.json ]      && echo "PLATFORM: vercel"
[ -f netlify.toml ]     && echo "PLATFORM: netlify"
[ -f Procfile ]         && echo "PLATFORM: heroku"
[ -f railway.toml ]     && echo "PLATFORM: railway"

# Check for health endpoint
curl -sf "${URL}/health" -w "\n%{http_code}" 2>/dev/null | tail -1
curl -sf "${URL}/api/health" -w "\n%{http_code}" 2>/dev/null | tail -1
```

创建工作目录：

```bash
mkdir -p .canary/baselines .canary/reports .canary/screenshots
```

---

### 第 2 阶段：基线捕获（`--baseline` 模式）

在部署**之前**运行，捕获当前健康状态。

对每个待监控页面记录：

1. **HTTP 状态** — 页面是否返回 200？
2. **响应时间** — 加载需要多长时间？
3. **内容快照** — 关键文本内容，用于之后检测空白页

```bash
# For each page URL
for PAGE_PATH in "/" "/dashboard" "/settings" "/api/health"; do
  SLUG=$(echo "$PAGE_PATH" | tr '/' '_' | tr -d '?&=')
  RESULT=$(curl -sf -o /dev/null -w "%{http_code}|%{time_total}" "${BASE_URL}${PAGE_PATH}" 2>/dev/null)
  STATUS=$(echo "$RESULT" | cut -d'|' -f1)
  TIME_MS=$(echo "$RESULT" | awk -F'|' '{printf "%.0f", $2 * 1000}')
  echo "  ${PAGE_PATH}: HTTP ${STATUS}, ${TIME_MS}ms"
done
```

将基线保存至 `.canary/baselines/baseline.json`：

```json
{
  "url": "<base-url>",
  "timestamp": "<ISO-8601>",
  "branch": "<branch-name>",
  "commit": "<git-SHA>",
  "pages": {
    "/": { "status": 200, "time_ms": 450 },
    "/dashboard": { "status": 200, "time_ms": 680 },
    "/api/health": { "status": 200, "time_ms": 45 }
  }
}
```

然后**停止**并告知用户："基线已捕获。请部署你的变更，然后运行 `/canary <url>` 进行监控。"

---

### 第 3 阶段：页面发现

如果未指定页面，则自动发现待监控页面。

**从应用中获取：**

```bash
# Check sitemap if available
curl -sf "${URL}/sitemap.xml" 2>/dev/null | grep -oP '(?<=<loc>)[^<]+' | head -10

# Check robots.txt for known paths
curl -sf "${URL}/robots.txt" 2>/dev/null | grep -i "allow\|disallow" | head -10

# Common paths to always check
echo "Always check: / /login /dashboard /settings /api/health"
```

如果未找到任何页面，默认只监控 `/` 首页。

---

### 第 4 阶段：监控循环

在指定时长内进行监控（默认：10 分钟），每 60 秒执行一次检查。

**每次检查周期：**

```bash
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)
CHECK_NUM=$((CHECK_NUM + 1))

for PAGE_PATH in "${PAGES[@]}"; do
  # Check HTTP status and response time
  RESULT=$(curl -sf -o /dev/null -w "%{http_code}|%{time_total}" \
    --max-time 10 "${BASE_URL}${PAGE_PATH}" 2>/dev/null || echo "0|0")
  STATUS=$(echo "$RESULT" | cut -d'|' -f1)
  TIME_MS=$(echo "$RESULT" | awk -F'|' '{printf "%.0f", $2 * 1000}')

  # Compare against baseline
  BASELINE_STATUS=$(jq -r ".pages[\"${PAGE_PATH}\"].status // 200" .canary/baselines/baseline.json 2>/dev/null)
  BASELINE_TIME=$(jq -r ".pages[\"${PAGE_PATH}\"].time_ms // 1000" .canary/baselines/baseline.json 2>/dev/null)

  echo "  [Check #${CHECK_NUM}] ${PAGE_PATH}: HTTP ${STATUS} (${TIME_MS}ms)"
done
```

**告警级别：**

| 级别 | 条件 | 触发时机 |
|------|------|----------|
| **CRITICAL（严重）** | 页面加载失败 | HTTP 状态非 2xx、curl 超时、DNS 故障 |
| **HIGH（高）** | 新出现错误 | 错误率相比基线上升（控制台错误、5xx 响应） |
| **MEDIUM（中）** | 性能回归 | 响应时间超过基线 2 倍 |
| **LOW（低）** | 新出现死链 | 之前正常的路由现在返回 404 |

**核心原则：**
- **基于变化告警，而非绝对值。** 基线中有 3 个错误的页面，只要仍然是 3 个就没问题。出现 1 个**新**错误才触发告警。
- **容忍瞬态故障。** 只对持续 2 次以上连续检查的异常发出告警，单次网络抖动不算告警。

**当 CRITICAL 或 HIGH 告警连续触发 2 次时：**

```
CANARY ALERT
════════════════════════════════════════
Time:     [check #N at Xs elapsed]
Page:     [URL]
Level:    [CRITICAL / HIGH / MEDIUM / LOW]
Finding:  [what changed — be specific]
Baseline: [baseline value]
Current:  [current value]
════════════════════════════════════════
Options:
  A) Investigate now — stop monitoring, focus on this issue
  B) Continue monitoring — wait for next check to confirm
  C) Rollback — revert the deploy
  D) Dismiss — known issue, continue monitoring
```

---

### 第 5 阶段：健康报告

监控结束（或用户停止）后，生成摘要。

```
CANARY REPORT — [url]
═══════════════════════════════════════════════════
Duration:    [X minutes]
Checks:      [N total per page]
Pages:       [N pages monitored]
Commit:      [deployed SHA]
Status:      [HEALTHY / DEGRADED / BROKEN]

Per-Page Results:
─────────────────────────────────────────
  Page           Status      Avg Time   Alerts
  /              HEALTHY     450ms      0
  /dashboard     DEGRADED    1100ms     1 medium (was 450ms)
  /settings      HEALTHY     380ms      0
  /api/health    HEALTHY     45ms       0

Alerts Fired: [N] (X critical, Y high, Z medium, W low)

VERDICT: [DEPLOY HEALTHY / DEPLOY HAS ISSUES — see alerts above]
═══════════════════════════════════════════════════
```

将报告保存至 `.canary/reports/<date>-canary.md`。

---

### 第 6 阶段：基线更新

如果部署健康且用户想更新基线：

```bash
cp .canary/reports/latest-snapshot.json .canary/baselines/baseline.json
echo "Baseline updated to commit $(git rev-parse --short HEAD)"
```

---

## 输出格式

完整的 CANARY REPORT 模板见第 5 阶段。

监控过程中的内联告警格式：
```
[08:42:15] Check #3 — /dashboard: ALERT HIGH — response time 1250ms (baseline: 420ms)
[08:43:15] Check #4 — /dashboard: ALERT HIGH — response time 1180ms (baseline: 420ms)
→ Consistent across 2 checks. Firing alert.
```

## 用法

```
/canary https://app.example.com                # 监控首页 10 分钟
/canary https://app.example.com --baseline     # 部署前捕获基线
/canary https://app.example.com --duration 5m  # 监控 5 分钟
/canary https://app.example.com --quick        # 单次健康检查（无循环）
/canary https://app.example.com --pages /,/dashboard,/api/health
```

## 使用技巧

1. **部署到生产前务必捕获基线** — 运行 `/canary <url> --baseline`
2. **部署后立即开始监控** — 前 5 分钟能发现 90% 的回归问题
3. **CRITICAL 告警 = 立即排查** — 不要等监控结束再处理
4. **MEDIUM 告警（性能）** — 可能是缓存预热，再等 2-3 次检查后再行动
5. **将 `.canary/baselines/` 纳入 git** — 这样任何团队成员都能基于相同基线运行 canary

## 相关命令

- `/ship` — 部署前检查清单（在部署前运行）
- `/land-and-deploy` — 完整的合并到验证流水线（自动运行 canary）
- `/qa` — 上线前的交互式 QA 测试

$ARGUMENTS
