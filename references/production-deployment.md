# Production Deployment

Deploy LangGraph applications to production using LangGraph Platform (cloud/self-hosted) or custom infrastructure.

## Contents

- [Deployment Options](#deployment-options)
- [LangGraph Platform](#langgraph-platform)
- [Application Structure](#application-structure)
- [LangGraph CLI](#langgraph-cli)
- [RemoteGraph Client](#remotegraph-client)
- [Self-Hosted Deployment](#self-hosted-deployment)
- [Production Checklist](#production-checklist)

---

## Deployment Options

| Option | Description | Best For |
|--------|-------------|----------|
| LangGraph Cloud | Fully managed by LangChain | Fastest deployment, minimal ops |
| Self-Hosted Platform | Run LangGraph Platform on your infra | Compliance, data sovereignty |
| Custom Docker | Roll your own with FastAPI/etc | Full control, existing infra |

---

## LangGraph Platform

LangGraph Platform is a runtime for deploying stateful agent workflows with:
- **Execution APIs** — Invoke, stream, manage runs
- **Persistence** — Built-in checkpointing and memory
- **Monitoring** — Observability via LangSmith integration
- **Scaling** — Automatic scaling for production loads

---

## Application Structure

### Required Files

```
my-langgraph-app/
├── langgraph.json      # Deployment configuration
├── requirements.txt    # Python dependencies
├── agent.py            # Graph definition
└── .env                # Local env vars (not deployed)
```

### langgraph.json

```json
{
  "dependencies": ["."],
  "graphs": {
    "my_agent": "./agent.py:graph"
  },
  "env": ".env"
}
```

| Key | Description |
|-----|-------------|
| `dependencies` | Pip packages or paths to install |
| `graphs` | Map of graph names to `file:variable` paths |
| `env` | Environment file for local development |
| `dockerfile_lines` | Optional: Extra Docker commands |

### Agent Code (agent.py)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import InMemorySaver
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

def create_graph():
    builder = StateGraph(MessagesState)
    builder.add_node("agent", call_model)
    builder.add_edge(START, "agent")

    # Use InMemorySaver for local dev; Platform provides production checkpointer
    checkpointer = InMemorySaver()
    return builder.compile(checkpointer=checkpointer)

# Export for LangGraph Platform
graph = create_graph()
```

### Extended Configuration

```json
{
  "dependencies": [
    ".",
    "langchain-openai>=0.1.0",
    "langgraph>=0.2.0"
  ],
  "graphs": {
    "agent": "./agent.py:graph",
    "researcher": "./agents/researcher.py:graph"
  },
  "env": ".env",
  "dockerfile_lines": [
    "RUN apt-get update && apt-get install -y poppler-utils",
    "RUN pip install pdf2image"
  ]
}
```

---

## LangGraph CLI

### Installation

```bash
pip install -U langgraph-cli
```

### Commands

```bash
# Deploy to LangGraph Platform
langgraph deploy --config langgraph.json

# Build Docker image locally
langgraph build --config langgraph.json

# Run locally for development
langgraph dev --config langgraph.json
```

### Development Server

```bash
# Start local development server
langgraph dev

# Server runs at http://localhost:8000
# API docs at http://localhost:8000/docs
```

---

## RemoteGraph Client

Interact with deployed graphs using `RemoteGraph`:

```python
from langgraph.pregel.remote import RemoteGraph
from langgraph_sdk import get_client

# Connect to deployed graph
url = "https://your-deployment.langgraph.com"
remote_graph = RemoteGraph(
    name="agent",
    url=url,
    api_key="your-langsmith-api-key"
)

# Use exactly like local graph
result = remote_graph.invoke({
    "messages": [{"role": "user", "content": "Hello!"}]
})

# Streaming
for chunk in remote_graph.stream({
    "messages": [{"role": "user", "content": "Tell me a story"}]
}, stream_mode="updates"):
    print(chunk)

# Thread-based persistence
client = get_client(url=url)
thread = client.threads.create()

config = {"configurable": {"thread_id": thread["thread_id"]}}
result = remote_graph.invoke(
    {"messages": [{"role": "user", "content": "Remember: code=12345"}]},
    config
)

# Continue conversation
result2 = remote_graph.invoke(
    {"messages": [{"role": "user", "content": "What was the code?"}]},
    config
)
print(result2["messages"][-1].content)  # "The code was 12345"
```

### LangGraph SDK

```bash
pip install langgraph-sdk
```

```python
from langgraph_sdk import get_client

client = get_client(url="https://your-deployment.langgraph.com")

# Create thread
thread = client.threads.create()

# Run agent
run = client.runs.create(
    thread_id=thread["thread_id"],
    graph_id="agent",
    input={"messages": [{"role": "user", "content": "Hello"}]}
)

# Wait for completion
result = client.runs.wait(thread_id=thread["thread_id"], run_id=run["run_id"])
```

---

## Self-Hosted Deployment

### Docker Deployment

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Expose the LangGraph server port
EXPOSE 8000

CMD ["python", "-m", "langgraph", "serve", "--host", "0.0.0.0", "--port", "8000"]
```

### FastAPI Wrapper

```python
from fastapi import FastAPI
from pydantic import BaseModel
from agent import graph

app = FastAPI()

class InvokeRequest(BaseModel):
    messages: list
    thread_id: str | None = None

@app.post("/invoke")
async def invoke(request: InvokeRequest):
    config = {}
    if request.thread_id:
        config = {"configurable": {"thread_id": request.thread_id}}

    result = graph.invoke(
        {"messages": request.messages},
        config
    )
    return {"messages": result["messages"]}

@app.post("/stream")
async def stream(request: InvokeRequest):
    from fastapi.responses import StreamingResponse

    def generate():
        config = {}
        if request.thread_id:
            config = {"configurable": {"thread_id": request.thread_id}}

        for chunk in graph.stream(
            {"messages": request.messages},
            config,
            stream_mode="updates"
        ):
            yield f"data: {chunk}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

### PostgreSQL Persistence

```python
from langgraph.checkpoint.postgres import PostgresSaver

def create_production_graph():
    connection_string = os.environ["DATABASE_URL"]
    checkpointer = PostgresSaver.from_conn_string(connection_string)

    builder = StateGraph(MessagesState)
    # ... add nodes and edges

    return builder.compile(checkpointer=checkpointer)
```

---

## Production Checklist

### Before Deployment

- [ ] All environment variables configured in deployment platform (not in code)
- [ ] `langgraph.json` has correct graph paths
- [ ] Dependencies pinned to specific versions
- [ ] Production checkpointer configured (PostgreSQL recommended)
- [ ] Error handling in all nodes
- [ ] Timeouts configured for LLM calls

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
# or
ANTHROPIC_API_KEY=sk-ant-...

# For LangSmith observability
LANGSMITH_API_KEY=ls-...
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=my-production-agent

# For PostgreSQL persistence
DATABASE_URL=postgresql://user:pass@host:5432/langgraph
```

### Monitoring

```python
import logging
from datetime import datetime

logger = logging.getLogger("langgraph")

def monitored_node(state):
    start = datetime.now()
    try:
        result = process(state)
        duration = (datetime.now() - start).total_seconds()
        logger.info(f"Node completed in {duration:.2f}s")
        return result
    except Exception as e:
        logger.error(f"Node failed: {e}")
        raise
```

### Human-in-the-Loop Interrupts

```python
app = workflow.compile(
    checkpointer=checkpointer,
    interrupt_before=["human_approval"],  # Pause before this node
    interrupt_after=["critical_action"]   # Pause after this node
)
```

---

## Official Resources

- [LangGraph Platform docs](https://docs.langchain.com/langsmith/langgraph-platform)
- [Application structure](https://docs.langchain.com/oss/python/langgraph/application-structure)
- [LangGraph CLI](https://langchain-ai.github.io/langgraph/cloud/reference/cli/)
- [Deployment options](https://docs.langchain.com/langgraph-platform/deployment-options)
