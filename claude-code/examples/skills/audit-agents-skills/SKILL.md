> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: audit-agents-skills
description: "审计 Claude Code 智能体、技能和命令的质量与生产就绪状态。适用于评估技能质量、检查生产就绪评分，或将智能体与最佳实践模板进行对比。"
allowed-tools: Read Grep Glob Bash Write
effort: high
disable-model-invocation: true
metadata:
  version: 1.0.0
---

# 审计智能体/技能/命令（高级技能）

针对 Claude Code 智能体、技能和命令的全面质量审计系统。基于行业最佳实践，提供量化评分、对比分析和生产就绪等级评定。

## 用途

**问题**：手动验证智能体/技能容易出错且缺乏一致性。根据 LangChain Agent Report 2026 报告，29.5% 的组织在没有系统性评估的情况下部署智能体，导致"智能体缺陷"成为首要挑战（18% 的团队面临此问题）。

**解决方案**：基于 16 项加权标准的自动化质量评分，设有生产就绪阈值（80% = 最低 B 级，达标方可生产部署）。

**核心功能**：
- 量化评分（智能体/技能最高 32 分，命令最高 20 分）
- 加权标准（身份标识 3 倍权重，提示词 2 倍，验证 1 倍，设计 2 倍）
- 生产就绪等级评定（A-F 等级，80% 为阈值）
- 与参考模板的对比分析
- JSON/Markdown 双格式输出，支持程序化集成
- 针对不达标标准的修复建议

---

## 模式

| 模式 | 用法 | 输出 |
|------|-------|--------|
| **快速审计** | 仅检查前 5 项关键标准 | 快速通过/失败判断（20 个文件约 3-5 分钟） |
| **完整审计** | 每个文件检查全部 16 项标准 | 详细评分 + 建议（10-15 分钟） |
| **对比审计** | 完整审计 + 与模板基准对比 | 分析报告 + 差距识别（15-20 分钟） |

**默认模式**：完整审计（首次运行推荐）

---

## 方法论

### 为何选择这些标准？

16 项标准框架来源于：
1. **Claude Code 最佳实践**（终极指南第 4921 行：智能体验证清单）
2. **行业数据**（LangChain Agent Report 2026：评估缺口）
3. **生产故障**（社区反馈：硬编码路径、缺少错误处理等问题）
4. **组合模式**（技能应引用其他技能，智能体应保持模块化）

### 评分理念

**权重依据**：
- **身份标识（3 倍）**：如果用户无法找到/调用智能体，质量再好也没用（可发现性优先于质量）
- **提示词（2 倍）**：决定输出的可靠性与准确性
- **验证（1 倍）**：提升健壮性，但属于核心功能的次要因素
- **设计（2 倍）**：影响长期可维护性和可扩展性

**等级标准**：
- **A（90-100%）**：生产就绪，风险极低
- **B（80-89%）**：良好，达到生产部署阈值
- **C（70-79%）**：需在上线前改进
- **D（60-69%）**：存在明显缺口，不具备生产就绪条件
- **F（<60%）**：存在严重问题，需大幅重构

**行业对齐**：80% 阈值与软件工程生产部署最佳实践一致（如代码覆盖率 >80%、安全扫描通过率等）。

---

## 工作流程

### 第一阶段：发现

1. **扫描目录**：
   ```
   .claude/agents/
   .claude/skills/
   .claude/commands/
   examples/agents/      （如存在）
   examples/skills/      （如存在）
   examples/commands/    （如存在）
   ```

2. **按类型分类文件**（智能体/技能/命令）

3. **加载参考模板**（对比审计模式）：
   ```
   guide/examples/agents/     （基准文件）
   guide/examples/skills/     （基准文件）
   guide/examples/commands/   （基准文件）
   ```

### 第二阶段：评分引擎

从 `scoring/criteria.yaml` 加载评分标准：

```yaml
agents:
  max_points: 32
  categories:
    identity:
      weight: 3
      criteria:
        - id: A1.1
          name: "Clear name"
          points: 3
          detection: "frontmatter.name exists and is descriptive"
        # ... （共 16 项标准）
```

