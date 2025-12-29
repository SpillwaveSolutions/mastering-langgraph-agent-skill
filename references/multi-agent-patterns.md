# Multi-Agent Patterns

Build multi-agent systems where specialized agents collaborate through supervisors or swarm-based handoffs.

## Contents

- [Architecture Overview](#architecture-overview)
- [Supervisor Pattern](#supervisor-pattern)
- [Swarm Pattern](#swarm-pattern)
- [Nested Hierarchies](#nested-hierarchies)
- [Custom Handoff Tools](#custom-handoff-tools)
- [Choosing a Pattern](#choosing-a-pattern)

---

## Architecture Overview

Multi-agent systems use a "divide-and-conquer" approach:
- **Specialized agents** handle specific domains with curated tools
- **Coordinator** routes tasks to the appropriate expert
- **Shared state** carries context between agents

| Pattern | Control Flow | Best For |
|---------|--------------|----------|
| Supervisor | Central orchestrator decides routing | Hierarchical workflows, clear task delegation |
| Swarm | Agents hand off to each other directly | Peer-to-peer collaboration, dynamic routing |

---

## Supervisor Pattern

A central supervisor routes tasks to specialized agents. Uses `langgraph-supervisor` package.

### Installation

```bash
pip install langgraph-supervisor
```

### Basic Supervisor

```python
from langchain_openai import ChatOpenAI
from langgraph_supervisor import create_supervisor
from langgraph.prebuilt import create_react_agent

model = ChatOpenAI(model="gpt-4o")

# Define specialized tools
def add(a: float, b: float) -> float:
    """Add two numbers."""
    return a + b

def multiply(a: float, b: float) -> float:
    """Multiply two numbers."""
    return a * b

def web_search(query: str) -> str:
    """Search the web for information."""
    return f"Results for: {query}"

# Create specialized agents
math_agent = create_react_agent(
    model=model,
    tools=[add, multiply],
    name="math_expert",
    prompt="You are a math expert. Always use one tool at a time."
)

research_agent = create_react_agent(
    model=model,
    tools=[web_search],
    name="research_expert",
    prompt="You are a researcher with web search access. Do not do math."
)

# Create supervisor workflow
workflow = create_supervisor(
    agents=[research_agent, math_agent],
    model=model,
    prompt=(
        "You are a team supervisor managing research and math experts. "
        "For current events, use research_expert. For math, use math_expert."
    )
)

# Compile and run
app = workflow.compile()
result = app.invoke({
    "messages": [{"role": "user", "content": "What's 25 * 4?"}]
})

for message in result["messages"]:
    print(f"{message.type}: {message.content}")
```

### Output Modes

Control how much agent history to include:

```python
# Include full message history from workers
workflow = create_supervisor(
    agents=[agent1, agent2],
    output_mode="full_history"
)

# Include only final response from workers
workflow = create_supervisor(
    agents=[agent1, agent2],
    output_mode="last_message"
)
```

### Custom Handoff Tool Names

```python
from langgraph_supervisor import create_handoff_tool

workflow = create_supervisor(
    [research_agent, math_agent],
    tools=[
        create_handoff_tool(
            agent_name="math_expert",
            name="assign_to_math",
            description="Assign math problems to the math expert"
        ),
        create_handoff_tool(
            agent_name="research_expert",
            name="assign_to_research",
            description="Assign research tasks to the research expert"
        )
    ],
    model=model,
)
```

---

## Swarm Pattern

Agents hand off control directly to each other. Uses `langgraph-swarm` package.

### Installation

```bash
pip install langgraph-swarm
```

### Basic Swarm

```python
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.prebuilt import create_react_agent
from langgraph_swarm import create_handoff_tool, create_swarm

model = ChatOpenAI(model="gpt-4o")

def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

# Create agents with handoff tools
alice = create_react_agent(
    model,
    [add, create_handoff_tool(agent_name="Bob")],
    prompt="You are Alice, an addition expert.",
    name="Alice",
)

bob = create_react_agent(
    model,
    [create_handoff_tool(
        agent_name="Alice",
        description="Transfer to Alice, she can help with math"
    )],
    prompt="You are Bob, you speak like a pirate.",
    name="Bob",
)

# Create swarm with persistence
checkpointer = InMemorySaver()
workflow = create_swarm(
    [alice, bob],
    default_active_agent="Alice"
)
app = workflow.compile(checkpointer=checkpointer)

# Multi-turn conversation
config = {"configurable": {"thread_id": "1"}}

turn_1 = app.invoke(
    {"messages": [{"role": "user", "content": "I'd like to speak to Bob"}]},
    config,
)
# Bob is now active

turn_2 = app.invoke(
    {"messages": [{"role": "user", "content": "What's 5 + 7?"}]},
    config,
)
# Bob transfers to Alice, Alice calculates 12
```

### Customer Support Example

```python
from langgraph_swarm import create_handoff_tool, create_swarm
from langgraph.prebuilt import create_react_agent

# Handoff tools for specialized agents
transfer_to_hotel = create_handoff_tool(
    agent_name="hotel_assistant",
    description="Transfer to hotel-booking assistant for hotel searches and bookings.",
)

transfer_to_flight = create_handoff_tool(
    agent_name="flight_assistant",
    description="Transfer to flight-booking assistant for flight searches and bookings.",
)

transfer_to_triage = create_handoff_tool(
    agent_name="triage_assistant",
    description="Transfer back to triage for general questions.",
)

# Triage agent routes to specialists
triage_agent = create_react_agent(
    model,
    [transfer_to_hotel, transfer_to_flight],
    prompt="You are a travel assistant. Route users to the appropriate specialist.",
    name="triage_assistant",
)

hotel_agent = create_react_agent(
    model,
    [search_hotels, book_hotel, transfer_to_triage, transfer_to_flight],
    prompt="You are a hotel booking specialist.",
    name="hotel_assistant",
)

flight_agent = create_react_agent(
    model,
    [search_flights, book_flight, transfer_to_triage, transfer_to_hotel],
    prompt="You are a flight booking specialist.",
    name="flight_assistant",
)

# Build swarm
workflow = create_swarm(
    [triage_agent, hotel_agent, flight_agent],
    default_active_agent="triage_assistant"
)
app = workflow.compile(checkpointer=InMemorySaver())
```

---

## Nested Hierarchies

Build multi-level supervisors for complex organizations:

```python
from langgraph_supervisor import create_supervisor
from langgraph.prebuilt import create_react_agent

# Team-level supervisors
research_team = create_supervisor(
    [research_agent, data_agent],
    model=model,
    supervisor_name="research_supervisor"
).compile(name="research_team")

writing_team = create_supervisor(
    [writer_agent, editor_agent],
    model=model,
    supervisor_name="writing_supervisor"
).compile(name="writing_team")

# Top-level supervisor manages teams
top_level = create_supervisor(
    [research_team, writing_team],
    model=model,
    supervisor_name="top_level_supervisor",
    prompt="Route research tasks to research_team, content tasks to writing_team."
).compile(name="top_level_supervisor")
```

---

## Custom Handoff Tools

Create handoff tools with additional context:

```python
from typing import Annotated
from langchain_core.tools import tool, BaseTool, InjectedToolCallId
from langchain_core.messages import ToolMessage
from langgraph.types import Command
from langgraph.prebuilt import InjectedState

def create_custom_handoff(*, agent_name: str, name: str, description: str) -> BaseTool:

    @tool(name, description=description)
    def handoff_to_agent(
        task_description: Annotated[str, "Detailed description for the next agent"],
        state: Annotated[dict, InjectedState],
        tool_call_id: Annotated[str, InjectedToolCallId],
    ):
        tool_message = ToolMessage(
            content=f"Successfully transferred to {agent_name}",
            name=name,
            tool_call_id=tool_call_id,
        )
        return Command(
            goto=agent_name,
            graph=Command.PARENT,
            update={
                "messages": state["messages"] + [tool_message],
                "active_agent": agent_name,
                "task_description": task_description,  # Pass context
            },
        )

    return handoff_to_agent

# Usage
transfer_with_context = create_custom_handoff(
    agent_name="specialist",
    name="transfer_to_specialist",
    description="Transfer with detailed task description"
)
```

---

## Choosing a Pattern

| Scenario | Recommended Pattern |
|----------|---------------------|
| Clear hierarchy, one decision-maker | Supervisor |
| Peer-to-peer, agents know when to hand off | Swarm |
| Complex organization with teams | Nested Supervisors |
| Customer support with escalation | Swarm with triage agent |
| Research + execution pipeline | Supervisor or sequential swarm |

### Key Considerations

1. **Supervisor**: More predictable, central control, easier to debug
2. **Swarm**: More flexible, agents decide routing, better for dynamic workflows
3. **Hybrid**: Use swarm within teams, supervisor between teams

---

## Official Resources

- [langgraph-supervisor](https://github.com/langchain-ai/langgraph-supervisor-py) - Hierarchical multi-agent
- [langgraph-swarm](https://github.com/langchain-ai/langgraph-swarm-py) - Swarm-style handoffs
- [Multi-agent tutorial](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/multi-agent-collaboration/) - Official notebook
