# Mastering LangGraph Agent Skill

Build stateful AI agents and agentic workflows with LangGraph in Python. This skill provides comprehensive guidance for tool-using agents, branching workflows, conversation memory, human-in-the-loop oversight, multi-agent systems, and production deployment.

## Overview

This skill covers:
- **Tool-Using Agents** - LLM-tool loops that continue until task completion
- **Branching Workflows** - Multi-step pipelines with conditional routing
- **Persistence & Memory** - Checkpointers for conversation context across sessions
- **Human-in-the-Loop** - Pause workflows for human approval with `interrupt()`
- **Multi-Agent Systems** - Supervisor and swarm patterns for agent collaboration
- **Production Deployment** - LangGraph Platform, Docker, and self-hosted options
- **Debugging** - Time-travel, LangSmith tracing, and testing strategies

## Key Concepts Covered

| Concept | Description |
|---------|-------------|
| StateGraph | Core graph construction API |
| Nodes & Edges | Define steps and transitions |
| Conditional Edges | Route based on state values |
| MessagesState | Built-in state for chat applications |
| Checkpointers | Enable memory and time-travel |
| Command Objects | Control flow from within nodes |
| ToolMessage | Handle tool call results |

## Reference Files

This skill includes detailed reference documentation:

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

## Quick Example

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

## Requirements

- Python >= 3.9
- LangGraph (`pip install langgraph`)
- LLM provider (OpenAI, Anthropic, etc.)

## Installing with Skilz (Universal Installer)

The recommended way to install this skill across different AI coding agents is using the **skilz** universal installer.

### Install Skilz

```bash
pip install skilz
```

This skill supports [Agent Skill Standard](https://agentskills.io/) which means it supports 14 plus coding agents including Claude Code, OpenAI Codex, Cursor and Gemini.

### Git URL Options

You can use either `-g` or `--git` with HTTPS or SSH URLs:

```bash
# HTTPS URL
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill

# SSH URL
skilz install --git git@github.com:SpillwaveSolutions/mastering-langgraph-agent-skill.git
```

### Claude Code

Install to user home (available in all projects):
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill
```

Install to current project only:
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --project
```

### OpenCode

Install for [OpenCode](https://opencode.ai):
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --agent opencode
```

Project-level install:
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --project --agent opencode
```

### Gemini

Project-level install for Gemini:
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --agent gemini
```

### OpenAI Codex

Install for OpenAI Codex:
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --agent codex
```

Project-level install:
```bash
skilz install -g https://github.com/SpillwaveSolutions/mastering-langgraph-agent-skill --project --agent codex
```

### Install from Skillzwave Marketplace

```bash
# Claude to user home dir ~/.claude/skills
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph

# Claude skill in project folder ./claude/skills
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph --project

# OpenCode install to user home dir ~/.config/opencode/skills
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph --agent opencode

# OpenCode project level
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph --agent opencode --project

# OpenAI Codex install to user home dir ~/.codex/skills
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph

# OpenAI Codex project level ./.codex/skills
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph --agent opencode --project

# Gemini CLI (project level) -- only works with project level
skilz install SpillwaveSolutions_mastering-langgraph-agent-skill/mastering-langgraph --agent gemini
```

See [skill Listing](https://skillzwave.ai/skill/SpillwaveSolutions__mastering-langgraph-agent-skill__mastering-langgraph__SKILL/) to see how to install this exact skill to 14+ different coding agents.

### Other Supported Agents

Skilz supports 14+ coding agents including Windsurf, Qwen Code, Aidr, and more.

For the full list of supported platforms, visit [SkillzWave.ai/platforms](https://skillzwave.ai/platforms/) or see the [skilz-cli GitHub repository](https://github.com/SpillwaveSolutions/skilz-cli)

## License

MIT

---

<a href="https://skillzwave.ai/">Largest Agentic Marketplace for AI Agent Skills</a> | <a href="https://spillwave.com/">SpillWave: Leaders in AI Agent Development</a>