对每个文件执行：
1. 解析 frontmatter（YAML）
2. 提取内容章节
3. 运行检测模式（正则表达式、关键词搜索）
4. 计算评分：`(获得分数 / 满分) × 100`
5. 分配等级（A-F）

### 第三阶段：对比分析（仅对比审计模式）

对每个项目文件：
1. 根据描述相似度找到最接近的模板
2. 按标准项逐一对比评分
3. 识别差距：`模板分数 - 项目分数`
4. 标记显著差距（差距 >10 分）

**示例**：
```
项目文件：.claude/agents/debugging-specialist.md（评分：78%，等级 C）
最接近模板：examples/agents/debugging-specialist.md（评分：94%，等级 A）

差距分析：
- 防幻觉措施：-2 分（模板有，项目缺失）
- 边界情况文档：-1 分（模板有 5 个示例，项目只有 1 个）
- 集成文档：-1 分（模板引用了 3 个技能，项目无引用）

总差距：16 分（解释了 C 级与 A 级的差距）
```

### 第四阶段：报告生成

**Markdown 报告**（`audit-report.md`）：
- 汇总表格（整体 + 按类型分类）
- 各文件评分及主要问题
- 每个文件的详细分项（可折叠）
- 优先级建议

**JSON 输出**（`audit-report.json`）：
```json
{
  "metadata": {
    "project_path": "/path/to/project",
    "audit_date": "2026-02-07",
    "mode": "full",
    "version": "1.0.0"
  },
  "summary": {
    "overall_score": 82.5,
    "overall_grade": "B",
    "total_files": 15,
    "production_ready_count": 10,
    "production_ready_percentage": 66.7
  },
  "by_type": {
    "agents": { "count": 5, "avg_score": 85.2, "grade": "B" },
    "skills": { "count": 8, "avg_score": 78.9, "grade": "C" },
    "commands": { "count": 2, "avg_score": 92.0, "grade": "A" }
  },
  "files": [
    {
      "path": ".claude/agents/debugging-specialist.md",
      "type": "agent",
      "score": 78.1,
      "grade": "C",
      "points_obtained": 25,
      "points_max": 32,
      "failed_criteria": [
        {
          "id": "A2.4",
          "name": "Anti-hallucination measures",
          "points_lost": 2,
          "recommendation": "Add section on source verification"
        }
      ]
    }
  ],
  "top_issues": [
    {
      "issue": "Missing error handling",
      "affected_files": 8,
      "impact": "Runtime failures unhandled",
      "priority": "high"
    }
  ]
}
```

### 第五阶段：修复建议（可选）

针对每项不达标标准，生成**可执行的修复方案**：

```markdown
### 文件：.claude/agents/debugging-specialist.md
**问题**：缺少防幻觉措施（损失 2 分）

**修复方案**：
在"方法论"之后添加以下章节：

## 来源验证

- 技术性断言必须注明来源
- 使用表述："根据 [文档]……"、"基于 [工具输出]……"
- 如不确定，声明："我没有关于……的已验证信息"
- 禁止编造：统计数据、版本号、API 签名、堆栈跟踪

**检测方法**：Grep 关键词："verify"、"cite"、"source"、"evidence"
```

---

## 评分标准

完整定义见 `scoring/criteria.yaml`。概览如下：

### 智能体（满分 32 分）

| 类别 | 权重 | 标准数量 | 最高分 |
|----------|--------|----------------|------------|
| 身份标识 | 3 倍 | 4 | 12 |
| 提示词质量 | 2 倍 | 4 | 8 |
| 验证 | 1 倍 | 4 | 4 |
| 设计 | 2 倍 | 4 | 8 |

**关键标准**：
- 名称清晰（3 分）：不使用"agent1"等泛型命名
- 描述含触发条件（3 分）：包含"when"/"use"等触发词
- 角色定义（2 分）：包含"You are..."声明
- 3 个以上示例（1 分）：用法场景已文档化
- 单一职责（2 分）：专注明确，非"通用型"

### 技能（满分 32 分）

| 类别 | 权重 | 标准数量 | 最高分 |
|----------|--------|----------------|------------|
| 结构 | 3 倍 | 4 | 12 |
| 内容 | 2 倍 | 4 | 8 |
| 技术 | 1 倍 | 4 | 4 |
| 设计 | 2 倍 | 4 | 8 |

