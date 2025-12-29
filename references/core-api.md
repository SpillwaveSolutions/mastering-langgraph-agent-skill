# LangGraph Core API

Foundational concepts for building LangGraph applications.

## Contents

- [State](#state)
- [Nodes](#nodes)
- [Edges](#edges)
- [Graph Construction](#graph-construction)
- [Compile and Invoke](#compile-and-invoke)
- [Graph API vs Functional API](#graph-api-vs-functional-api)

---

## State

State is a shared mutable dictionary that carries data through the workflow. All nodes read from and write to this state.

### Defining State

Use `TypedDict` to define your state schema:

```python
from typing_extensions import TypedDict, Annotated
from langchain_core.messages import AnyMessage
import operator

class MyState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
    query: str
    results: list[str]
    step_count: int
```

### Aggregation Modes

| Pattern | Behavior | Use When |
|---------|----------|----------|
| `field: str` | Replace value | Single values, overwrite OK |
| `field: Annotated[list, operator.add]` | Append to list | Messages, accumulating results |

```python
# Replace mode (default)
class State(TypedDict):
    current_step: str  # Each update overwrites

# Append mode  
class State(TypedDict):
    messages: Annotated[list, operator.add]  # Each update appends
```

### State Design Principles

**Keep state raw:** Store data, not formatted prompts.

```python
# ✓ Good
class State(TypedDict):
    user_question: str
    retrieved_docs: list[str]
    intent: str

# ✗ Bad  
class State(TypedDict):
    full_prompt: str  # Mixes data with formatting
```

---

## Nodes

Nodes are Python functions that perform one step of the workflow.

### Node Signature

```python
def my_node(state: MyState) -> dict:
    # Read from state
    query = state["query"]
    
    # Do work
    result = process(query)
    
    # Return updates (only changed fields)
    return {"results": [result]}
```

### Return Values

**1. Dictionary (most common):** Updates merged into state
```python
def node(state) -> dict:
    return {"field": "new_value"}
```

**2. Command:** Update state AND control routing
```python
from langgraph.types import Command

def node(state) -> Command:
    return Command(
        update={"field": "value"},
        goto="next_node"  # Override normal edge
    )
```

### Node Best Practices

1. **Single purpose:** One node = one responsibility
2. **Descriptive names:** `classify_intent` not `step1`
3. **No side effects:** Don't modify global state
4. **Handle errors:** Return error info in state or raise for debugging

---

## Edges

Edges define transitions between nodes.

### Special Markers

```python
from langgraph.graph import START, END

# START: Entry point (where graph begins)
# END: Exit point (where graph terminates)
```

### Unconditional Edges

```python
graph.add_edge("node_a", "node_b")
graph.add_edge(START, "first_node")
graph.add_edge("last_node", END)
```

### Conditional Edges

```python
def route_by_type(state: MyState) -> str:
    if state["type"] == "urgent":
        return "urgent_handler"
    elif state["type"] == "normal":
        return "normal_handler"
    return END

graph.add_conditional_edges(
    "classifier",
    route_by_type,
    ["urgent_handler", "normal_handler", END]
)
```

### Conditional Edge Patterns

**Binary branch:**
```python
def should_continue(state) -> str:
    return END if state["done"] else "process_more"

graph.add_conditional_edges("check", should_continue, ["process_more", END])
```

**Loop back:**
```python
def check_complete(state) -> str:
    return END if state["iterations"] >= 3 else "iterate_again"

graph.add_conditional_edges("process", check_complete, ["iterate_again", END])
graph.add_edge("iterate_again", "process")  # Creates cycle
```

---

## Graph Construction

```python
from langgraph.graph import StateGraph, START, END

# 1. Initialize with state type
graph = StateGraph(MyState)

# 2. Add nodes
graph.add_node("node_a", node_a_function)
graph.add_node("node_b", node_b_function)

# 3. Add edges
graph.add_edge(START, "node_a")
graph.add_edge("node_a", "node_b")
graph.add_conditional_edges("node_b", router_fn, ["node_c", END])
graph.add_edge("node_c", END)

# 4. Compile
chain = graph.compile()
```

### Complete Example

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

class CounterState(TypedDict):
    count: int
    message: str

def increment(state: CounterState) -> dict:
    return {"count": state["count"] + 1}

def check_limit(state: CounterState) -> str:
    return END if state["count"] >= 5 else "increment"

graph = StateGraph(CounterState)
graph.add_node("increment", increment)
graph.add_edge(START, "increment")
graph.add_conditional_edges("increment", check_limit, ["increment", END])

chain = graph.compile()
result = chain.invoke({"count": 0, "message": ""})
# result["count"] == 5
```

---

## Compile and Invoke

### Basic Compilation

```python
chain = graph.compile()
```

### With Persistence

```python
from langgraph.checkpoint.memory import InMemorySaver

chain = graph.compile(checkpointer=InMemorySaver())
```

### Invoking

```python
# Without persistence
result = chain.invoke({"query": "hello", "messages": []})

# With persistence
result = chain.invoke(
    {"query": "hello", "messages": []},
    config={"configurable": {"thread_id": "user-123"}}
)
```

### Testing Specific Nodes

```python
node = chain.nodes["node_name"]
output = node.invoke({"query": "test"})
```

### Visualization

```python
print(chain.get_graph().draw_mermaid())
chain.get_graph().draw_mermaid_png(output_file_path="graph.png")
```

---

## Graph API vs Functional API

### Graph API (Declarative)

```python
graph = StateGraph(State)
graph.add_node("step1", step1_fn)
graph.add_node("step2", step2_fn)
graph.add_edge(START, "step1")
graph.add_edge("step1", "step2")
graph.add_edge("step2", END)
chain = graph.compile()
```

### Functional API (Imperative)

```python
from langgraph.func import entrypoint, task

@task
def step1(inputs):
    return {"result": process1(inputs)}

@task
def step2(inputs):
    return {"result": process2(inputs)}

@entrypoint()
def workflow(inputs):
    r1 = step1(inputs)
    r2 = step2(r1)
    return r2
```

| Aspect | Graph API | Functional API |
|--------|-----------|----------------|
| Style | Declarative | Imperative |
| Visualization | Natural fit | Requires compilation |
| Complex loops | More verbose | More natural |
| Runtime | Same | Same |

Both compile to the same execution engine.
