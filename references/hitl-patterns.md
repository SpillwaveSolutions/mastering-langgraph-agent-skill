# Human-in-the-Loop Patterns

Pause agents for human approval, correction, or additional input.

## Contents

- [Overview](#overview)
- [The interrupt() Function](#the-interrupt-function)
- [Approval Workflows](#approval-workflows)
- [Correction Workflows](#correction-workflows)
- [Requesting Additional Input](#requesting-additional-input)
- [Best Practices](#best-practices)

---

## Overview

| Pattern | Use Case |
|---------|----------|
| Approval | Review draft before sending |
| Correction | Fix agent mistakes mid-execution |
| Input | Request missing information |
| Escalation | Hand off complex cases |

**Requirement:** HITL requires a checkpointer for state persistence.

---

## The interrupt() Function

```python
from langgraph.types import interrupt

def review_node(state):
    draft = state["draft"]
    
    # Pause and ask human
    human_response = interrupt({
        "draft": draft,
        "question": "Approve or provide feedback."
    })
    
    # Resumes here after human responds
    if human_response.get("approved"):
        return {"status": "approved"}
    return {"feedback": human_response.get("feedback")}
```

### Resuming After Interrupt

```python
config = {"configurable": {"thread_id": "review-123"}}

# Initial invocation (pauses at interrupt)
result = chain.invoke({"draft": "Hello world..."}, config=config)
# Returns interrupt payload

# Human provides response
human_input = {"approved": True}
final = chain.invoke(human_input, config=config)
```

---

## Approval Workflows

### Draft → Review → Send

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.types import interrupt
from typing_extensions import TypedDict

class EmailState(TypedDict):
    request: str
    draft: str
    approved: bool
    sent: bool

def generate_draft(state: EmailState) -> dict:
    draft = llm.invoke(f"Write email for: {state['request']}")
    return {"draft": draft.content}

def human_review(state: EmailState) -> dict:
    response = interrupt({
        "draft": state["draft"],
        "action": "approve_or_edit"
    })
    
    if response.get("approved"):
        return {"approved": True}
    elif response.get("edited_draft"):
        return {"draft": response["edited_draft"], "approved": True}
    return {"approved": False}

def send_email(state: EmailState) -> dict:
    if state["approved"]:
        send(state["draft"])
        return {"sent": True}
    return {"sent": False}

def route_after_review(state: EmailState) -> str:
    return "send" if state["approved"] else "__end__"

graph = StateGraph(EmailState)
graph.add_node("generate", generate_draft)
graph.add_node("review", human_review)
graph.add_node("send", send_email)

graph.add_edge(START, "generate")
graph.add_edge("generate", "review")
graph.add_conditional_edges("review", route_after_review, ["send", "__end__"])
graph.add_edge("send", END)

chain = graph.compile(checkpointer=InMemorySaver())
```

---

## Correction Workflows

### Rollback and Retry

```python
# Find checkpoint before mistake
for state in chain.get_state_history(config):
    print(f"{state.config['configurable']['checkpoint_id']}: {state.next}")

# Resume from earlier checkpoint
corrected_config = {
    "configurable": {
        "thread_id": "task-123",
        "checkpoint_id": "checkpoint-before-mistake"
    }
}

chain.update_state(corrected_config, {"classification": "correct_value"})
result = chain.invoke(None, config=corrected_config)
```

### Mid-Execution Verification

```python
def classification_node(state) -> dict:
    classification = llm.invoke(f"Classify: {state['input']}").content
    
    if state.get("confidence", 1.0) < 0.8:
        human_check = interrupt({
            "proposed": classification,
            "question": "Is this correct?"
        })
        if not human_check.get("correct"):
            classification = human_check.get("corrected_value")
    
    return {"classification": classification}
```

---

## Requesting Additional Input

```python
def process_order(state) -> dict:
    if not state.get("shipping_address"):
        address = interrupt({
            "question": "Please provide shipping address",
            "required_fields": ["street", "city", "zip"]
        })
        return {"shipping_address": address}
    
    return {"order_status": "processing"}
```

---

## Best Practices

### 1. Always Use Checkpointer

```python
# ✓ Required for interrupt()
chain = graph.compile(checkpointer=InMemorySaver())
```

### 2. Clear Interrupt Payloads

```python
# ✓ Clear and actionable
interrupt({
    "draft": state["draft"],
    "action": "approve_or_reject",
    "instructions": "Review for tone and accuracy"
})
```

### 3. Handle All Response Cases

```python
def review_node(state):
    response = interrupt({"draft": state["draft"]})
    
    if response.get("approved"):
        return {"status": "approved"}
    elif response.get("rejected"):
        return {"status": "rejected", "reason": response.get("reason")}
    elif response.get("edited"):
        return {"draft": response["edited"], "status": "revised"}
    return {"status": "pending"}
```

### 4. Audit Trail

```python
def audited_review(state):
    response = interrupt({"draft": state["draft"]})
    
    return {
        "approved": response.get("approved"),
        "reviewer": response.get("reviewer_id"),
        "review_timestamp": response.get("timestamp")
    }
```
