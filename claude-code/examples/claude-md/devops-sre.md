> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "DevOps/SRE CLAUDE.md 模板"
description: "适用于基础设施项目和 SRE 工作流的 CLAUDE.md 配置"
tags: [claude-md, template, devops, ci-cd, observability]
---

# DevOps/SRE CLAUDE.md 模板

为基础设施项目和 SRE 工作流优化的 CLAUDE.md 配置。

## 使用方式

将以下内容复制到项目的 `CLAUDE.md` 文件中，并自定义 `[方括号]` 标注的部分。

---

## 模板

```markdown
# DevOps/SRE 项目配置

## 基础设施上下文

### 环境
- 云服务商：[AWS/GCP/Azure/自建]
- Kubernetes：[EKS/GKE/AKS/k3s/无]
- IaC 工具：[Terraform/Pulumi/CloudFormation/Ansible]
- CI/CD：[GitHub Actions/GitLab CI/Jenkins/ArgoCD]

### 服务地图
- [service-1]：[描述，关键路径：是/否]
- [service-2]：[描述，关键路径：是/否]
- [database]：[PostgreSQL/MySQL/MongoDB，托管位置]

### 访问方式
- 集群访问：[kubectl context 名称]
- 云 CLI：[aws/gcloud/az profile 名称]
- 密钥：[Vault/SSM/Secrets Manager - 绝不分享实际值]

## FIRE 框架默认规则

所有基础设施问题均使用 FIRE 框架处理：
- **F**irst Response（首响应）：明确症状、影响范围、近期变更
- **I**nvestigate（排查）：基于证据的系统性诊断
- **R**emediate（修复）：提出方案，等待批准
- **E**valuate（评估）：生成事后复盘，整理预防措施

## 安全规则

### 未经批准禁止执行
- `kubectl delete` 或 `kubectl scale down`
- `terraform destroy`
- 任何生产数据库写操作
- IAM/安全组修改
- 生产命名空间中的任何命令

### 必须满足
- 变更前提供回滚方案
- 确认环境（生产 vs 预发布）
- 扩缩容操作的影响评估

## 响应偏好

### 处理故障时
- 以影响评估开始
- 优先缓解而非排查根因（初期）
- 提供精确命令，而非泛泛指导
- 所有操作包含时间戳

### 代码审查时
- 关注点：安全性、资源限制、幂等性
- 标记：硬编码值、缺失的错误处理
- 建议：补充监控/告警

### 编写文档时
- 格式：带代码块的 Markdown
- 风格：Runbook 格式（编号步骤）
- 包含：前置条件、回滚步骤、验证步骤

## 常用上下文

### Kubernetes 命名空间
- `production`：[关键服务，需审批]
- `staging`：[可自由测试]
- `monitoring`：[Prometheus, Grafana]
- `ingress`：[nginx, cert-manager]

### Terraform 工作空间/模块
- `modules/`：[共享基础设施组件]
- `environments/prod/`：[生产环境，默认仅 plan]
- `environments/staging/`：[可安全 apply]

### 监控
- 指标：[Prometheus/CloudWatch/Datadog URL]
- 日志：[ELK/CloudWatch/Loki URL]
- 告警：[PagerDuty/OpsGenie 集成]

## 团队规范

### Commit 消息
- 格式：[conventional commits / 自定义格式]
- 示例：`fix(k8s): increase memory limit for payment-service`

### PR 要求
- [ ] 包含 Terraform plan 输出
- [ ] 列出受影响的服务
- [ ] 记录回滚流程

### Runbook 格式
```
# [Runbook 标题]
## 症状
## 前置条件
## 步骤
## 验证
## 回滚
## 升级处理
```
```

---

## 定制指南

### 面向 Kubernetes 密集型团队

添加到"常用上下文"：
```markdown
### 关键 Pod
- `payment-api`：直接影响营收，最大停机时间 30 秒
- `auth-service`：阻断所有已认证请求
- `api-gateway`：唯一入口

### 扩缩容规则
- payment-api：最小 3，最大 10，CPU > 70% 时扩容
- auth-service：最小 2，最大 5，按连接数扩容
```

### 面向 Terraform 密集型团队

添加以下章节：
```markdown
## Terraform 规范
- State 后端：[S3 存储桶 / GCS 存储桶]
- 锁表：[DynamoDB 表名]
- 模块注册表：[内部 / Terraform 官方注册表]
- 所需 provider 版本：[参见 versions.tf]

### 模块标准
- 所有资源使用 var.tags 打标签
- 命名规范：{项目}-{环境}-{资源}
- 输出：必须导出 ARN、ID、名称
```

### 面向多云团队

添加到"环境"章节：
```markdown
### 云凭证
- AWS：Profile `company-prod` / `company-staging`
- GCP：项目 `company-prod-123` / `company-staging-456`
- Azure：订阅 `prod-sub-id` / `staging-sub-id`

### 跨云服务
- DNS：[AWS Route53 / Cloudflare]
- CDN：[CloudFront / Cloud CDN]
- 密钥：[HashiCorp Vault - URL]
```

---

## 与智能体集成

将此 CLAUDE.md 与 DevOps/SRE 智能体配合使用：

```json
{
  "agents": {
    "sre": {
      "path": ".claude/agents/devops-sre.md",
      "model": "sonnet"
    }
  }
}
```

然后通过以下方式调用：`@sre investigate this pod crash`

---

## 参见

- [DevOps & SRE 指南](../../guide/ops/devops-sre.md) — 完整 FIRE 框架文档
- [DevOps 智能体](../agents/devops-sre.md) — 基础设施任务的智能体 persona
- [安全加固](../../guide/security/security-hardening.md) — 安全最佳实践