**关键标准**：
- 有效的 SKILL.md（3 分）：命名规范
- 名称有效（3 分）：小写字母，1-64 个字符，不含空格
- 方法论说明（2 分）：包含工作流程章节
- 无硬编码路径（1 分）：不含 `/Users/`、`/home/`
- 触发条件明确（2 分）：包含"使用时机"章节

### 命令（满分 20 分）

| 类别 | 权重 | 标准数量 | 最高分 |
|----------|--------|----------------|------------|
| 结构 | 3 倍 | 4 | 12 |
| 质量 | 2 倍 | 4 | 8 |

**关键标准**：
- 有效的 frontmatter（3 分）：包含 name + description
- 参数提示（3 分）：如使用 `$ARGUMENTS`
- 分步骤工作流程（3 分）：有编号章节
- 错误处理（2 分）：提及失败场景

---

## 检测模式

### Frontmatter 解析

```python
import yaml
import re

def parse_frontmatter(content):
    match = re.search(r'^---\n(.*?)\n---', content, re.DOTALL)
    if match:
        return yaml.safe_load(match.group(1))
    return None
```

### 关键词检测

```python
def has_keywords(text, keywords):
    text_lower = text.lower()
    return any(kw in text_lower for kw in keywords)

# 示例
has_trigger = has_keywords(description, ['when', 'use', 'trigger'])
has_error_handling = has_keywords(content, ['error', 'failure', 'fallback'])
```

### 重叠检测（去重检查）

```python
def jaccard_similarity(text1, text2):
    words1 = set(text1.lower().split())
    words2 = set(text2.lower().split())
    intersection = words1 & words2
    union = words1 | words2
    return len(intersection) / len(union) if union else 0

# 相似度 > 0.5（50% 关键词重叠）时标记
if jaccard_similarity(desc1, desc2) > 0.5:
    issues.append("High overlap with another file")
```

### Token 估算（近似值）

```python
def estimate_tokens(text):
    # 粗略估算：1 token ≈ 0.75 个单词
    word_count = len(text.split())
    return int(word_count * 1.3)

# 检查预算
tokens = estimate_tokens(file_content)
if tokens > 5000:
    issues.append("File too large (>5K tokens)")
```

---

## 行业背景

**来源**：LangChain Agent Report 2026（公开报告，第 14-22 页）

**主要发现**：
- **29.5%** 的组织在没有系统性评估的情况下部署智能体
- **18%** 将"智能体缺陷"列为首要挑战
- **仅 12%** 使用自动化质量检查（88% 依赖手动或不检查）
- **43%** 反映难以长期维持智能体质量
- **主要问题**：幻觉（31%）、错误处理不当（28%）、触发条件不清晰（22%）

**启示**：
1. **自动化缺口**：大多数团队依赖手动清单（规模扩大时易出错）
2. **质量债务**：未经验证就部署的智能体会积累技术债务
3. **维护负担**：43% 的团队长期维护困难（缺少追踪系统）

**本技能的解决方案**：
- 自动化：用量化评分替代手动清单
- 追踪：JSON 输出支持随时间推移的趋势分析
- 标准化：80% 阈值提供清晰的生产准入门槛

---

## 输出示例

### 快速审计（前 5 项标准）

```markdown
# 快速审计：智能体/技能/命令

**文件数**：15（5 个智能体，8 个技能，2 个命令）
**关键问题**：3 个文件未通过前 5 项标准

## 前 5 项标准（通过/失败）

| 文件 | 名称有效 | 有触发条件 | 错误处理 | 无硬编码路径 | 有示例 |
|------|------------|--------------|----------------|--------------------|----------|
| agent1.md | ✅ | ✅ | ❌ | ✅ | ❌ |
| skill2/ | ✅ | ❌ | ✅ | ❌ | ✅ |

## 需要处理的问题

1. **添加错误处理**：5 个文件
2. **移除硬编码路径**：3 个文件
3. **添加使用示例**：4 个文件
```

### 完整审计

完整结构见上方第四阶段：报告生成。

### 对比审计（完整 + 基准对比）

