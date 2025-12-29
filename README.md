# Mastering LangGraph Agent Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![LangGraph](https://img.shields.io/badge/LangGraph-latest-green.svg)](https://github.com/langchain-ai/langgraph)
[![Agent Skill Standard](https://img.shields.io/badge/Agent%20Skill-Standard-purple.svg)](https://agentskills.io/)
[![SkillzWave](https://img.shields.io/badge/SkillzWave-Marketplace-orange.svg)](https://skillzwave.ai/skill/SpillwaveSolutions__mastering-langgraph-agent-skill__mastering-langgraph__SKILL/)

Build stateful AI agents and agentic workflows with LangGraph in Python. This skill provides comprehensive guidance for tool-using agents, branching workflows, conversation memory, human-in-the-loop oversight, multi-agent systems, and production deployment.

## Table of Contents

- [Overview](#overview)
- [Key Concepts](#key-concepts)
- [Quick Start](#quick-start)
- [Reference Documentation](#reference-documentation)
- [Requirements](#requirements)
- [Installation](#installing-with-skilz-universal-installer)
- [License](#license)

## Overview

This skill covers essential LangGraph patterns for building production-ready AI agents:

| Topic | Description |
|-------|-------------|
| **Tool-Using Agents** | LLM-tool loops that continue until task completion |
| **Branching Workflows** | Multi-step pipelines with conditional routing |
| **Persistence & Memory** | Checkpointers for conversation context across sessions |
| **Human-in-the-Loop** | Pause workflows for human approval with `interrupt()` |
| **Multi-Agent Systems** | Supervisor and swarm patterns for agent collaboration |
| **Production Deployment** | LangGraph Platform, Docker, and self-hosted options |
| **Debugging** | Time-travel, LangSmith tracing, and testing strategies |

## Key Concepts

| Concept | Description |
|---------|-------------|
| `StateGraph` | Core graph construction API |
| Nodes & Edges | Define steps and transitions |
| Conditional Edges | Route based on state values |
| `MessagesState` | Built-in state for chat applications |
| Checkpointers | Enable memory and time-travel |
| `Command` Objects | Control flow from within nodes |
| `ToolMessage` | Handle tool call results |

## Quick Start

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AnyMessage
from typing_extensions import TypedDict, Annotated
import operator

# Define state with append-mode messages
class State(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]

# Create chat node
llm = ChatOpenAI(model="gpt-4")

def chat(state: State) -> dict:
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

# Build and compile graph
graph = StateGraph(State)
graph.add_node("chat", chat)
graph.add_edge(START, "chat")
graph.add_edge("chat", END)

chain = graph.compile(checkpointer=InMemorySaver())

# Invoke with thread_id for memory persistence
result = chain.invoke(
    {"messages": [HumanMessage(content="Hello!")]},
    config={"configurable": {"thread_id": "user-123"}}
)
```

## Reference Documentation

This skill includes detailed reference guides for each major topic:

| Reference | Topic |
|-----------|-------|
| [core-api.md](references/core-api.md) | StateGraph, nodes, edges, compilation |
| [tool-agent-pattern.md](references/tool-agent-pattern.md) | ReAct agents, tool integration |
| [workflow-patterns.md](references/workflow-patterns.md) | Branching, parallel execution, prompt chaining |
| [persistence-memory.md](references/persistence-memory.md) | Checkpointers, thread_id, time-travel |
| [hitl-patterns.md](references/hitl-patterns.md) | interrupt(), breakpoints, human approval |
| [multi-agent-patterns.md](references/multi-agent-patterns.md) | Supervisor, swarm, nested hierarchies |
| [production-deployment.md](references/production-deployment.md) | LangGraph Platform, Docker, RemoteGraph |
| [debugging-monitoring.md](references/debugging-monitoring.md) | Testing, LangSmith, visualization |
| [official-resources.md](references/official-resources.md) | 150+ official documentation links |

## Requirements

- **Python**: >= 3.9
- **LangGraph**: `pip install langgraph`
- **LLM Provider**: OpenAI, Anthropic, or other supported providers

## Installing with Skilz (Universal Installer)

The recommended way to install this skill across different AI coding agents is using the **skilz** universal installer. This skill supports the [Agent Skill Standard](https://agentskills.io/), which means it works with 14+ coding agents including Claude Code, OpenAI Codex, Cursor, and Gemini.

### Install Skilz

```bash
pip install skilz
```

### Install from Git

You can use either `-g` or `--git` with HTTPS or SSH URLs:

```bash
# HTTPS URL
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill

# SSH URL
skilz install --git git@github.com:SpillwaveSolutions/mastering-langgraph-agent-skill.git
```

### Install from SkillzWave Marketplace

```bash
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph
```

### Agent-Specific Installation

#### Claude Code

```bash
# Install to user home (available in all projects)
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill

# Install to current project only
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --project
```

#### OpenCode

```bash
# User-level install
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --agent opencode

# Project-level install
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --project --agent opencode
```

#### Gemini CLI

```bash
# Project-level install (Gemini only supports project level)
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --agent gemini
```

#### OpenAI Codex

```bash
# User-level install
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --agent codex

# Project-level install
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --project --agent codex
```

### Other Supported Agents

Skilz supports 14+ coding agents including Windsurf, Qwen Code, Aidr, and more. For the full list of supported platforms, visit:

- [SkillzWave Platforms](https://skillzwave.ai/platforms/)
- [skilz-cli GitHub Repository](https://github.com/SpillwaveSolutions/skilz-cli)
- [Skill Listing](https://skillzwave.ai/skill/SpillwaveSolutions__mastering-langgraph-agent-skill__mastering-langgraph__SKILL/)

## License

MIT

---

[SkillzWave - Largest Agentic Marketplace for AI Agent Skills](https://skillzwave.ai/) | [SpillWave - Leaders in AI Agent Development](https://spillwave.com/)
