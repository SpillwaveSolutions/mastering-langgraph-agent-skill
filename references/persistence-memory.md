# Persistence and Memory

Enable conversation memory, survive crashes, and debug with time-travel.

## Contents

- [Why Persistence?](#why-persistence)
- [Checkpointers](#checkpointers)
- [Thread IDs](#thread-ids)
- [Multi-Turn Conversations](#multi-turn-conversations)
- [Time Travel](#time-travel)
- [Durable Execution](#durable-execution)

---

## Why Persistence?

| Feature | Benefit |
|---------|---------|
| Conversation memory | Agent remembers previous turns |
| Crash recovery | Resume from last checkpoint |
| Time-travel | Debug by replaying from any point |
| Audit trail | Full history of state changes |

---

## Checkpointers

### InMemorySaver (Development)

```python
from langgraph.checkpoint.memory import InMemorySaver

chain = graph.compile(checkpointer=InMemorySaver())
```

### PostgresSaver (Production)

```python
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://user:pass@localhost:5432/langgraph"
checkpointer = PostgresSaver.from_conn_string(DB_URI)
chain = graph.compile(checkpointer=checkpointer)
```

### SQLiteSaver

```python
from langgraph.checkpoint.sqlite import SqliteSaver

checkpointer = SqliteSaver.from_conn_string("checkpoints.db")
chain = graph.compile(checkpointer=checkpointer)
```

---

## Thread IDs

Identify separate conversations:

```python
chain = graph.compile(checkpointer=InMemorySaver())

config = {"configurable": {"thread_id": "user-123"}}

# First message
chain.invoke({"messages": [HumanMessage(content="I'm Alice")]}, config=config)

# Second message - remembers Alice
chain.invoke({"messages": [HumanMessage(content="What's my name?")]}, config=config)

# Different thread - doesn't know Alice
other_config = {"configurable": {"thread_id": "user-456"}}
chain.invoke({"messages": [HumanMessage(content="What's my name?")]}, config=other_config)
```

### Thread ID Strategies

| Strategy | Example | Use Case |
|----------|---------|----------|
| User ID | `"user-123"` | One conversation per user |
| Session ID | `"user-123-session-5"` | Multiple per user |
| UUID | `str(uuid4())` | Unique per conversation |

---

## Multi-Turn Conversations

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AnyMessage
from typing_extensions import TypedDict, Annotated
import operator

class State(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]

llm = ChatOpenAI(model="gpt-4")

def chat(state: State) -> dict:
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

graph = StateGraph(State)
graph.add_node("chat", chat)
graph.add_edge(START, "chat")
graph.add_edge("chat", END)

chain = graph.compile(checkpointer=InMemorySaver())
config = {"configurable": {"thread_id": "demo"}}

# Turn 1
chain.invoke({"messages": [HumanMessage(content="My color is blue.")]}, config=config)

# Turn 2 - remembers
result = chain.invoke({"messages": [HumanMessage(content="What's my color?")]}, config=config)
# "Your color is blue."
```

---

## Time Travel

### Viewing History

```python
for state in chain.get_state_history(config):
    print(f"Checkpoint: {state.config['configurable']['checkpoint_id']}")
    print(f"Next: {state.next}")
```

### Resuming from Checkpoint

```python
checkpoint_id = "abc123..."

resume_config = {
    "configurable": {
        "thread_id": "my-thread",
        "checkpoint_id": checkpoint_id
    }
}

result = chain.invoke(None, config=resume_config)
```

### Modifying and Resuming

```python
state = chain.get_state(config)
modified = dict(state.values)
modified["messages"].append(HumanMessage(content="Try again."))

chain.update_state(config, modified)
result = chain.invoke(None, config=config)
```

### Forking

```python
fork_config = {
    "configurable": {
        "thread_id": "forked-thread",
        "checkpoint_id": checkpoint_id
    }
}
result = chain.invoke(new_input, config=fork_config)
```

---

## Durable Execution

Survive crashes and resume:

### Idempotency

```python
# ⚠️ Problem: re-sends on resume
def send_email(state):
    send(state["draft"])
    return {"sent": True}

# ✓ Fix: check first
def send_email(state):
    if not state.get("sent"):
        send(state["draft"])
    return {"sent": True}

# ✓ Or use @task
from langgraph.func import task

@task
def send_email(state):
    send(state["draft"])
    return {"sent": True}
```

### Long-Running Workflows

```python
chain = graph.compile(checkpointer=PostgresSaver.from_conn_string(DB_URI))

config = {"configurable": {"thread_id": "long-job-1"}}
chain.invoke(initial_state, config=config)

# If server restarts, invoke again - resumes from checkpoint
chain.invoke(None, config=config)
```

---

## Production Setup

```python
import os

ENV = os.getenv("ENVIRONMENT", "development")

if ENV == "production":
    from langgraph.checkpoint.postgres import PostgresSaver
    checkpointer = PostgresSaver.from_conn_string(os.getenv("DATABASE_URL"))
else:
    from langgraph.checkpoint.memory import InMemorySaver
    checkpointer = InMemorySaver()

chain = graph.compile(checkpointer=checkpointer)
```