```markdown
# 对比审计报告

## 项目 vs 模板

| 文件 | 项目评分 | 模板评分 | 差距 | 主要缺失项 |
|------|---------------|----------------|-----|-------------|
| debugging-specialist.md | 78%（C） | 94%（A） | -16 分 | 防幻觉措施、边界情况 |
| testing-expert/ | 85%（B） | 91%（A） | -6 分 | 集成文档 |

## 建议

关注以下差距以达到模板质量：
1. **防幻觉措施**（8 个文件）：添加来源验证章节
2. **边界情况文档**（5 个文件）：添加失败场景示例
3. **集成文档**（4 个文件）：列出兼容的智能体/技能
```

---

## 使用方法

### 基本用法（完整审计）

```bash
# 在 Claude Code 中
Use skill: audit-agents-skills

# 指定路径
Use skill: audit-agents-skills for ~/projects/my-app
```

### 带选项使用

```bash
# 快速审计（速度快）
Use skill: audit-agents-skills with mode=quick

# 对比审计（基准分析）
Use skill: audit-agents-skills with mode=comparative

# 生成修复建议
Use skill: audit-agents-skills with fixes=true

# 自定义输出路径
Use skill: audit-agents-skills with output=~/Desktop/audit.json
```

### 仅输出 JSON

```bash
# 用于程序化集成
Use skill: audit-agents-skills with format=json output=audit.json
```

---

## 与 CI/CD 集成

### Pre-commit 钩子

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 对已变更的智能体/技能/命令文件运行快速审计
changed_files=$(git diff --cached --name-only | grep -E "^\.claude/(agents|skills|commands)/")

if [ -n "$changed_files" ]; then
    echo "正在对已变更文件运行快速审计..."
    # 运行审计（需要 Claude Code CLI 封装）
    # 如有文件评分 <80%，以退出码 1 退出
fi
```

### GitHub Actions

```yaml
name: 审计智能体/技能
on: [pull_request]
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: 运行质量审计
        run: |
          # 运行审计技能
          # 解析 JSON 输出
          # 如 overall_score < 80 则失败
```

---

## 命令 vs 技能对比

| 维度 | 命令（`/audit-agents-skills`） | 技能（本文件） |
|--------|----------------------------------|-------------------|
| **范围** | 仅当前项目 | 多项目、支持对比 |
| **输出** | Markdown 报告 | Markdown + JSON |
| **速度** | 快速（5-10 分钟） | 较慢（含对比时 10-20 分钟） |
| **深度** | 标准 16 项标准 | 相同 + 基准分析 |
| **修复建议** | 通过 `--fix` 参数 | 内置建议 |
| **程序化** | 终端输出 | JSON 支持 CI/CD 集成 |
| **适用场景** | 日常快速检查、开发工作流 | 深度审计、质量追踪 |

**建议**：日常检查使用命令，发布门控和质量追踪使用技能。

---

## 维护

### 更新标准

编辑 `scoring/criteria.yaml`：
```yaml
agents:
  categories:
    identity:
      criteria:
        - id: A1.5  # 新增标准
          name: "API versioning specified"
          points: 3
          detection: "mentions API version or compatibility"
```

版本升级：标准变更时，在 frontmatter 中递增 `version`。

### 添加新文件类型

若要支持新文件类型（如"工作流"）：
1. 在 `scoring/criteria.yaml` 中添加：
   ```yaml
   workflows:
     max_points: 24
     categories: [...]
   ```
2. 更新检测逻辑（文件路径模式）
3. 更新报告模板

---

## 相关资源

- **命令版本**：`.claude/commands/audit-agents-skills.md`
- **智能体验证清单**：指南第 4921 行（手动 16 项标准）
- **技能验证**：指南第 5491 行（规范文档）
- **参考模板**：`examples/agents/`、`examples/skills/`、`examples/commands/`

---

## 更新日志

**v1.0.0**（2026-02-07）：
- 首次发布
- 16 项标准框架（智能体/技能/命令）
- 3 种审计模式（快速/完整/对比）
- JSON + Markdown 双格式输出
- 修复建议
- 行业背景（LangChain 2026 报告）

---

**技能已就绪**：`audit-agents-skills`
