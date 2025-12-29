# Tool-Using Agent Pattern

Build agents that call external tools in a loop until the task is complete.

## Contents

- [Overview](#overview)
- [Defining Tools](#defining-tools)
- [Binding Tools to Model](#binding-tools-to-model)
- [Agent State](#agent-state)
- [The Agent Loop](#the-agent-loop)
- [Complete Example](#complete-example)
- [Tool Error Handling](#tool-error-handling)
- [Advanced Patterns](#advanced-patterns)

---

## Overview

A tool-using agent follows this cycle:

```
START → LLM decides → Tool needed? 
                         ↓ Yes: Execute tool → Loop back to LLM
                         ↓ No: Return answer → END
```

---

## Defining Tools

```python
from langchain_core.tools import tool

@tool
def search(query: str) -> str:
    """Search the knowledge base for information.
    
    Args:
        query: The search query string
    """
    return f"Results for: {query}"

@tool
def calculate(expression: str) -> float:
    """Evaluate a mathematical expression.
    
    Args:
        expression: Math expression like '2 + 3 * 4'
    """
    return eval(expression)
```

**Requirements:** Clear docstring, type hints, descriptive name.

---

## Binding Tools to Model

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)
tools = [search, calculate]
model_with_tools = llm.bind_tools(tools)
tools_by_name = {t.name: t for t in tools}
```

---

## Agent State

```python
from typing_extensions import TypedDict, Annotated
from langchain_core.messages import AnyMessage
import operator

class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
```

---

## The Agent Loop

### LLM Node

```python
from langchain_core.messages import SystemMessage

def llm_node(state: AgentState) -> dict:
    system = SystemMessage(content="You are a helpful assistant with tools.")
    response = model_with_tools.invoke([system] + state["messages"])
    return {"messages": [response]}
```

### Tool Node

```python
from langchain_core.messages import ToolMessage

def tool_node(state: AgentState) -> dict:
    results = []
    for tool_call in state["messages"][-1].tool_calls:
        tool = tools_by_name[tool_call["name"]]
        output = tool.invoke(tool_call["args"])
        results.append(ToolMessage(
            content=str(output),
            tool_call_id=tool_call["id"]
        ))
    return {"messages": results}
```

### Router

```python
from typing import Literal

def should_continue(state: AgentState) -> Literal["tool_node", "__end__"]:
    last_message = state["messages"][-1]
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "tool_node"
    return "__end__"
```

### Assembly

```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(AgentState)
graph.add_node("llm", llm_node)
graph.add_node("tool_node", tool_node)
graph.add_edge(START, "llm")
graph.add_conditional_edges("llm", should_continue, ["tool_node", END])
graph.add_edge("tool_node", "llm")

agent = graph.compile()
```

---

## Complete Example

```python
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, SystemMessage, ToolMessage, AnyMessage
from typing_extensions import TypedDict, Annotated
from typing import Literal
import operator

@tool
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

tools = [add, multiply]
tools_by_name = {t.name: t for t in tools}

llm = ChatOpenAI(model="gpt-4", temperature=0)
model_with_tools = llm.bind_tools(tools)

class State(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]

def llm_node(state: State) -> dict:
    system = SystemMessage(content="You are a calculator assistant.")
    response = model_with_tools.invoke([system] + state["messages"])
    return {"messages": [response]}

def tool_node(state: State) -> dict:
    results = []
    for tc in state["messages"][-1].tool_calls:
        output = tools_by_name[tc["name"]].invoke(tc["args"])
        results.append(ToolMessage(content=str(output), tool_call_id=tc["id"]))
    return {"messages": results}

def should_continue(state: State) -> Literal["tool_node", "__end__"]:
    if state["messages"][-1].tool_calls:
        return "tool_node"
    return "__end__"

graph = StateGraph(State)
graph.add_node("llm", llm_node)
graph.add_node("tool_node", tool_node)
graph.add_edge(START, "llm")
graph.add_conditional_edges("llm", should_continue, ["tool_node", "__end__"])
graph.add_edge("tool_node", "llm")

agent = graph.compile()

result = agent.invoke({
    "messages": [HumanMessage(content="What is 3 + 4 multiplied by 2?")]
})
print(result["messages"][-1].content)
```

**Expected Output:**
```
14
```

The agent processes the request as follows:
1. LLM receives the question and generates tool calls for `add(3, 4)` then `multiply(7, 2)`
2. Tool node executes each tool call and returns ToolMessage results
3. LLM synthesizes final answer from tool outputs

---

## Tool Error Handling

```python
def tool_node(state: State) -> dict:
    results = []
    for tc in state["messages"][-1].tool_calls:
        try:
            tool = tools_by_name[tc["name"]]
            output = tool.invoke(tc["args"])
            content = str(output)
        except Exception as e:
            content = f"Error: {type(e).__name__}: {str(e)}"
        
        results.append(ToolMessage(content=content, tool_call_id=tc["id"]))
    return {"messages": results}
```

---

## Advanced Patterns

### Maximum Iterations

```python
class State(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
    iterations: int

def llm_node(state: State) -> dict:
    response = model_with_tools.invoke(state["messages"])
    return {"messages": [response], "iterations": state.get("iterations", 0) + 1}

def should_continue(state: State) -> str:
    if state.get("iterations", 0) >= 10:
        return "__end__"
    if state["messages"][-1].tool_calls:
        return "tool_node"
    return "__end__"
```

### Dynamic Tool Selection

```python
def llm_node(state: State) -> dict:
    if state.get("mode") == "math":
        active_tools = [add, multiply]
    else:
        active_tools = [search]
    
    model = llm.bind_tools(active_tools)
    response = model.invoke(state["messages"])
    return {"messages": [response]}
```
