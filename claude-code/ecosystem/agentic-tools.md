> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Agent Tools: Beyond Claude Code"
description: "Comparative guide to terminal coding agents, autonomous coders, and multi-agent frameworks. Covers Hermes Agent, Codex CLI, Aider, Devin, SWE-agent, CrewAI, LangGraph, and AutoGen with a decision framework."
tags: [agents, hermes, codex-cli, aider, devin, swe-agent, crewai, langgraph, autogen, comparison]
---

# Agent Tools: Beyond Claude Code

Claude Code is one tool in a field that has expanded dramatically since 2024. Dozens of agent frameworks, autonomous coders, and multi-agent systems have shipped, each with different trade-offs. This page maps that field so you can decide when Claude Code is the right call, and when something else fits better.

**What this page covers**: terminal coding agents, autonomous coders, multi-agent orchestration frameworks, and agent orchestration tooling. Claude Code's own multi-agent capabilities (agent teams, event-driven workflows, programmatic usage) are documented separately, linked throughout.

**What it does not cover**: GUI-based AI coding IDEs (Cursor, Windsurf, Cline), which are covered in [AI Ecosystem §6](./ai-ecosystem.md#section-6). Multi-Claude orchestration tools (Gas Town, multiclaude, Conductor desktop app) are in [Third-Party Tools: Multi-Agent Orchestration](./third-party-tools.md#multi-agent-orchestration).

---

## The Spectrum

Agent tools fall on a spectrum from interactive to autonomous:

```
Interactive pair programmer
  Claude Code, Codex CLI, Aider, Goose
        |
  Hermes Agent (interactive + scheduled + messaging gateways)
        |
Autonomous issue fixer
  SWE-agent, Devin, claude -p in CI
        |
Multi-agent framework (build your own)
  CrewAI, LangGraph, AutoGen/MAF
```

**Interactive agents**: you stay in the loop, approve actions, redirect the agent. Best for daily coding, debugging, and exploratory work where requirements shift.

**Autonomous agents**: you assign a task and come back to a result. Best for well-specified, bounded tasks: fix this bug, implement this spec, review this PR. The quality of the task description determines the quality of the output more than the agent choice.

**Multi-agent frameworks**: libraries for building custom agent systems. Not coding tools themselves. You use LangGraph to build an agent, not to write code.

---

## Section 1: Terminal Coding Agents

These tools do what Claude Code does: sit in your terminal, read your codebase, write code, run commands. The differences are in model support, cost model, and specific capabilities.

---

### 1.1 Codex CLI (OpenAI)

OpenAI's direct answer to Claude Code. Launched April 2025, built in Rust, open-sourced under Apache 2.0.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [openai/codex](https://github.com/openai/codex) |
| **Stars** | 86,200+ (May 2026) |
| **Install** | `npm install -g @openai/codex` |
| **Language** | Rust (96%) |
| **License** | Apache 2.0 |
| **Version** | v0.134.0 (May 26, 2026) |
| **Releases** | 800+ since April 2025 |
| **Contributors** | 400+ |

#### What Is Codex CLI?

A terminal AI agent for writing, editing, and running code, built on OpenAI's model family. The architecture mirrors Claude Code closely: you describe a task, the agent reads files, makes edits, runs tests, and iterates. The main difference is the model provider: Codex CLI talks to GPT-4o, o3, o4-mini, and other OpenAI models, not Claude.

ChatGPT Pro and Team subscribers get Codex CLI usage included in their plan, making it a zero-marginal-cost tool for teams already paying for OpenAI.

#### Claude Code vs Codex CLI

| Aspect | Claude Code | Codex CLI |
|--------|-------------|-----------|
| **Models** | Claude 3.5/4 family only | GPT-4o, o3, o3-mini, o4-mini, plus future OpenAI models |
| **Language** | TypeScript | Rust |
| **License** | Open source | Apache 2.0 |
| **Subscription** | Anthropic Claude Max ($20-$200/mo) | OpenAI ChatGPT Pro/Team ($20-$30/mo) |
| **MCP Support** | Native, growing ecosystem | MCP compatible |
| **Release cadence** | Weekly | Very high (800+ releases in 13 months) |
| **Memory** | CLAUDE.md + Auto Memory | AGENTS.md convention |
| **Skills/Hooks** | Full system | Compatible with agentskills.io standard |

#### When to Choose Codex CLI

Good fit if you are already on a ChatGPT Pro or Team plan and want to avoid a second subscription. Also the right call if you prefer GPT-4o or o3 for specific tasks (reasoning, long-context analysis) and want a terminal agent that uses those models natively.

Poor fit if your team has invested in Claude Code workflows, CLAUDE.md files, and Anthropic-specific patterns. The cognitive cost of context-switching between two agent environments is real.

#### Quick Start

```bash
npm install -g @openai/codex
export OPENAI_API_KEY=sk-...
codex
```

OpenAI's [Codex docs](https://github.com/openai/codex/blob/main/README.md) cover setup in detail.

---

### 1.2 Hermes Agent (formerly OpenClaw)

The most starred open-source agent framework as of May 2026. Created by Nous Research, the AI lab known for its Hermes series of fine-tuned models. Was called OpenClaw until late 2025, when it rebranded on Anthropic reinstating subscription support.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) |
| **Stars** | 170,000+ (May 2026) |
| **Install** | `pip install hermes-agent` or `curl -sSL install.hermes-agent.dev \| sh` |
| **Language** | Python (89%), TypeScript (8%) |
| **License** | MIT |
| **Version** | v0.14.0 (May 16, 2026) |
| **Release cadence** | Weekly (v0.10 Apr 16 → v0.14 May 16) |
| **Contributors** | 215+ |
| **Creator** | Nous Research (Teknium, @teknium1) |

