# 🤖 Claude Agentic AI System

A production-grade, educational implementation of an **Agentic AI system** built on top of Anthropic's Claude LLM. This project demonstrates core concepts in autonomous AI agent design: multi-step planning, tool-based execution, and iterative reasoning.

> Built as part of a final-year engineering capstone project to showcase understanding of modern Agentic AI architecture.

---

## 📌 What Is an Agentic AI?

An **Agentic AI** is an autonomous system where a language model acts as a *reasoning engine* — it receives a high-level goal, breaks it into steps, decides which tools to use, acts on those decisions, observes the results, and iterates until the goal is accomplished.

This project implements the **ReAct pattern** (Reason + Act), a widely-used framework for building reliable AI agents.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER GOAL (input)                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        AGENT LOOP                               │
│                                                                 │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────────┐  │
│   │          │    │          │    │          │    │         │  │
│   │  PLAN    │───▶│   ACT    │───▶│  OBSERVE │───▶│ ITERATE │  │
│   │ (Claude) │    │  (Tool)  │    │ (Result) │    │(Context)│  │
│   │          │◀───│          │◀───│          │◀───│         │  │
│   └──────────┘    └──────────┘    └──────────┘    └─────────┘  │
│                                                                 │
│                    Repeats up to max_iterations                 │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FINAL ANSWER (output)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

| Component | File | Responsibility |
|-----------|------|----------------|
| **Agent Loop** | `agent/agent_loop.py` | Orchestrates the Plan→Act→Observe→Iterate cycle |
| **Planner** | `agent/planner.py` | Sends context to Claude; receives reasoning + tool calls |
| **Observer** | `agent/observer.py` | Executes tools; captures and formats results |
| **Context Manager** | `memory/context_manager.py` | Maintains full conversation history (short-term memory) |
| **Tool Registry** | `tools/tool_registry.py` | Central catalogue; routes tool calls to implementations |
| **Prompt Templates** | `prompts/templates.py` | All prompts in one place for easy iteration |

---

## 📁 Project Structure

```
claude-agent/
├── main.py                        # ← Entry point
├── requirements.txt
├── .env.example
├── .gitignore
│
├── agent/
│   ├── agent_loop.py              # Core Plan→Act→Observe loop
│   ├── planner.py                 # Claude API interface (reasoning)
│   └── observer.py                # Tool execution + result capture
│
├── memory/
│   └── context_manager.py         # Conversation history / short-term memory
│
├── tools/
│   ├── base_tool.py               # Abstract base class for all tools
│   ├── tool_registry.py           # Tool registration + dispatch
│   ├── web_search_tool.py         # Web search (mock → swap for Tavily/SerpAPI)
│   ├── summarizer_tool.py         # Text summarisation via Claude Haiku
│   ├── file_writer_tool.py        # Write results to disk
│   └── calculator_tool.py         # Safe arithmetic evaluator
│
├── prompts/
│   └── templates.py               # System prompt + all prompt templates
│
├── examples/
│   └── research_task.py           # Full example: research + report writing
│
├── tests/
│   ├── test_tools.py
│   └── test_context_manager.py
│
├── docs/
│   ├── ARCHITECTURE.md            # Deep-dive architecture explanation
│   └── ADDING_TOOLS.md            # Guide to adding custom tools
│
├── outputs/                       # Agent-generated files land here
└── logs/                          # agent.log written here
```

---

## 🚀 Quick Start

### 1. Clone and install

```bash
git clone https://github.com/YOUR_USERNAME/claude-agent.git
cd claude-agent
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Set your API key

```bash
cp .env.example .env
# Edit .env and add your Anthropic API key
export ANTHROPIC_API_KEY=sk-ant-...
```

### 3. Run the agent

```bash
# Default research task
python main.py

# Custom goal
python main.py --goal "Explain how Large Language Models work and save a report"

# Adjust iteration limit
python main.py --goal "Calculate compound interest for 1000 at 7% over 10 years" --max-iterations 3
```

### 4. Run tests

```bash
pytest tests/ -v
```

---

## 🔄 The Agent Loop — Step by Step

Here is exactly what happens when you run the agent:

```
User Goal: "Research Agentic AI and save a report"

━━━ ITERATION 1 ━━━
PLAN    Claude thinks: "I need to find what Agentic AI is. I'll search the web."
ACT     Calls: web_search(query="Agentic AI definition and components")
OBSERVE Result: 3 search results with titles, URLs, snippets

━━━ ITERATION 2 ━━━
PLAN    Claude thinks: "I have background info. Now I need applications."
ACT     Calls: web_search(query="Agentic AI real world applications 2024")
OBSERVE Result: More search results

━━━ ITERATION 3 ━━━
PLAN    Claude thinks: "I have enough raw data. Let me condense it."
ACT     Calls: summarize_text(text="[all gathered info]", focus="key points")
OBSERVE Result: Clean bullet-point summary

━━━ ITERATION 4 ━━━
PLAN    Claude thinks: "Now I'll write the report to disk."
ACT     Calls: write_file(filename="report.md", content="# Agentic AI Report...")
OBSERVE Result: "Written 1,240 characters to outputs/report.md"

━━━ ITERATION 5 ━━━
PLAN    Claude thinks: "I have everything. Time for the final answer."
       → stop_reason = "end_turn" → returns Final Answer
```

---

## 🔧 Available Tools

### `web_search`
Searches the web for information. In the current implementation uses a mock knowledge base. Swap `_mock_search()` in `web_search_tool.py` for a real API (Tavily, SerpAPI, Brave Search).

### `summarize_text`
Uses Claude Haiku (fast, cheap) as a nested sub-agent to condense raw text into bullet points. Demonstrates **tool chaining** and **model routing**.

### `write_file`
Safely writes content to `outputs/`. Path traversal is prevented — only filenames (no `../` escapes) are accepted.

### `calculator`
Evaluates arithmetic expressions using Python's `ast` module (no `eval()` — safe by design).

---

## ➕ Adding a New Tool

1. Create `tools/my_tool.py` subclassing `BaseTool`
2. Implement `name`, `description`, `parameters`, `execute()`
3. Register in `ToolRegistry._register_defaults()`

See `docs/ADDING_TOOLS.md` for a complete walkthrough.

---

## 🧠 Key Concepts Demonstrated

| Concept | Where |
|---------|-------|
| ReAct agent pattern | `agent/agent_loop.py` |
| Claude as reasoning engine | `agent/planner.py` |
| Structured tool use (JSON Schema) | `tools/base_tool.py` + `tool_registry.py` |
| Prompt engineering | `prompts/templates.py` |
| Short-term memory / context | `memory/context_manager.py` |
| Separation of concerns | Entire project structure |
| Safe tool execution | `agent/observer.py` (error handling) |
| Model routing | `summarizer_tool.py` (Haiku for sub-tasks) |

---

## 📚 References

- [Anthropic Tool Use Documentation](https://docs.anthropic.com/en/docs/tool-use)
- [ReAct: Synergising Reasoning and Acting (Yao et al., 2022)](https://arxiv.org/abs/2210.03629)
- [Anthropic Claude Models](https://docs.anthropic.com/en/docs/models-overview)

---

## 📄 License

MIT License — free to use, modify, and distribute with attribution.

## Author 

Jenson Antony
