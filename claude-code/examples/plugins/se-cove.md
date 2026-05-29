> 📚 **AI Spark Wiki** · Claude Code 知识库

---
plugin: chain-of-verification
marketplace: vertti/se-cove-claude-plugin
version: 1.1.1
license: MIT
research: arXiv:2309.11495 (ACL 2024 Findings)
---

# SE-CoVe：验证链

Meta 验证链方法论在 Claude Code 中的软件工程适配版本。

## 研究基础

**论文**：《验证链减少大语言模型的幻觉》（Chain-of-Verification Reduces Hallucination in Large Language Models）
**作者**：Dhuliawala 等（Meta AI）
**发表**：ACL 2024 Findings
**来源**：[arXiv:2309.11495](https://arxiv.org/abs/2309.11495) | [ACL Anthology](https://aclanthology.org/2024.findings-acl.212/)

## 工作原理

五阶段流水线，确保独立验证：

1. **基线**：生成初始方案
2. **规划器**：从方案声明中创建验证问题
3. **执行器**：独立回答问题（完全不看基线方案）
4. **综合器**：比较发现，识别差异
5. **输出**：生成经过验证的最终方案

**核心创新**：验证器在不访问草稿代码的情况下运行，从而防止确认偏差。

## 性能指标

来自 Meta 研究论文的数据（Llama 65B 模型）：

| 任务类型 | 指标 | 提升 | 计算成本 |
|-----------|--------|-------------|-------------------|
| 传记生成 | FACTSCORE | +28%（55.9→71.4） | 输出量减少 26%（16.6→12.3 条事实） |
| 闭卷问答 | F1 分数 | +23%（0.39→0.48） | token 消耗约 2 倍 |
| 列表类问题 | 精确率 | +112%（0.17→0.36） | 总答案数减少 |

**来源**：Dhuliawala 等，ACL 2024 Findings（表 1，第 4.3 节）

**核心洞察**：更高准确率的代价是计算量增加和输出量减少。

## 适用场景

### 推荐使用

- **关键代码评审**：架构决策、安全敏感代码
- **复杂调试**：多组件故障分析
- **API/库集成**：正确性优先于速度
- **可接受 2 倍成本**：token 预算允许为质量付出溢价

### 不推荐使用

- **琐碎改动**：简单修复、格式调整、错别字
- **探索性编码**：快速原型开发、实验性工作
- **token 预算紧张**：成本是主要约束时
- **需要全面输出**：需要所有事实而非仅准确子集时

## 安装

```bash
# 添加插件市场
/plugin marketplace add vertti/se-cove-claude-plugin

# 安装插件（单独执行此命令）
/plugin install chain-of-verification
```

**注意**：命令需分开粘贴执行（Claude Code 市场限制）。

## 使用方式

```bash
# 调用验证
/chain-of-verification:verify <你的问题>

# 支持自动补全
/ver<Tab>
```

## 局限性

来自研究论文（第 6 节）：

1. **不是万能药**：减少幻觉，但不能完全消除
2. **计算成本**：token 使用量约为基线生成的 2 倍（基于实现估算）
3. **输出量权衡**：生成的结果更少但更准确
4. **模型特异性**：在 Llama 65B 上测试；能否推广至 GPT-4/Claude/Sonnet 尚未验证
5. **任务依赖性**：不同任务类型的性能差异显著（23-112%）
6. **仅针对事实性幻觉**：不解决错误的推理步骤或主观意见

## 源代码

- **GitHub**：[vertti/se-cove-claude-plugin](https://github.com/vertti/se-cove-claude-plugin)
- **版本**：1.1.1（2026-01-23）
- **许可证**：MIT
- **作者**：Janne Sinivirta

## 相关资源

- 主指南章节：[插件系统](../../guide/ultimate-guide.md#85-plugin-system)
- 方法论：[多智能体编排](../../guide/core/methodologies.md#multi-agent-orchestration)
- 验证循环：[自主迭代](../../guide/core/methodologies.md#verification-loops)