#### What Is Hermes Agent?

A self-improving terminal agent that works with 200+ LLM providers, runs on any platform, and connects to 22 messaging platforms (Telegram, Discord, Slack, WhatsApp, Signal, Teams, LINE, SimpleX, and more). The distinguishing feature is its learning loop: after completing tasks, Hermes analyzes what worked, extracts reusable patterns, and generates skills automatically. Each session makes the agent marginally better at your specific workflows.

The OpenClaw history matters for two reasons. First, the migration path is clean: `hermes-agent` imports OpenClaw memories, skills, and settings during setup, so switching costs are low. Second, the Anthropic billing controversy from early 2026 was specifically about OpenClaw/Hermes being used on Claude Max subscriptions without proper programmatic billing attribution. Anthropic now explicitly includes Hermes in the programmatic usage bucket (see [Billing: Programmatic vs Interactive](../ultimate-guide.md#the-interactiveprogrammatic-billing-split-effective-june-15-2026)).

#### Claude Code vs Hermes Agent

| Aspect | Claude Code | Hermes Agent |
|--------|-------------|--------------|
| **Models** | Claude only | 200+ via OpenRouter, OpenAI, Anthropic, HuggingFace, local |
| **Self-improvement** | Each session starts fresh | Skills auto-generated from recurring patterns |
| **Messaging** | Terminal + IDE | Terminal + 22 chat platforms |
| **Cron scheduling** | Routines (Anthropic cloud) | Built-in cron, runs locally |
| **Billing** | Subscription or API | Pay your LLM provider directly |
| **Agent SDK** | Anthropic-specific | `ctx.llm` plugin for any provider |
| **Skills** | SKILL.md system | Skills Hub (agentskills.io) + auto-generated |
| **Memory** | CLAUDE.md + Auto Memory | Cross-session persistent memory, agent-curated |

#### When to Choose Hermes Agent

The model-agnostic case is the strongest argument. If you want to run Claude for code generation, GPT-4o for specific reasoning tasks, and a local model (via Ollama) for offline work, Hermes handles all three in a single agent. Claude Code cannot.

The self-improving loop is genuinely differentiated. Over 30-40 sessions on the same codebase, Hermes builds a library of skills specific to your project's patterns. This compounds in a way that static CLAUDE.md files do not, though the comparison is complex because CLAUDE.md is human-authored and intentional while Hermes skills are machine-generated.

The 22 messaging platform integrations are useful for teams that want to interact with their agent via Telegram or Slack rather than a terminal. Not a priority for most developers, but critical for some workflows.

Poor fit if you are invested in Anthropic's ecosystem (Claude Max subscription, Routines, the Agent SDK). Running Hermes with Claude models hits the programmatic billing bucket, meaning your $200/mo Max subscription's $200 credit gets consumed by both interactive terminal use and Hermes API calls. Factor that in.

#### Quick Start

```bash
pip install hermes-agent

# Or one-line installer
curl -sSL install.hermes-agent.dev | sh

# Import from OpenClaw if migrating
hermes import --from openclaw

# Start
hermes
```

---

### 1.3 Aider

The original terminal AI pair programmer. Launched in 2023 by Paul Gauthier before Claude Code existed, Aider established many of the conventions that later tools adopted: direct file editing, automatic git commits, multi-file context windows.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [Aider-AI/aider](https://github.com/Aider-AI/aider) |
| **Stars** | 45,400+ (May 2026) |
| **Install** | `pip install aider-install && aider-install` |
| **Language** | Python (80%) |
| **License** | Apache 2.0 |
| **Creator** | Paul Gauthier (paul-gauthier) |
| **PyPI downloads** | 5.3M+ |

#### What Is Aider?

A Python-based coding assistant that edits files in your local git repo and auto-commits with descriptive messages. Key characteristic: near-universal model support via LiteLLM, covering GPT-4o, Claude 3.5/4, Gemini, Ollama, and dozens of other providers. Aider popularized the "whole file" and "diff" editing formats that informed how later agents handle file modifications.

The SWE-Bench benchmark trajectory tells the story well: Aider held the top score on SWE-Bench Verified for several months in 2024-2025 before larger-context models and more capable agents surpassed it. That benchmark record established its reputation as a serious tool, not just a convenience wrapper.

#### Claude Code vs Aider

| Aspect | Claude Code | Aider |
|--------|-------------|-------|
| **Model support** | Claude only | GPT-4o, Claude, Gemini, Ollama, 50+ providers |
| **Git integration** | Native (reads .git, runs git) | Deep (auto-commits, commit messages, blame context) |
| **Architecture** | Anthropic proprietary | Open source, LiteLLM under the hood |
| **File editing** | Tool-based (Edit, Write) | Whole-file or diff format sent to model |
| **Web search** | Via MCP | Not native (requires plugin) |
| **Agentic loop** | Full (multi-turn, tool use) | Full (auto-accepts changes in architect mode) |
| **Release cadence** | Weekly | Monthly (last: v0.86.0, Aug 2025) |

The last release date (August 2025) is worth noting. Aider remains maintained and functional, but the release cadence has slowed relative to Claude Code and Hermes. This is not a warning sign by itself, but worth checking if you need cutting-edge features.

#### When to Choose Aider

Best case: you need multi-model support in a mature, battle-tested tool and do not want the operational overhead of Hermes. Aider is simpler to configure than Hermes, has a smaller footprint, and has years of community documentation.

Also a good fit for teams that have strong git discipline and want every AI change explicitly committed with a clear message. Aider's auto-commit behavior is more aggressive than Claude Code's (which asks before committing by default).

#### Quick Start

```bash
pip install aider-install && aider-install

# With Claude
export ANTHROPIC_API_KEY=sk-ant-...
aider --model claude-sonnet-4-6

# With GPT-4o
export OPENAI_API_KEY=sk-...
aider
```

See [aider.chat](https://aider.chat) for the full model list and configuration options.

---

### 1.4 Goose (AAIF/Block)

A general-purpose agent, not just a coding tool. Originally built by Block (formerly Square), transferred to the Linux Foundation's AAIF (Agentic AI Infrastructure Foundation) for long-term governance neutrality.

**Full coverage in [AI Ecosystem §11.1: Goose](./ai-ecosystem.md#111-goose-open-source-alternative-block).**

Quick stats: 45,900+ stars (May 2026), Rust (63%) + TypeScript (30%), Apache 2.0, daily active development, 368+ contributors. The headline difference from Claude Code: provider-agnostic (Claude, GPT, Gemini, Ollama, 15+ providers), with recipe-based reusable workflows and heterogeneous subagent teams where each subagent can run a different model.

---

## Section 2: Autonomous Coding Agents

These tools run without you watching. You give them a task description (a GitHub issue, a spec, a bug report), and they produce a pull request. The interaction model is fundamentally different from terminal agents: less iterative, more like assigning work to a colleague.

---

### 2.1 Devin (Cognition)

The first commercial fully autonomous software engineer. Closed-source, cloud-hosted, enterprise-priced.

| Attribute | Details |
|-----------|---------|
| **Website** | [devin.ai](https://devin.ai) |
| **Type** | Cloud SaaS, proprietary |
| **Pricing** | Core: $20/mo (pay-as-you-go ACUs), Team: $500/mo (250 ACUs), Enterprise: custom |
| **Launched** | 2024 |
| **Valuation** | $25B (April 2026 fundraise) |
| **Notable acquisition** | Windsurf AI-native IDE (July 2025) |
| **Enterprise customers** | Goldman Sachs, Microsoft, Palantir, Citi, Dell |

#### What Is Devin?

An autonomous software engineer that runs in a cloud-based Linux VM with its own shell, code editor, and browser. Devin plans its approach, writes code, runs tests, reads error messages, and iterates until the task is complete or it gets stuck. The primary interface is Slack: you send a message like "fix issue #342" and Devin opens a PR when done.

Billing is in ACUs (Agent Compute Units), where 1 ACU maps to roughly 15 minutes of agent work. A complex feature might consume 10-20 ACUs; a simple bug fix might use 1-3.

#### Claude Code vs Devin

| Aspect | Claude Code | Devin |
|--------|-------------|-------|
| **Execution environment** | Your local machine | Cloud Linux VM (sandboxed) |
| **Interaction model** | Interactive (you watch) | Async (assign and check back) |
| **State** | Session-scoped | Persistent across the task |
| **Pricing** | Subscription ($20-$200/mo) | Per-task ACU billing ($0.07-$0.15/ACU approx) |
| **Who drives** | You (pair programming) | Agent (autonomous, you review) |
| **Task specification** | Conversational, iterative | Upfront (better spec = better output) |
| **Browser access** | Via MCP (Playwright) | Built-in, native |
| **Code review integration** | You review in your IDE | Devin posts a PR, you review on GitHub |

#### When to Choose Devin

Devin works best when the task is well-specified, bounded, and does not require continuous judgment calls. Refactoring a specific module, implementing a documented API endpoint, fixing a regression with a known root cause: these are Devin tasks. Designing a new system architecture, debugging an obscure production issue, or writing code that depends on implicit context in your codebase: these require a more interactive loop.

The $500/month Team plan (250 ACUs) is substantial. At that price point, you are paying for the async value: developers not blocked waiting for agent output, agents running in parallel on multiple tasks, no context switching. If your bottleneck is developer attention rather than raw throughput, Devin is worth the calculation. If you want to stay in the loop and iterate interactively, Claude Code at $200/month delivers more value per dollar.

The Windsurf acquisition (July 2025) signals Cognition moving toward a full developer environment, not just a background agent. Watch for integrated workflows combining interactive coding (Windsurf IDE) and autonomous task execution (Devin) in the same product.

---

### 2.2 SWE-agent (Princeton)

An academic agent designed specifically for resolving GitHub issues from an issue description alone. NeurIPS 2024 paper, Princeton NLP Group and Stanford.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent) |
| **Stars** | 19,300+ (May 2026) |
| **Paper** | NeurIPS 2024 |
| **License** | MIT |
| **Language** | Python (95%) |
| **Version** | v1.1.0 (May 2025) |
| **Maintainers** | Princeton NLP Group + Stanford |

#### What Is SWE-agent?

An agent pipeline that takes a GitHub issue URL and a model, then attempts to reproduce the bug, write a fix, and produce a patch. Its architecture uses an Agent-Computer Interface (ACI) layer that abstracts terminal, file editing, and test running into a consistent set of commands regardless of the underlying environment. This ACI design is the main academic contribution: it shows that agent performance correlates strongly with how well the environment exposes information, not just with the model's raw capability.

SWE-agent + Claude 3.7 holds state-of-the-art on SWE-Bench Full (open-weights). The benchmark is the key context: SWE-Bench measures the percentage of real GitHub issues an agent can resolve end-to-end, and SWE-agent was designed with that benchmark as its optimization target.

#### When to Choose SWE-agent

Primarily academic and research use. If you want to run systematic evaluations of how different models perform on real GitHub issues, SWE-agent is the right tool because it has the reproducibility infrastructure (trajectory logging, evaluation harness, config YAML) that production tools skip.

For production batch issue resolution, Devin's cloud sandbox and better error recovery make it more practical. SWE-agent requires you to set up the environment and handle failures manually.

The research value is real: teams building agent systems can use SWE-agent's trajectory data (generated from issue resolution runs) to fine-tune models. Nous Research's SWE-agent-LM-32b (open weights, SoTA on SWE-Bench for open models) was trained on trajectories generated by SWE-agent.

```bash
pip install swe-agent

# Run on a GitHub issue
sweagent run \
  --agent.model.name=claude-sonnet-4-6 \
  --env.repo.github_url=https://github.com/org/repo \
  --problem_statement.github_url=https://github.com/org/repo/issues/123
```

---

### 2.3 Claude Code in Headless Mode

Claude Code's own autonomous mode: `claude -p "task"` runs a single instruction non-interactively and exits. Combined with CI/CD, it becomes an autonomous agent that triggers on GitHub events, runs on schedule via Routines, or processes tasks programmatically via the Agent SDK.

**This falls in the programmatic billing bucket since June 15, 2026.** See [Billing: Programmatic vs Interactive](../ultimate-guide.md#the-interactiveprogrammatic-billing-split-effective-june-15-2026) for the credit limits and overage rates.

Patterns:

```bash
# Single task, exits when done
claude -p "Write tests for src/auth.ts, aim for 80% coverage"

# GitHub Actions: triggered by issue label
# See workflows/event-driven-agents.md for the full pattern

# Agent SDK: programmatic with tools
# See ai-ecosystem.md §14 (Claude Managed Agents)
```

Cross-references:
- **Event-driven patterns**: [workflows/event-driven-agents.md](../workflows/event-driven-agents.md)
- **Agent teams**: [workflows/agent-teams.md](../workflows/agent-teams.md)
- **Managed Agents (cloud)**: [ai-ecosystem.md §14](./ai-ecosystem.md#14-claude-managed-agents)

---

## Section 3: Multi-Agent Frameworks

These are not coding tools. They are libraries for building custom multi-agent applications from scratch: marketing pipelines, research automation, document processing, customer support bots. You would use them if you are building a product that has AI agents inside it, not if you are a developer wanting an agent to write code for you.

The relationship to Claude Code: Claude (the model) can be one of the LLMs powering agents built with these frameworks. The frameworks themselves do not compete with Claude Code any more than Express.js competes with a browser.

---

### 3.1 CrewAI

Role-based multi-agent orchestration. The dominant choice for teams that want to define agents by job function (Researcher, Writer, Editor) and let them collaborate on structured tasks.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) |
| **Stars** | 52,300+ (May 2026) |
| **Language** | Python (99%) |
| **License** | MIT |
| **Version** | v1.14.5 (May 18, 2026) |
| **Executions** | 2B+ agent task executions reported |
| **Downloads** | 27M+ |
| **Enterprise customers** | 150+ |

#### What Is CrewAI?

You define agents with a role, a goal, and a backstory (the "crew"). You define tasks and assign them to agents. CrewAI handles routing: sequential (A finishes, then B starts), parallel (A and B run simultaneously), or hierarchical (a manager agent delegates to specialists). Each agent can use tools, including MCP servers and web search. Multiple LLM providers supported (Claude, GPT, Gemini, Ollama).

It stands apart from LangChain (the older framework it frequently gets compared to) because it does not depend on LangChain at all. Standalone Python library.

#### When to Use CrewAI

The right level of abstraction for teams that can describe their workflow in human roles. If you can say "I want a researcher who gathers information, a writer who drafts, and an editor who refines," CrewAI handles the orchestration and inter-agent communication. You write agent definitions, not orchestration code.

Avoid it when your workflow has complex conditional branching, requires durable execution across failures, or needs fine-grained control over how state passes between agents. LangGraph handles those cases better.

```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="Technical Researcher",
    goal="Find accurate technical information",
    backstory="Expert at synthesizing documentation and research papers",
    llm="claude-sonnet-4-6"
)

writer = Agent(
    role="Technical Writer",
    goal="Write clear, accurate documentation",
    backstory="Experienced at translating technical concepts",
    llm="claude-sonnet-4-6"
)

task = Task(
    description="Research and document the new auth API endpoints",
    expected_output="Markdown documentation with examples",
    agent=writer,
    context=[research_task]  # researcher's output feeds writer
)

crew = Crew(agents=[researcher, writer], tasks=[task], process=Process.sequential)
result = crew.kickoff()
```

---

### 3.2 LangGraph

Graph-based agent orchestration from LangChain. Lower-level than CrewAI, more flexible, better for complex stateful workflows.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) |
| **Stars** | 33,100+ (May 2026) |
| **Language** | Python (99%) + JS version available |
| **License** | MIT |
| **Version** | v1.2.2 (May 26, 2026) |
| **Production users** | Klarna, Replit, Elastic |

#### What Is LangGraph?

An agent construction framework that models workflows as directed graphs with nodes (agent steps) and edges (transitions). The key primitives are state (a typed dict that persists across all nodes), conditional edges (branching based on state), and persistence (checkpointing so an interrupted workflow resumes from the last checkpoint, not from scratch). Human-in-the-loop is a first-class pattern: you can pause execution at any node and wait for a human decision before continuing.

LangGraph does not bundle agents. You define the workflow logic and plug in whatever LLM you want. The framework ensures that state transitions are predictable, failures are recoverable, and the workflow can be debugged step by step.

#### When to Use LangGraph

The right tool when your agent needs to survive failures, branch on runtime conditions, or require human approval at specific decision points. Examples: a code review pipeline that escalates to a human when the agent detects a security-relevant change; a data processing workflow that checkpoints after each expensive step so restarts do not re-process completed stages; a multi-step research agent that pauses for human guidance when it hits ambiguous source material.

Steeper learning curve than CrewAI. Worth it when the workflow complexity justifies the investment.

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list
    task_complete: bool

graph = StateGraph(AgentState)
graph.add_node("agent", call_agent)
graph.add_node("tools", call_tools)
graph.add_conditional_edges("agent", should_continue, {"continue": "tools", "end": END})
graph.add_edge("tools", "agent")
graph.set_entry_point("agent")

app = graph.compile(checkpointer=MemorySaver())  # Durable execution
```

LangSmith (LangChain's observability product) integrates natively for debugging and tracing agent runs.

---

### 3.3 AutoGen / Microsoft Agent Framework

Microsoft's multi-agent framework, mid-transition from the original AutoGen library (maintenance mode since September 2025) to the Microsoft Agent Framework (MAF), which merges AutoGen and Semantic Kernel into one SDK.

| Attribute | Details (MAF) |
|-----------|---------|
| **GitHub** | [microsoft/agent-framework](https://github.com/microsoft/agent-framework) |
| **Stars** | 10,800+ (MAF, active) |
| **Legacy GitHub** | [microsoft/autogen](https://github.com/microsoft/autogen) (58,400 stars, maintenance mode since Sep 2025) |
| **Language** | Python + C# + TypeScript |
| **License** | MIT |
| **Version** | python-1.6.0 (May 22, 2026) |
| **Production release** | v1.0 (April 2026) |

#### What Is Microsoft Agent Framework?

MAF is the merge of AutoGen (Python, conversational multi-agent) and Semantic Kernel (C# + Python, function-calling abstractions). The result is a cross-runtime framework: Python agents can coordinate with .NET agents, all backed by the same messaging layer. It implements the A2A (Agent-to-Agent) protocol, Microsoft's contribution to agent interoperability, and supports MCP.

The AutoGen star count (58,400) reflects its historical reputation. AutoGen pioneered the "conversable agent" pattern where agents talk to each other in a structured conversation loop. That pattern is still the dominant mental model in the framework even as the implementation evolved.

#### When to Use MAF

Strong fit for Microsoft ecosystem teams: .NET + Python shops, Azure deployments, enterprise environments where Semantic Kernel is already established. The cross-runtime story is real: a Python agent can call tools implemented as .NET Semantic Kernel functions.

Less compelling for teams without existing .NET investment. If you are Python-only, CrewAI or LangGraph have larger communities and more tutorials.

---

### 3.4 Anthropic Agent SDK

Anthropic's own framework for building multi-agent systems programmatically, distinct from Claude Code. Covered in detail in [AI Ecosystem §14: Claude Managed Agents](./ai-ecosystem.md#14-claude-managed-agents).

The operative distinction: Claude Code is a finished product you use as a developer; the Agent SDK is a library you use to build products that have Claude inside them. The Agent SDK handles tool use, context management, and multi-agent coordination via the Messages API. It is also in the programmatic billing bucket (see billing cross-reference above).

---

## Section 4: Agent Orchestration Tools

Tools that sit above agent frameworks and manage how agents are deployed, routed, and operated at scale. Not to be confused with multi-Claude orchestration tools (Gas Town, multiclaude) which are covered in [Third-Party Tools](./third-party-tools.md#multi-agent-orchestration).

---

### 4.1 Conductor (Gemini CLI methodology)

A development methodology, not a product. "Conductor" started as an extension for Gemini CLI that enforces a Context, Spec, Plan, Implement workflow: before writing any code, the agent creates and commits a spec document, then a plan document, then implements against both.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [gemini-cli-extensions/conductor](https://github.com/gemini-cli-extensions/conductor) |
| **Stars** | 3,600+ (May 2026) |
| **License** | Apache 2.0 |

The methodology has been ported to Claude Code via community repos: [lackeyjb/claude-conductor](https://github.com/lackeyjb/claude-conductor), [ryanmac/code-conductor](https://github.com/ryanmac/code-conductor), and the wshobson/agents plugin marketplace. None of these have significant traction on their own, but the pattern itself (spec-before-code, committed documentation) maps directly to Claude Code's [Spec-First Development workflow](../workflows/spec-first.md).

---

### 4.2 Conductor (Microsoft CLI)

An entirely separate project from the Gemini one. A YAML-first CLI for deterministic multi-agent workflows where the routing logic is static configuration, not LLM decisions.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [microsoft/conductor](https://github.com/microsoft/conductor) |
| **Stars** | 158 (May 2026, brand new) |
| **License** | MIT |
| **Launched** | May 14, 2026 (Microsoft Open Source Blog) |

Core idea: define your agent workflow in YAML (which agents run in sequence, which in parallel, which model each uses, what gets passed between stages) and execute it deterministically. No LLM in the orchestration loop, only in the agent steps. Supports both GitHub Copilot SDK and Anthropic Agent SDK as providers. Very early stage (158 stars, days old at time of writing), but backed by Microsoft's open-source team.

---

### 4.3 Hermes Control Room

A community template by Shann (@shannhk, Lisbon) for managing a fleet of Hermes agents on a VPS. Not a Nous Research project.

| Attribute | Details |
|-----------|---------|
| **GitHub** | [shannhk/hermes-agent-control-room](https://github.com/shannhk/hermes-agent-control-room) |
| **Stars** | 474 (May 2026) |
| **Age** | 12 days (as of May 27, 2026) |
| **Type** | Template/documentation, not executable software |

The concept: a folder structure with governance docs, a registry of deployed agents, runbooks for common operations, and 8 bundled Hermes skills for VPS provisioning, task routing, backup, security auditing, and cron planning. Agents share a filesystem-based task bus (inbox/working/outbox/archive per specialty). The orchestrator reads the control room docs to know agent capabilities, routes tasks via the bus, and synthesizes results.

The pattern is sound for anyone running 3+ Hermes agents. The specific repo is too new (7 commits) to recommend as a production dependency. Watch for a v1.0 with more operational hardening.

---

## Section 5: Decision Framework

### Full Comparison Matrix

| Tool | Open Source | Stars | Model Support | Mode | Language | Cost |
|------|------------|-------|---------------|------|----------|------|
| **Claude Code** | Yes (TS) | 112K | Claude only | Interactive + headless | TypeScript | $20-$200/mo |
| **Codex CLI** | Yes | 86K | GPT-4o, o3, o4-mini | Interactive + headless | Rust | Included in ChatGPT Pro/Team |
| **Hermes Agent** | Yes (MIT) | 170K | 200+ providers | Interactive + cron + messaging | Python | Pay-per-LLM-call |
| **Aider** | Yes | 45K | 50+ providers | Interactive | Python | Pay-per-LLM-call |
| **Goose** | Yes | 46K | 15+ providers | Interactive + subagents | Rust | Pay-per-LLM-call |
| **Devin** | No | N/A | Proprietary | Fully autonomous | Proprietary | $20-$500/mo |
| **SWE-agent** | Yes (MIT) | 19K | Any (Claude, GPT...) | Autonomous (issue → PR) | Python | Pay-per-LLM-call |
| **CrewAI** | Yes (MIT) | 52K | 50+ providers | Framework (build your own) | Python | Framework is free |
| **LangGraph** | Yes (MIT) | 33K | Any | Framework | Python/JS | Framework is free |
| **AutoGen/MAF** | Yes (MIT) | 58K/11K | Any | Framework | Python/C#/TS | Framework is free |

### Situation to Tool Guide

| Situation | Recommended |
|-----------|-------------|
| Daily coding, already on Claude Max | Claude Code |
| Daily coding, already on ChatGPT Pro | Codex CLI |
| Daily coding, want any model | Hermes Agent or Aider |
| Daily coding, general-purpose agent | Goose |
| Assign a task, come back to a PR | Devin ($500/mo) or `claude -p` in CI |
| Fix GitHub issues autonomously, research/benchmark | SWE-agent |
| Orchestrate multiple Claude Code instances | Gas Town, multiclaude, Ruflo (see [Third-Party Tools](./third-party-tools.md#multi-agent-orchestration)) |
| Build a multi-agent product with roles | CrewAI |
| Build a stateful, recoverable workflow | LangGraph |
| Build in .NET + Python with Microsoft stack | AutoGen/MAF |
| Anthropic ecosystem, cloud-hosted agents | Anthropic Agent SDK (see [ai-ecosystem.md §14](./ai-ecosystem.md#14-claude-managed-agents)) |
| Manage a fleet of Hermes agents on VPS | Hermes Control Room pattern |

### The Model Lock-In Question

The single most clarifying question for choosing between Claude Code, Codex CLI, Hermes, Aider, and Goose: does the tool need to work with exactly one model provider, or multiple?

If you are committed to Claude and the Anthropic ecosystem (subscription, Routines, Agent SDK, CLAUDE.md tooling), Claude Code is unambiguously the right choice. The integration is native and the feature velocity from Anthropic is high.

If you need model flexibility (local models for sensitive code, cheaper models for routine tasks, specific models for benchmarking), Hermes Agent handles the broadest range with the most automation. Aider and Goose are simpler alternatives with smaller footprints.

If your team is OpenAI-first and already paying for ChatGPT Pro, Codex CLI costs nothing incremental.

### The Autonomy vs Control Trade-off

Higher autonomy means the agent can complete more work without you watching, but also means more ways to go off track on ambiguous tasks. The right autonomy level depends on how well-specified your tasks are, not on which tool is "more powerful."

Claude Code headless (`claude -p`) and SWE-agent give you controlled autonomy: you set the task, the agent runs, you review the output. Devin gives you maximal autonomy with a cloud sandbox: the agent has a full Linux environment and can take actions you did not anticipate. More power, more review required before merging.

Interactive agents (Claude Code terminal, Hermes, Aider, Goose) give you real-time control. You watch the agent think, redirect it when it goes wrong, and approve destructive actions. For exploratory work where requirements shift mid-session, interactive is faster than autonomous despite appearing more manual.

---

## Cross-References

- **Multi-Claude orchestration** (Gas Town, multiclaude, Ruflo, Conductor desktop): [Third-Party Tools: Multi-Agent Orchestration](./third-party-tools.md#multi-agent-orchestration)
- **Goose deep dive**: [AI Ecosystem §11.1](./ai-ecosystem.md#111-goose-open-source-alternative-block)
- **Building custom agents with Anthropic SDK**: [AI Ecosystem §14](./ai-ecosystem.md#14-claude-managed-agents)
- **Claude Code's own agent team patterns**: [workflows/agent-teams.md](../workflows/agent-teams.md)
- **Event-driven autonomous patterns**: [workflows/event-driven-agents.md](../workflows/event-driven-agents.md)
- **Programmatic billing (Hermes, Codex CLI, third-party harnesses)**: [Ultimate Guide: Billing Split](../ultimate-guide.md#the-interactiveprogrammatic-billing-split-effective-june-15-2026)
- **Agent harness engineering (theoretical framework)**: [core/agent-harness.md](../core/agent-harness.md)
- **Coding agents comparison matrix** (23 tools, 11 criteria): [coding-agents-matrix.dev](https://coding-agents-matrix.dev)
