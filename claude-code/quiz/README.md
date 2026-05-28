> 📚 **AI Spark Wiki** · Claude Code 知识库

# Claude Code 知识测验

通过交互式多选题测试你对 Claude Code 的理解。

[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Linux%20%7C%20Windows-blue.svg)]()

## 快速开始

```bash
# 进入 quiz 目录
cd quiz

# 安装依赖
npm install

# 运行测验
npm start

# 或直接运行
node src/index.js
```

## 功能特色

- **4 种用户画像**：初级、高级、资深用户、产品经理
- **10 个主题分类**：从快速上手到高级模式全覆盖
- **159 道精选题目**：考察实用知识，而非死记硬背
- **即时反馈**：从错题中学习，附带详细解析
- **文档链接**：直接引用指南对应章节
- **得分追踪**：了解强项和薄弱环节
- **会话持久化**：历史记录保存至 `~/.claude-quiz/`
- **跨平台支持**：兼容 macOS、Linux 和 Windows

## 使用方法

### 交互模式（推荐）

```bash
npm start
```

系统会提示你：
1. 选择用户画像（初级/高级/资深用户/产品经理）
2. 选择主题（全部或指定章节）
3. 用 A/B/C/D 回答问题

### 命令行选项

```bash
node src/index.js [options]

Options:
  -p, --profile <type>   预选用户画像 (junior|senior|power|pm)
  -t, --topics <list>    指定测验章节 (1-10，逗号分隔)
  -c, --count <n>        限制题目数量 (1-50)
  -d, --dynamic          通过 claude -p 启用动态题目生成
  -h, --help             显示帮助信息
  -v, --version          显示版本号
```

### 示例

```bash
# 交互模式
npm start

# 高级画像，默认主题
node src/index.js -p senior

# 资深用户，指定主题（智能体、Hooks、MCP）
node src/index.js -p power -t 4,7,8

# 快速 10 题测验
node src/index.js -c 10

# 初级画像 + 动态题目生成
node src/index.js -p junior -d
```

## 示例会话

以下是一次典型测验会话的样子：

```
============================================================
   CLAUDE CODE KNOWLEDGE QUIZ
   Master Claude Code: The Complete Guide
============================================================

? Select your profile: Senior Developer (40 min to mastery)
? Select topics to quiz: Custom selection...
? Select topics (space to toggle, enter to confirm):
  ◉ [2] Core Concepts
  ◉ [4] Agents
  ◉ [7] Hooks

------------------------------------------------------------
Starting quiz: 20 questions for senior profile
------------------------------------------------------------

------------------------------------------------------------
Question 1/20 [Core Concepts]

At what context percentage should you use /compact?

  A) 0-50%
  B) 50-70%
  C) 70-90%
  D) Only at 100%

? Your answer: C

✓ CORRECT!

Progress: █░░░░░░░░░░░░░░░░░░░ 1/20 | Score: 1/1 (100%)

------------------------------------------------------------
Question 2/20 [Hooks]

What exit code should a PreToolUse hook return to BLOCK an operation?

  A) 0
  B) 1
  C) 2
  D) -1

? Your answer: A

✗ INCORRECT. The correct answer is C) 2

Explanation:
Exit code 2 blocks the operation. Exit code 0 allows it to proceed.
Other exit codes are treated as errors and logged but don't block.

See: https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide/blob/main/guide/ultimate-guide.md#72-creating-hooks
     (Line 4164)

Progress: ██░░░░░░░░░░░░░░░░░░ 2/20 | Score: 1/2 (50%)
? Continue to next question? Yes

... (more questions) ...

============================================================
   QUIZ COMPLETE
============================================================

Overall Score: 16/20 (80%)

By Category:
  Core Concepts       6/7  (86%)  [████████░░]
  Agents              5/7  (71%)  [███████░░░]
  Hooks               5/6  (83%)  [████████░░]

Weak Areas (< 75%):
  - Agents: Review section 4 in the guide

Recommended Reading:
  4. guide/ultimate-guide.md#4-agents

Time: 9 minutes 2 seconds

------------------------------------------------------------
? What would you like to do? (Use arrow keys)
❯ Retry wrong questions only
  New quiz (different questions)
  Exit
```

## 用户画像

| 画像 | 题目数量 | 重点章节 |
|---------|-----------|-------------|
| **初级开发者** | 15 | 第 1-3、6 章（基础必备） |
| **高级开发者** | 20 | 第 2-4、7、9 章（架构、自动化） |
| **资深用户** | 25 | 全部章节 |
| **产品经理** | 10 | 第 1-3 章（概念概览） |

## 主题分类

