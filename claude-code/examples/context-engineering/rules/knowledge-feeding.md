> 📚 **AI Spark Wiki** · Claude Code 知识库

# 知识输入协议

上下文工程不是一次性配置——它会随着时间积累价值，Claude 会不断学习哪些方法对你的项目最有效。本文件定义了何时以及如何将这些学习成果回写到 `CLAUDE.md` 中。

## 何时进行知识输入

在以下类型的会话结束时运行本协议：

- 完成了一个功能或有实质意义的代码变更
- 发现了一个效果良好、应成为标准做法的新模式
- 遇到了需要纠正的错误（尤其是重复出现的错误）
- 做出了会影响未来工作的架构决策
- 确立了对某个库、工具或方法的使用偏好

以下类型的轻量会话可跳过：拼写修正、文档编辑、小型配置调整。

## 知识输入提示词

在符合条件的会话结束时，将以下内容粘贴给 Claude：

```
Before we close this session, run a knowledge feed:

1. What patterns did we establish that should become permanent rules?
2. What did I correct or redirect that should be a rule to prevent recurrence?
3. Were any architectural decisions made that CLAUDE.md should record?
4. Is there anything in CLAUDE.md that this session proved wrong or outdated?

Output only high-signal items (3-5 max). Use the knowledge feed format below.
Skip anything obvious or already covered.
```

## 知识输入输出格式

Claude 应按以下结构输出发现内容：

```markdown
## Knowledge Feed — [YYYY-MM-DD]

### New Pattern (if applicable)
**What**: [One sentence describing the pattern]
**Why**: [Why this is the right approach for this project]
**Rule to add**:
> [Exact text to paste into CLAUDE.md, ready to copy]

### Anti-Pattern Found (if applicable)
**What happened**: [What Claude did wrong or what was corrected]
**Why it's wrong here**: [Project-specific reason, not generic best practice]
**Rule to add**:
> Never: [specific behavior to avoid and why]

### Architecture Decision (if applicable)
**Decision**: [What was decided]
**Rationale**: [Why — especially if it goes against common practice]
**Rule to add**:
> [Exact text to paste into CLAUDE.md]

### Stale Rule to Remove (if applicable)
**Rule**: [Current rule in CLAUDE.md]
**Why remove**: [What changed that makes this obsolete]
```

## 集成工作流

收到知识输入内容后：

1. **添加前先审查**——并非所有模式都能推广。问自己："新团队成员需要知道这个吗？还是它只对这次会话的上下文有意义？"
2. **将相关规则复制**到 `CLAUDE.md` 的对应章节
3. **删除 Claude 标记为过时的规则**
4. **提交更新**，附上有意义的提交信息：

```bash
git add CLAUDE.md
git commit -m "context: [short description of what was learned]"

# Examples:
# context: add zod validation rule after form refactor
# context: remove Redux rule — migrated to Zustand
# context: document payment webhook idempotency pattern
```

## 质量过滤

在将任何规则添加到 `CLAUDE.md` 之前，检查以下几点：

- **是否特定于本项目？** 通用最佳实践不属于这里——Claude 本来就知道。
- **是否可执行？** "注意异步代码"毫无意义。"始终检查订单状态机中的竞态条件"才是有用的。
- **是否已有覆盖？** 添加前先搜索类似规则，重复规则会降低执行力。
- **6 个月后还成立吗？** 避免与临时状态绑定的规则（"我们正在迁移到 X，所以暂时别用 Y"）。

## 责任归属

知识输入在作为团队习惯而非某人专属工作时效果最佳。任何与 Claude 协作的团队成员都可以贡献知识输入。负责审查 `CLAUDE.md` 变更 PR 的人负责质量过滤。
