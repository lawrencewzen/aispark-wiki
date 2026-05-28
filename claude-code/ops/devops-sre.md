> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "DevOps 与 SRE：使用 Claude Code"
description: "用 Claude Code 进行基础设施诊断的 FIRE 框架与 DevOps 工作流"
tags: [devops, guide, ci-cd, workflows]
---

# DevOps 与 SRE：使用 Claude Code

**阅读时间**：30 分钟
**技能水平**：中级（假设具备 DevOps 基础知识）
**前置条件**：Claude Code 基础（主指南[第 1-2 节](#getting-started)）

---

> **FIRE 框架**：使用 Claude Code 进行基础设施诊断的系统性方法。
>
> **F**irst Response（首次响应）→ **I**nvestigate（调查）→ **R**emediate（修复）→ **E**valuate（评估）

---

## 目录

1. [快速开始](#快速开始)
2. [模式：基础设施诊断](#模式基础设施诊断)
3. [模式：事件响应](#模式事件响应)
4. [模式：基础设施即代码](#模式基础设施即代码)
5. [护栏与采用](#护栏与采用)
6. [快速参考](#快速参考)

---

# 快速开始

**目标**：5 分钟内用 Claude Code 进行 DevOps 工作。

## 快速自检

| 情况 | 跳转到 |
|-----------|---------|
| 我正在处理紧急事件 | [紧急情况：K8s 故障排查](#kubernetes-故障排查) |
| 第一次为 DevOps 使用 Claude | [教程：第一次诊断](#你的第一次基础设施诊断) |
| 想自动化 runbook | [模式：事件响应](#模式事件响应) |
| 为团队评估 | [护栏与采用](#护栏与采用) |
| 需要可直接使用的提示词 | [快速参考](#快速参考) |

## FIRE 框架

使用 Claude 进行每次基础设施诊断都遵循这一模式：

```
F - 首次响应  → 给 Claude 症状 + 上下文
I - 调查      → Claude 分析日志、指标、配置
R - 修复      → Claude 提出修复方案（需人工审批）
E - 评估      → 事后分析、文档、预防
```

### 为什么用 FIRE？

| 阶段 | 人类角色 | Claude 角色 |
|-------|------------|-------------|
| **F**irst Response | 描述症状，提供上下文 | 分诊，优先检查项 |
| **I**nvestigate | 运行命令，粘贴输出 | 分析，关联，假设 |
| **R**emediate | **审批或拒绝** | 提出修复方案，说明影响 |
| **E**valuate | 审查，共享知识 | 生成事后分析，记录 |

**关键**：Claude 提议，你审批。这在以下情况尤为重要：
- 破坏性操作（删除、缩容、重启）
- 生产环境
- 安全敏感变更

## 你的第一次基础设施诊断

### 示例：Pod CrashLoopBackOff

```bash
# 步骤 1：首次响应 - 提供上下文
claude "payment-service 命名空间中有一个 Pod 处于 CrashLoopBackOff 状态。
环境：生产环境
集群：gke-prod-us-east1
开始时间：10 分钟前
我应该首先检查什么（3 件事）？"

# 步骤 2：调查 - 运行建议的命令并共享输出
kubectl describe pod payment-api-7d4b8c6f5-x2j9k -n payment-service
# 复制输出并粘贴给 Claude

# 步骤 3：修复 - Claude 提议，你审批
claude "根据这个 describe 输出，修复方案是什么？
约束：给我精确命令但不要执行任何操作。"

# 步骤 4：评估 - 记录事件
claude "为我们的 Wiki 创建简短事件说明：
- 发生了什么
- 根本原因
- 应用的修复
- 预防建议"
```

### Claude 的返回内容（首次响应示例）

```markdown
## CrashLoopBackOff 分诊（优先级顺序）

1. **检查退出代码和最新日志**
   ```bash
   kubectl logs payment-api-7d4b8c6f5-x2j9k -n payment-service --previous
   ```
   最常见：OOM（退出代码 137）、配置错误（退出代码 1）、依赖失败

2. **检查 Pod 事件和资源限制**
   ```bash
   kubectl describe pod payment-api-7d4b8c6f5-x2j9k -n payment-service
   ```
   关注：OOMKilled、ImagePullBackOff、存活探针失败

3. **检查最近的部署变更**
   ```bash
   kubectl rollout history deployment/payment-api -n payment-service
   ```
   如果最近有变更：在调查的同时考虑回滚
```

---

# 模式：基础设施诊断

**目标**：对常见基础设施问题进行系统性故障排查。

## Kubernetes 故障排查

### K8s MCP 服务器设置

对于持久性 K8s 上下文，安装 K8s MCP 服务器：

```json
// ~/.claude.json（或 .mcp.json）
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-kubernetes"]
    }
  }
}
```

**优势**：Claude 可以直接查询集群状态，减少复制粘贴循环。

**没有 MCP**：你需要手动将 kubectl 输出传给 Claude（仍然有效）。

### 按症状分类的提示词

复制粘贴这些提示词，替换 `<括号内>` 的值。

#### CrashLoopBackOff

```bash
kubectl describe pod <pod> -n <ns> | claude "分析这个 CrashLoopBackOff：
1. 退出代码是什么？含义是什么？
2. 检查最近 5 次重启的模式（时间、是一致的还是在升级？）
3. 根据事件提出 3 个最可能的根本原因
4. 给我调查每个假设的精确命令"
```

**Claude 会识别的常见原因**：
- 退出代码 137：OOMKilled（内存限制达到）
- 退出代码 1：应用程序错误（配置错误、缺少依赖）
- 退出代码 143：SIGTERM（优雅关闭超时）

#### OOMKilled

```bash
kubectl top pods -n <ns> && kubectl describe pod <pod> -n <ns> | claude "这个 Pod 被 OOMKilled 了：
1. 比较请求 vs 限制 vs 实际使用
2. 这是内存泄漏还是配置不足？
3. 如果是泄漏：容器中有哪些模式值得调查？
4. 如果是配置不足：根据这些数据建议最优资源设置"
```

**内存泄漏的后续提问**：
```bash
claude "Pod 每 2 小时重启一次，都是 OOMKilled。
内存从 200Mi 线性增长到 512Mi 限制后崩溃。
语言：Node.js 18
这个技术栈下内存泄漏的 3 个首要检查点是什么？"
```

#### ImagePullBackOff

```bash
kubectl describe pod <pod> -n <ns> | claude "ImagePullBackOff 诊断：
1. 这是认证问题、网络问题还是镜像名称错误？
2. 精确的错误信息告诉我们什么？
3. 给我验证镜像存在和凭证有效的命令"
```

#### Pending Pod（无法调度）

```bash
kubectl describe pod <pod> -n <ns> && kubectl describe nodes | claude "Pod 卡在 Pending 状态：
1. 这是资源限制、节点选择器还是亲和性规则？
2. 哪些节点被考虑过，为什么被拒绝？
3. 最快修复方案 vs 正确解决方案是什么？"
```

#### Service 无法访问

```bash
kubectl get svc,endpoints -n <ns> && kubectl describe svc <svc> -n <ns> | claude "Service 无法访问：
1. 有健康的端点吗？
2. 选择器是否正确匹配 Pod？
3. 是否有网络策略阻止流量？
为每种可能性提供诊断命令"
```

### 案例研究：生产事故根本原因

**情况**：电商平台，凌晨 3 点告警，checkout service 返回 503。

**FIRE 实战**：

```bash
# F - 首次响应
claude "事件：checkout-service 返回 503，10 分钟前开始。
影响：100% 结账尝试失败。
环境：AWS EKS 生产环境，us-east-1。
最近变更：2 小时前部署（新功能标志逻辑）。
最快的诊断路径是什么？"

# I - 调查（Claude 建议先检查 Pod）
kubectl get pods -n checkout -l app=checkout-service
# 输出：3/5 个 Pod 处于 CrashLoopBackOff

kubectl logs checkout-service-xxx --previous | tail -50 | claude "分析崩溃日志"
# Claude 识别：panic: nil pointer dereference in feature flag code

# R - 修复
claude "已识别根本原因：最近部署的功能标志逻辑中的空指针。
选项：
A) 回滚到之前版本
B) 修补空值检查
凌晨 3 点哪个更快更安全？"
# Claude 建议：回滚（更快、已验证状态，明天再正确修复）

kubectl rollout undo deployment/checkout-service -n checkout
# 2 分钟内服务恢复

# E - 评估（次日）
claude "根据这次事件生成事后分析：
时间线：凌晨 3:02 告警，3:15 找到根本原因，3:17 回滚，3:19 恢复
根本原因：commit abc123 的功能标志空指针
影响：15 分钟结账中断
格式：无责任归因，聚焦预防"
```

**结果**：MTTR 15 分钟，清晰事后分析，识别预防行动项。

## 日志分析与关联

### 多服务日志关联

```bash
# 收集相关服务的日志
kubectl logs -l app=api-gateway -n ingress --since=10m > gateway.log
kubectl logs -l app=auth-service -n auth --since=10m > auth.log
kubectl logs -l app=payment-service -n payment --since=10m > payment.log

# 分析关联
cat gateway.log auth.log payment.log | claude "关联这些日志：
1. 找出失败交易的请求流
2. 识别故障起源位置
3. 时间或特定端点是否有规律？
4. 创建事件时间线"
```

### 日志模式检测

```bash
# 在错误模式中找异常
grep -E "ERROR|WARN|Exception" app.log | claude "分析错误模式：
1. 将相似错误聚类（按类型分组，而非按时间戳）
2. 什么是最频繁的 vs 最严重的？
3. 哪些错误有关联（同一根本原因）？
4. 优先调查顺序"
```

### Prometheus/Grafana 查询帮助

```bash
claude "我需要一个 PromQL 查询来：
- 显示 payment-service 的 p99 延迟
- 按端点分组
- 超过 500ms 持续 5 分钟时告警
同时包含 alert rule YAML"
```

## Claude 做不到的事（限制）

了解限制可以防止沮丧和不安全的依赖。

| 限制 | 影响 | 变通方法 |
|------------|--------|------------|
| **无实时集群状态** | 无法看到当前 Pod 状态 | 使用 K8s MCP 或粘贴 kubectl 输出 |
| **无直接 API 访问** | 无法调用 AWS/GCP API | 使用 MCP 服务器或共享 CLI 输出 |
| **上下文窗口限制** | 最多约 100K Token（词元） | 聚焦相关日志，不要全量转储 |
| **无持久记忆** | 会话间遗忘 | 使用 CLAUDE.md 保存项目上下文 |
| **幻觉风险** | 可能建议无效标志 | 运行前始终验证命令 |
| **无实时指标** | 无法看到当前图表 | 截图 Grafana 或粘贴指标值 |
| **无 secrets 访问** | 无法读取 vault/secrets | 好事！永远不要将 secrets 分享给任何 LLM |

### 何时不用 Claude

- **30 秒内需要决策的紧急情况**：你的肌肉记忆更快
- **高度机密的事件**：数据泄露调查（法律影响）
- **简单、显而易见的修复**：如果你知道答案，直接做
- **受合规限制的环境**：检查是否允许使用 AI 工具
- **AI 特定安全事件**：检测到提示注入、MCP 被攻击、智能体窃取数据 → 见[安全加固——响应](../security/security-hardening.md#part-3-response-when-things-go-wrong)获取专用程序（终止开关架构、遏制级别、事件时间线）

### Claude 的优势领域

- **复杂根本原因分析**：多个相互作用的系统
- **文档生成**：事后分析、runbook、程序
- **学习新工具**：不熟悉的云服务、新 k8s 功能
- **第二意见**：验证你的假设
- **批量操作**：为多个环境生成配置

---

# 模式：事件响应

**目标**：事件管理的结构化工作流。

## 独自处理事件的工作流

**现实**：凌晨 3 点，只有你一个人。这个工作流专为单人设计。

### FIRE 实战：独自处理事件

#### F - 首次响应（30 秒）

```bash
claude "事件：[症状 - 具体描述]
上下文：[服务]，[环境]，[开始时间]
最近变更：[部署、基础设施变更、流量峰值]
当前影响：[受影响用户百分比，如已知则填写收入影响]
最关键的 3 件事是什么？"
```

**示例**：
```bash
claude "事件：API /checkout 端点返回 500 错误
上下文：checkout-service，production-us-east1，5 分钟前开始
最近变更：凌晨 2:45 部署了 v2.3.4，添加了新支付提供商
当前影响：约 30% 的结账请求失败
最关键的 3 件事是什么？"
```

#### I - 调查（2-5 分钟）

运行 Claude 建议的命令，共享输出：

```bash
# Claude 建议先检查 Pod 健康
kubectl get pods -n checkout | claude "快速评估这个 Pod 列表"

# 然后检查最近日志
kubectl logs -l app=checkout --since=5m | head -100 | claude "分析错误模式"

# 然后检查部署差异对比
kubectl rollout history deployment/checkout-service -n checkout | claude "最近部署改变了什么？"
```

**实用技巧**：一个终端运行命令，另一个进行 Claude 对话。

#### R - 修复（需审批）

```bash
claude "根据调查：
- 根本原因：[你的理解]
- 证据：[关键发现]

提出修复选项：
1. 快速缓解（恢复服务）
2. 正确修复（解决根本原因）

约束：我需要在任何操作前审批。给我精确命令。"
```

**审批门示例**：
```
Claude："建议：回滚到 v2.3.3
命令：kubectl rollout undo deployment/checkout-service -n checkout
风险：低 - 之前版本稳定运行 2 周
替代方案：缩减新支付提供商的功能标志

你想采取哪种方式？"

你："执行回滚"
```

#### E - 评估（事后，不在事件期间）

```bash
claude "创建事件事后分析：

时间线：
- 凌晨 2:45：部署 v2.3.4
- 凌晨 3:00：第一次告警触发
- 凌晨 3:05：事件声明
- 凌晨 3:12：识别根本原因（新支付提供商代码空指针）
- 凌晨 3:15：启动回滚
- 凌晨 3:17：服务恢复

格式：无责任归因，聚焦系统而非人
包含：带负责人的行动项"
```

## 事件沟通

### 利益相关者更新生成器

```bash
claude "为利益相关者生成事件更新：

事件：Checkout 服务降级
当前状态：已缓解，正在监控
影响：15 分钟 30% 结账失败
预计完全解决时间：2 小时（下次部署时正确修复）

受众：非技术高管
语气：专业、令人安心、基于事实
长度：最多 3 句话"
```

**输出示例**：
> 我们经历了 15 分钟的结账服务中断，影响约 30% 的交易，现已解决。问题是由最近更新中的软件 bug 引起的，已快速回滚。我们将在下次计划维护窗口期间部署永久修复，预计对客户无影响。

### 事件桥接提示词

用于实时事件频道：

```bash
claude "我正在管理一个事件桥接。帮我：
1. 维护持续的事件时间线
2. 当遇到死胡同时建议下一步调查步骤
3. 每 15 分钟起草沟通更新
4. 在我应该升级时发出标志

当前状态：[粘贴最新更新]
我现在应该向桥接沟通什么？"
```

## 多智能体模式：事后分析

**何时使用多智能体**：不在活跃事件期间使用。用于事后全面分析。

```bash
# 智能体 1：时间线重建
claude "你是事件时间线分析师。
从这些日志和 Slack 消息重建精确时间线：
[粘贴日志和沟通]
输出：带时间戳的事件，谁在什么时候做了什么"

# 智能体 2：根本原因分析
claude "你是根本原因分析师。
根据这个时间线和系统架构，进行五问分析：
[粘贴智能体 1 的时间线]
输出：根本原因链，贡献因素"

# 智能体 3：预防建议
claude "你是 SRE 流程改进专家。
根据这个根本原因分析：
[粘贴智能体 2 的 RCA]
输出：优先预防措施，工作量估算，负责人建议"
```

### 案例研究：OpsWorker.ai MTTR 降低

**背景**：SRE 团队管理 200+ 个微服务，5 名值班工程师。

**使用 Claude 之前**：
- 平均 MTTR：45 分钟
- 事后分析：经常延迟或跳过
- 知识孤岛：每个工程师了解不同服务

**Claude 集成**：
1. FIRE 框架用于所有事件
2. Claude 在 1 小时内生成初始事后分析草稿
3. Runbook 用 Claude 辅助的故障排查增强

**3 个月后**：
- 平均 MTTR：18 分钟（降低 60%）
- 事后分析完成率：95% 在 24 小时内
- 知识共享：Claude 生成的 runbook 所有人均可访问

**关键洞察**：最大收益不是速度，而是一致性和文档。

---

# 模式：基础设施即代码

**目标**：利用 Claude 进行 Terraform、Ansible 和 GitOps 工作流。

## Terraform 与 Claude

### 参考：Anton Babenko 的 Terraform Skill（技能模块）

Claude Code 最全面的 Terraform Skills（技能模块）：

**仓库**：[antonbabenko/terraform-skill](https://github.com/antonbabenko/terraform-skill)
**作者**：Anton Babenko（terraform-aws-modules 创建者，10 亿次下载）

```bash
# 安装
cd ~/.claude/skills/
git clone https://github.com/antonbabenko/terraform-skill.git terraform
```

**提供的内容**：
- 模块结构最佳实践
- AWS、GCP、Azure 模式
- 状态管理指南
- CI/CD 集成模式

### 常用 Terraform 提示词

#### Plan 审查

```bash
terraform plan -out=plan.txt && cat plan.txt | claude "审查这个 Terraform plan：
1. 有危险的变更吗？（数据丢失、中断）
2. 变更是我们预期的吗？
3. 有遗漏的变更我们应该添加吗？
4. 如果可见，成本影响如何？"
```

#### 模块生成

```bash
claude "生成一个 Terraform 模块用于：
- AWS ECS Fargate 服务
- 带 ALB 和目标组
- 基于 CPU 的自动扩展
- 从 SSM Parameter Store 获取 Secrets

遵循这些约定：
- 使用 for_each 而非 count
- 所有资源用 var.tags 打标签
- 输出服务 URL 和 ARN"
```

#### 状态迁移助手

```bash
claude "我需要将一个资源移动到不同的状态文件：
当前状态：terraform-prod/terraform.tfstate
资源：aws_s3_bucket.logs
目标状态：terraform-shared/terraform.tfstate

最安全的程序是什么？包含回滚步骤。"
```

### 配置漂移检测工作流

```bash
# 检测漂移
terraform plan -detailed-exitcode 2>&1 | tee drift.txt

# 用 Claude 分析
cat drift.txt | claude "分析这个 Terraform 漂移：
1. 什么在 Terraform 之外发生了变化？
2. 这个漂移是预期的（手动变更）还是令人担忧的？
3. 我们应该导入这些变更还是恢复到 Terraform 状态？
4. 最安全的修复路径是什么？"
```

## Ansible 与 Claude

### Playbook 审查

```bash
cat playbook.yml | claude "审查这个 Ansible playbook：
1. 幂等性问题？
2. 安全关切？
3. 错误处理缺口？
4. 性能优化？"
```

### Role 生成

```bash
claude "生成一个 Ansible role 用于：
- 安装和配置 Nginx
- 通过 Let's Encrypt（certbot）配置 SSL 证书
- 加固配置（禁用 server tokens 等）
- 日志轮转

遵循最佳实践：
- 使用 handlers 处理服务重启
- 变量放在 defaults/main.yml
- 包含 molecule 测试结构"
```

## GitOps 与 Claude

### ArgoCD Application 审查

```bash
cat application.yaml | claude "审查这个 ArgoCD Application：
1. 同步策略是否适合该环境？
2. 是否定义了资源健康检查？
3. 有同步波次排序问题吗？
4. 命名空间和项目权限是否正确？"
```

### Helm Values 生成

```bash
claude "生成 Helm values 用于将 [应用名] 部署到：
- 环境：staging
- 资源：有限（成本敏感）
- 副本数：2
- Ingress：仅内部
- Secrets：来自 external-secrets operator

基础 chart：[chart 名称]
包含解释每个值的注释"
```

## 安全审查自动化

### 基础设施安全扫描

```bash
# 运行 tfsec 或 checkov，分析结果
tfsec . --format=json | claude "分析这些安全发现：
1. 按严重性和可利用性排序
2. 在我们的环境中哪些是误报？
3. 对于真实问题：修复方案是什么？
4. 哪些可以用文档化的理由忽略？"
```

### IAM 策略审查

```bash
cat iam-policy.json | claude "审查这个 IAM 策略：
1. 是否遵循最小权限原则？
2. 有过于宽泛的操作吗？（*、admin 等）
3. 资源限制是否合适？
4. 建议一个仍然有效的更严格版本"
```

---

# 护栏与采用

**目标**：安全实施 Claude Code 并获得团队认可。

## 成本意识

### Claude Code 成本

| 模型 | 输入（每百万 Token（词元）） | 输出（每百万 Token（词元））|
|-------|-------------------|-------------------|
| Sonnet 4 | 3 美元 | 15 美元 |
| Opus 4 | 15 美元 | 75 美元 |

**典型 DevOps 会话**：20K-50K Token（词元）= 0.10-0.50 美元

**成本控制策略**：
1. 常规任务使用 Sonnet（默认）
2. 复杂多系统分析保留 Opus
3. 对话过长时使用 `/compact` 减少上下文
4. 避免粘贴整个日志文件；先 grep 相关部分

### Claude 建议带来的基础设施成本

**注意**：Claude 看不到你的云账单。始终问：

```bash
claude "在应用这些变更之前，估算：
1. 每月成本影响（计算、存储、网络）
2. 有可能无限扩展的资源吗？
3. 成本优化替代方案？"
```

## 安全边界

### 永远不要与 Claude 分享

| 数据类型 | 原因 | 替代方案 |
|-----------|---------|-------------|
| API 密钥、令牌 | 可能被缓存/记录 | 使用占位符：`<API_KEY>` |
| 生产 secrets | 安全风险 | 描述 secret 类型，而非值 |
| 客户 PII | 隐私/合规 | 使用匿名示例 |
| 专有算法 | 知识产权保护 | 描述行为，而非代码 |
| 含 PII 的事件详情 | 法律责任 | 分享前脱敏 |

### 安全提示词模板

```bash
claude "调试这个认证问题：
- 服务：auth-service
- 错误：有效令牌返回 401 Unauthorized
- 环境：staging（非生产）
- 令牌格式：带 claims 的 JWT [user_id, org_id, exp]
- 注意：我已脱敏所有实际令牌值

这是脱敏后的日志：
[粘贴已替换 secrets 的日志]"
```

### 生产环境审批门

始终对以下操作要求人工审批：

```yaml
# 示例：生产变更清单
需要审批：
  - kubectl delete
  - kubectl scale（缩容）
  - terraform destroy
  - DROP TABLE / DELETE FROM
  - rm -rf（tmp 目录外）
  - 任何生产数据库写操作
  - 任何 IAM 策略变更
  - 任何安全组修改
```

## 团队推广清单

### 第一阶段：试点（1-2 名工程师，2 周）

- [ ] 为试点用户安装 Claude Code
- [ ] 创建包含公共上下文的团队 CLAUDE.md
- [ ] 记录前 5 个成功用例
- [ ] 识别一个要标准化的工作流
- [ ] 跟踪节省的时间（前后对比）

### 第二阶段：扩展（团队，4 周）

- [ ] 在团队会议中分享试点经验
- [ ] 创建团队特定提示词库
- [ ] 建立安全准则（什么可以分享/不可以分享）
- [ ] 设置共享 Skills（技能模块）/命令仓库
- [ ] 定义何时使用 Claude，何时不用

### 第三阶段：优化（持续进行）

- [ ] 每月审查提示词库
- [ ] A/B 测试：Claude 辅助 vs 传统处理类似事件
- [ ] 回馈社区（awesome-lists、本指南）
- [ ] 跟踪 MTTR、事后分析完成率、文档质量

### 避免的采用陷阱

| 陷阱 | 为什么发生 | 预防措施 |
|---------|---------------|------------|
| **过度依赖** | Claude 太有帮助了 | 强制留出学习时间，不仅追求输出 |
| **盲目信任** | 命令通常有效 | 运行前始终审查 |
| **上下文倾倒** | 希望 Claude 自己弄明白 | 提供聚焦的上下文，而非全部内容 |
| **跳过验证** | 时间压力 | 将验证纳入工作流 |
| **影子使用** | 无团队可见性 | 分享成功，将使用常态化 |

---

# 快速参考

## FIRE 框架摘要

```
┌─────────────────────────────────────────────────────────────┐
│ F - 首次响应                                                │
│   "事件：[症状]。上下文：[服务, 环境, 时间]。              │
│    最近变更：[什么]。影响：[谁受影响]。                    │
│    最关键的 3 件事是什么？"                                │
├─────────────────────────────────────────────────────────────┤
│ I - 调查                                                    │
│   运行 Claude 建议的命令                                    │
│   共享输出："[输出] | claude '分析这个'"                  │
│   迭代直到识别根本原因                                      │
├─────────────────────────────────────────────────────────────┤
│ R - 修复                                                    │
│   "根据[发现]，提出修复方案。                              │
│    约束：在任何操作前我需要审批。"                         │
│   审批 → 执行 │ 拒绝 → 继续调查                          │
├─────────────────────────────────────────────────────────────┤
│ E - 评估                                                    │
│   "创建事后分析：时间线、根本原因、预防措施。              │
│    格式：无责任归因，带负责人的行动项。"                   │
└─────────────────────────────────────────────────────────────┘
```

## 按症状分类的提示词

### Kubernetes

| 症状 | 提示词 |
|---------|--------|
| CrashLoopBackOff | `kubectl describe pod <pod> -n <ns> \| claude "退出代码含义？3 个可能原因？调查命令？"` |
| OOMKilled | `kubectl top pods && describe pod \| claude "泄漏还是配置不足？最优资源？"` |
| ImagePullBackOff | `kubectl describe pod \| claude "认证、网络还是镜像名称错误？验证命令？"` |
| Pending | `kubectl describe pod && describe nodes \| claude "资源、选择器还是亲和性问题？"` |
| Service 无法访问 | `kubectl get svc,endpoints \| claude "健康端点？选择器匹配？网络策略？"` |

### 云/基础设施

| 症状 | 提示词 |
|---------|--------|
| 高延迟 | `[指标] \| claude "瓶颈位置？是计算、网络还是依赖？"` |
| 磁盘满 | `df -h && du -sh /* \| claude "什么在消耗空间？安全删除吗？"` |
| 连接拒绝 | `netstat -tlnp \| claude "服务在监听？端口正确？防火墙规则？"` |
| SSL 证书过期 | `openssl s_client -connect host:443 \| claude "距过期天数？续签步骤？"` |
| DNS 问题 | `dig +trace domain \| claude "解析在哪里失败？"` |

### Terraform

| 任务 | 提示词 |
|------|--------|
| Plan 审查 | `terraform plan \| claude "危险变更？遗漏变更？成本影响？"` |
| 漂移分析 | `terraform plan -detailed-exitcode \| claude "什么漂移了？预期的吗？修复？"` |
| 模块请求 | `claude "为[资源]生成 Terraform 模块，满足[需求]"` |

## DevOps 的 MCP 服务器

| 服务器 | 用途 | 安装 |
|--------|---------|---------|
| Kubernetes | 直接集群访问 | `npx -y @anthropic/mcp-kubernetes` |
| AWS | AWS API 访问 | `npx -y @anthropic/mcp-aws` |
| GCP | GCP API 访问 | `npx -y @anthropic/mcp-gcp` |
| Prometheus | 直接指标查询 | 社区：搜索 awesome-mcp-servers |
| Terraform | 状态/plan 分析 | 社区：搜索 awesome-mcp-servers |

**配置位置**：`~/.claude.json`（字段 `"mcpServers"`）

```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-kubernetes"]
    }
  }
}
```

## 外部资源

### Awesome Lists

- **[awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)**（8,100 stars）：包含 SRE 的智能体 persona
- **[awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)**（4,600 stars）：包含基础设施相关的 Skills（技能模块）

### 官方资源

- **[terraform-skill](https://github.com/antonbabenko/terraform-skill)**：Anton Babenko 的生产级 Terraform Skills（技能模块）
- **[Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)**：官方文档

### 社区

- **Anthropic Discord**：#claude-code 频道
- **Reddit**：r/ClaudeAI
- **GitHub**：在 awesome-lists 上提 issue 申请新功能

---

## 参见

- **[智能体模板](../../examples/agents/devops-sre.md)**：DevOps/SRE 智能体 persona
- **[CLAUDE.md 模板](../../examples/claude-md/devops-sre.md)**：DevOps 团队的项目配置
- **[安全加固指南](../security/security-hardening.md)**：额外安全实践
- **[架构指南](../core/architecture.md)**：Claude Code 内部工作原理

---

*欢迎贡献！如果你有效果良好的 DevOps 提示词，可以考虑添加到 awesome-lists 或向本指南提交 PR。*
