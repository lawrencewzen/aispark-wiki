> 📚 **AI Spark Wiki** · Claude Code 知识库

# AI 指令 — {{DEVELOPER_NAME}}
<!-- Generated: {{GENERATED_DATE}} | OS: {{OS}} | Tool: {{TOOL}} -->
<!-- 请勿手动编辑 — 由 profile + modules 自动生成 -->
<!-- 如需更新：编辑 profiles/{{DEVELOPER_SLUG}}.yaml 或 modules/ 目录，然后运行： -->
<!-- npx ts-node sync-ai-instructions.ts {{DEVELOPER_SLUG}} -->

---

## 项目上下文

{{MODULE:core-standards}}

---

## Git 工作流

{{MODULE:git-workflow}}

---

## 测试

{{MODULE:test-conventions}}

---

{{#if typescript}}
## TypeScript 规则

{{MODULE:typescript-rules}}

---
{{/if}}

{{#if python}}
## Python 规则

{{MODULE:python-rules}}

---
{{/if}}

## 环境与路径

{{MODULE:{{OS}}-paths}}

---

{{#if cursor}}
## Cursor 专属指令

{{MODULE:cursor-rules}}

---
{{/if}}

{{#if windsurf}}
## Windsurf 专属指令

{{MODULE:windsurf-rules}}

---
{{/if}}

## 沟通风格

{{#if verbose}}
对每个决策提供详细解释。展示已考虑的备选方案。包含推理过程。
{{/if}}
{{#if concise}}
简洁表达。每点一句话。跳过显而易见的细节。
{{/if}}
{{#if terse}}
最简输出。尽量只给代码。除非被问及，否则不做解释。
{{/if}}