| # | 主题 | 核心概念 |
|---|-------|--------------|
| 1 | 快速上手与安装 | 安装、首次工作流、核心命令 |
| 2 | 核心概念 | 上下文管理、计划模式、交互循环 |
| 3 | 记忆与设置 | CLAUDE.md、.claude/ 文件夹、权限 |
| 4 | 智能体 | 自定义智能体、专业化、编排 |
| 5 | Skills（技能模块） | 可复用知识、技能组合 |
| 6 | 命令 | 斜杠命令、自定义命令 |
| 7 | Hooks（钩子） | 事件系统、安全钩子、退出码 |
| 8 | MCP 服务器 | Context7、Serena、Sequential、插件 |
| 9 | 高级模式 | 三位一体、CI/CD、组合 |
| 10 | 参考速查 | 快捷键、故障排查、日常工作流 |

## 测验流程

```
┌─────────────────────────────────────────────┐
│          CLAUDE CODE KNOWLEDGE QUIZ         │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│  选择用户画像（初级/高级/资深/产品经理）       │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│  选择主题（全部或指定 1-10）                  │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│  第 1/20 题 [类别]                           │
│                                             │
│  以下场景中推荐的操作是什么？                 │
│                                             │
│    A) 选项 A                                │
│    B) 选项 B                                │
│    C) 选项 C                                │
│    D) 选项 D                                │
└─────────────────────────────────────────────┘
                     │
            ┌────────┴────────┐
            ▼                 ▼
     ┌──────────┐      ┌──────────────────┐
     │  正确！  │      │ 错误             │
     │          │      │ 正确答案：C      │
     │          │      │ 解析...          │
     │          │      │ 参见：guide#...  │
     └──────────┘      └──────────────────┘
            │                 │
            └────────┬────────┘
                     ▼
              （下一题）
                     │
                     ▼
┌─────────────────────────────────────────────┐
│           测验完成                          │
│                                             │
│  总得分：16/20 (80%)                        │
│                                             │
│  按类别：                                   │
│    核心概念：  6/7  (86%)  [████████░░]     │
│    智能体：    5/7  (71%)  [███████░░░]     │
│    Hooks：     5/6  (83%)  [████████░░]     │
│                                             │
│  薄弱环节：智能体（复习第 4 章）             │
└─────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│  [R] 只重做错题                             │
│  [N] 新测验（不同题目）                     │
│  [E] 退出                                   │
└─────────────────────────────────────────────┘
```

## 会话持久化

测验会话自动保存至 `~/.claude-quiz/`：

```
~/.claude-quiz/
  sessions/
    2026-01-12_143052.json   # 单次会话数据
    2026-01-12_150623.json
  stats.json                  # 汇总统计
```

### 会话数据包含：
- 已选用户画像和主题
- 已作答题目（正确/错误/跳过）
- 用时
- 分类得分明细
- 错题及正确答案

## 动态题目生成

启用 `--dynamic` 标志且已安装 Claude CLI 时，测验可通过 `claude -p` 实时生成额外题目：

```bash
node src/index.js -d
```

此功能需要：
- 已安装 Claude CLI（`npm install -g @anthropic-ai/claude-code`）
- 已配置有效的 API Key

在以下情况下动态题目会补充静态题库：
- 所选主题的静态题库已耗尽
- 用户请求新题目
- 重复测验需要多样性

## 贡献题目

欢迎提交题目！格式请参见 [`templates/question-template.yaml`](./templates/question-template.yaml)。

### 质量标准

**好题目考察：**
- 实际应用（"什么情况下应该……"）
- 决策判断（"以下哪种方法最适合……"）
- 理解深度（"为什么 Claude 会……"）
- 故障排查（"如果遇到……你会怎么做"）

**避免：**
- 纯记忆（"确切的命令语法是……"）
- 死背（"列出所有键盘快捷键……"）
- 可能变化的版本特定细节

### 提交题目

1. Fork 本仓库
2. 将题目添加到对应的 `questions/XX-category.yaml` 文件
3. 严格遵循模板格式
4. 确保 `doc_reference` 指向有效章节
5. 提交 PR 并说明新增题目内容

## 故障排查

### "No questions available"

- 检查所选主题在 `questions/` 目录中是否有题目文件
- 验证 YAML 文件格式是否正确（用 YAML 验证器检查）
- 尝试选择其他主题或"全部主题"

### "Cannot find module 'yaml'"

```bash
cd quiz
npm install
```

### 测验卡在输入处

- 确认在交互式终端中运行
- 若卡住请按回车
- 用 Ctrl+C 退出并重启

## 许可证

与父仓库相同：[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

---

**所属项目**：[claude-code-ultimate-guide](https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide)
