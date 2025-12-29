# Debugging and Monitoring

Test nodes, visualize graphs, trace executions, and diagnose issues.

## Contents

- [Unit Testing Nodes](#unit-testing-nodes)
- [Graph Visualization](#graph-visualization)
- [LangSmith Tracing](#langsmith-tracing)
- [Step-by-Step Debugging](#step-by-step-debugging)
- [Common Issues and Fixes](#common-issues-and-fixes)

---

## Unit Testing Nodes

### Direct Testing

```python
def test_classify_node():
    state = {"query": "billing help", "messages": []}
    result = classify_node(state)
    
    assert "classification" in result
    assert result["classification"] in ["billing", "technical", "general"]
```

### Via Compiled Graph

```python
def test_node_via_graph():
    chain = graph.compile()
    node = chain.nodes["classify"]
    result = node.invoke({"query": "billing", "messages": []})
    assert result["classification"] == "billing"
```

### Mocking LLM

```python
from unittest.mock import Mock, patch

def test_with_mock_llm():
    mock_llm = Mock()
    mock_llm.invoke.return_value.content = "Mocked response"
    
    with patch("my_module.llm", mock_llm):
        result = generate_response({"query": "test"})
    
    assert result["response"] == "Mocked response"
```

---

## Graph Visualization

### Mermaid (Text)

```python
chain = graph.compile()
print(chain.get_graph().draw_mermaid())
```

### PNG Image

```python
chain.get_graph().draw_mermaid_png(output_file_path="graph.png")
```

### Jupyter

```python
from IPython.display import Image, display
display(Image(chain.get_graph().draw_mermaid_png()))
```

---

## LangSmith Tracing

### Setup

```bash
export LANGSMITH_API_KEY="ls-..."
export LANGSMITH_TRACING=true
export LANGSMITH_PROJECT="my-agent"
```

### Selective Tracing

```python
import langsmith as ls

with ls.tracing_context(project_name="experiment-1", enabled=True):
    result = chain.invoke(input_state, config=config)
```

### Adding Metadata

```python
config = {
    "configurable": {"thread_id": "user-123"},
    "metadata": {"user_id": "123", "version": "1.0.0"}
}
result = chain.invoke(input_state, config=config)
```

---

## Step-by-Step Debugging

### Stream Each Step

```python
for event in chain.stream(input_state, config=config):
    print(f"Node: {list(event.keys())}")
    print(f"Output: {event}")
```

### Interrupt After Node

```python
chain = graph.compile(
    checkpointer=InMemorySaver(),
    interrupt_after=["classify"]
)

partial = chain.invoke(input_state, config=config)
state = chain.get_state(config)
print(f"Classification: {state.values.get('classification')}")

# Continue
final = chain.invoke(None, config=config)
```

---

## Common Issues and Fixes

### 1. Messages Disappearing

**Symptom:** Only last message retained.

```python
# ✗ Wrong
messages: list[AnyMessage]

# ✓ Fix
messages: Annotated[list[AnyMessage], operator.add]
```

### 2. No Memory Between Turns

**Symptom:** Agent forgets previous turns.

```python
# ✓ Fix: Add checkpointer + thread_id
chain = graph.compile(checkpointer=InMemorySaver())
chain.invoke(input, config={"configurable": {"thread_id": "id"}})
```

### 3. Wrong Node Executed

**Debug:** Log router decision.

```python
def route_debug(state) -> str:
    result = "node_a" if state["condition"] else "node_b"
    print(f"Routing: {state['condition']} → {result}")
    return result
```

### 4. Tool Not Called

**Causes:** Tool not bound, poor docstring.

```python
# Check binding
print(model_with_tools.kwargs.get("tools"))

# Improve docstring
@tool
def search(query: str) -> str:
    """Search knowledge base. Use for ANY factual question."""
    return results
```

### 5. Infinite Loop

**Fix:** Add iteration limit.

```python
def should_continue(state) -> str:
    if state.get("iterations", 0) >= 10:
        return END
    # ... normal logic
```

### 6. State Update Not Applied

**Cause:** Key mismatch.

```python
class State(TypedDict):
    result: str

def node(state) -> dict:
    return {"results": "value"}  # ✗ Typo: "results" vs "result"
```

### Debug Checklist

- [ ] Visualize: `chain.get_graph().draw_mermaid()`
- [ ] Check state keys match node returns
- [ ] Verify `operator.add` on list fields
- [ ] Confirm checkpointer + thread_id for memory
- [ ] Enable LangSmith tracing
- [ ] Use `stream()` for step-by-step view
