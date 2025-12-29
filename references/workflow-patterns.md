# Workflow Patterns

Build structured multi-step pipelines with branching, parallel execution, and prompt chaining.

## Contents

- [Workflows vs Agents](#workflows-vs-agents)
- [Sequential Chain](#sequential-chain)
- [Conditional Branching](#conditional-branching)
- [Parallel Execution](#parallel-execution)
- [Prompt Chaining](#prompt-chaining)
- [Map-Reduce Pattern](#map-reduce-pattern)
- [Complete Example](#complete-example)

---

## Workflows vs Agents

| Aspect | Workflow | Agent |
|--------|----------|-------|
| Flow | Predetermined steps | LLM decides next step |
| Branching | Explicit conditions | Tool availability |
| Predictability | High | Variable |

Use **workflows** when the sequence is known ahead of time.

---

## Sequential Chain

A → B → C → END

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

class State(TypedDict):
    input: str
    step1_output: str
    step2_output: str

def step1(state: State) -> dict:
    return {"step1_output": f"Processed: {state['input']}"}

def step2(state: State) -> dict:
    return {"step2_output": f"Final: {state['step1_output']}"}

graph = StateGraph(State)
graph.add_node("step1", step1)
graph.add_node("step2", step2)
graph.add_edge(START, "step1")
graph.add_edge("step1", "step2")
graph.add_edge("step2", END)

chain = graph.compile()
```

---

## Conditional Branching

```
classify → handler_a → END
         → handler_b → END
```

### Router Function

```python
def route_by_category(state: State) -> str:
    category = state["category"]
    if category == "urgent":
        return "urgent_handler"
    elif category == "normal":
        return "normal_handler"
    return "fallback_handler"
```

### Adding Conditional Edges

```python
graph.add_conditional_edges(
    "classify",
    route_by_category,
    ["urgent_handler", "normal_handler", "fallback_handler"]
)
```

### Pass/Fail Loop

```python
def check_quality(state: State) -> str:
    return "accept" if state["score"] >= 0.8 else "revise"

graph.add_conditional_edges("evaluate", check_quality, ["accept", "revise"])
graph.add_edge("accept", END)
graph.add_edge("revise", "improve")
graph.add_edge("improve", "evaluate")  # Loop back
```

---

## Parallel Execution

Fan out to multiple nodes, then join:

```python
from typing_extensions import Annotated
import operator

class State(TypedDict):
    text: str
    sentiment: str
    entities: Annotated[list[str], operator.add]
    summary: str

def analyze_sentiment(state: State) -> dict:
    return {"sentiment": "positive"}

def extract_entities(state: State) -> dict:
    return {"entities": ["entity1", "entity2"]}

def summarize(state: State) -> dict:
    return {"summary": "Brief summary..."}

graph = StateGraph(State)
graph.add_node("sentiment", analyze_sentiment)
graph.add_node("entities", extract_entities)
graph.add_node("summarize", summarize)
graph.add_node("combine", combine_results)

# Fan out
graph.add_edge(START, "sentiment")
graph.add_edge(START, "entities")
graph.add_edge(START, "summarize")

# Fan in
graph.add_edge("sentiment", "combine")
graph.add_edge("entities", "combine")
graph.add_edge("summarize", "combine")
graph.add_edge("combine", END)
```

LangGraph waits for all parallel branches before executing `combine`.

---

## Prompt Chaining

Sequential LLM calls building on each other:

```python
class State(TypedDict):
    topic: str
    outline: str
    draft: str
    final: str

def generate_outline(state: State) -> dict:
    prompt = f"Create an outline for: {state['topic']}"
    return {"outline": llm.invoke(prompt).content}

def write_draft(state: State) -> dict:
    prompt = f"Write a draft from this outline:\n{state['outline']}"
    return {"draft": llm.invoke(prompt).content}

def polish(state: State) -> dict:
    prompt = f"Polish this draft:\n{state['draft']}"
    return {"final": llm.invoke(prompt).content}

graph = StateGraph(State)
graph.add_node("outline", generate_outline)
graph.add_node("draft", write_draft)
graph.add_node("polish", polish)

graph.add_edge(START, "outline")
graph.add_edge("outline", "draft")
graph.add_edge("draft", "polish")
graph.add_edge("polish", END)
```

---

## Map-Reduce Pattern

```python
class State(TypedDict):
    documents: list[str]
    summaries: Annotated[list[str], operator.add]
    final_summary: str

def map_summarize(state: State) -> dict:
    summaries = []
    for doc in state["documents"]:
        summaries.append(llm.invoke(f"Summarize: {doc}").content)
    return {"summaries": summaries}

def reduce_combine(state: State) -> dict:
    combined = "\n".join(state["summaries"])
    final = llm.invoke(f"Combine:\n{combined}").content
    return {"final_summary": final}

graph = StateGraph(State)
graph.add_node("map", map_summarize)
graph.add_node("reduce", reduce_combine)
graph.add_edge(START, "map")
graph.add_edge("map", "reduce")
graph.add_edge("reduce", END)
```

---

## Complete Example

Joke workflow with quality check:

```python
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI
from typing_extensions import TypedDict

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

class JokeState(TypedDict):
    topic: str
    joke: str
    improved_joke: str
    final_joke: str

def generate_joke(state: JokeState) -> dict:
    response = llm.invoke(f"Tell a short joke about {state['topic']}.")
    return {"joke": response.content}

def check_punchline(state: JokeState) -> str:
    joke = state["joke"]
    return "pass" if ("?" in joke or "!" in joke) else "fail"

def improve_joke(state: JokeState) -> dict:
    response = llm.invoke(f"Make this funnier:\n{state['joke']}")
    return {"improved_joke": response.content}

def polish_joke(state: JokeState) -> dict:
    source = state.get("improved_joke") or state["joke"]
    response = llm.invoke(f"Add a twist:\n{source}")
    return {"final_joke": response.content}

graph = StateGraph(JokeState)
graph.add_node("generate", generate_joke)
graph.add_node("improve", improve_joke)
graph.add_node("polish", polish_joke)

graph.add_edge(START, "generate")
graph.add_conditional_edges("generate", check_punchline, {"pass": "polish", "fail": "improve"})
graph.add_edge("improve", "polish")
graph.add_edge("polish", END)

chain = graph.compile()
result = chain.invoke({"topic": "programmers"})
```

Flow: `START → generate → [pass] → polish → END`  
Or: `START → generate → [fail] → improve → polish → END`
