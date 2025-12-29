# Official LangGraph Resources

Quick reference to official documentation, tutorials, and tools.

## Contents

- [Documentation Sites](#documentation-sites)
- [Getting Started](#getting-started)
- [Core Concepts](#core-concepts)
- [How-To Guides](#how-to-guides)
- [Agent Development](#agent-development)
- [Platform & Deployment](#platform--deployment)
- [Tools & Integrations](#tools--integrations)
- [Troubleshooting](#troubleshooting)

---

## Documentation Sites

| Resource | URL |
|----------|-----|
| **Main Docs** | https://langchain-ai.github.io/langgraph/ |
| **GitHub Repo** | https://github.com/langchain-ai/langgraph |
| **LLMs.txt Index** | https://langchain-ai.github.io/langgraph/llms.txt |
| **API Reference** | https://langchain-ai.github.io/langgraph/reference/ |
| **LangSmith (Observability)** | https://docs.langchain.com/langsmith/ |

---

## Getting Started

| Tutorial | Description |
|----------|-------------|
| [Build Basic Chatbot](https://langchain-ai.github.io/langgraph/tutorials/get-started/1-build-basic-chatbot/) | State machine setup, first graph |
| [Add Tools](https://langchain-ai.github.io/langgraph/tutorials/get-started/2-add-tools/) | Integrate web search with Tavily |
| [Add Memory](https://langchain-ai.github.io/langgraph/tutorials/get-started/3-add-memory/) | Checkpointing for conversation context |
| [Human-in-the-Loop](https://langchain-ai.github.io/langgraph/tutorials/get-started/4-human-in-the-loop/) | Pause for human input with interrupts |
| [Customize State](https://langchain-ai.github.io/langgraph/tutorials/get-started/5-customize-state/) | Custom state fields beyond messages |
| [Time Travel](https://langchain-ai.github.io/langgraph/tutorials/get-started/6-time-travel/) | Rewind, add steps, replay history |

---

## Core Concepts

| Concept | URL |
|---------|-----|
| [Why LangGraph](https://langchain-ai.github.io/langgraph/concepts/why-langgraph/) | Reliability, extensibility, streaming |
| [Low-Level Concepts](https://langchain-ai.github.io/langgraph/concepts/low_level/) | States, Nodes, Edges |
| [Agentic Concepts](https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/) | Routers, tool-calling, planning |
| [Persistence](https://langchain-ai.github.io/langgraph/concepts/persistence/) | Checkpointers for state saving |
| [Memory](https://langchain-ai.github.io/langgraph/concepts/memory/) | Short-term and long-term memory |
| [Streaming](https://langchain-ai.github.io/langgraph/concepts/streaming/) | Real-time updates |
| [Human-in-the-Loop](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/) | Intervention points |
| [Breakpoints](https://langchain-ai.github.io/langgraph/concepts/breakpoints/) | Pause at specific points |
| [Time Travel](https://langchain-ai.github.io/langgraph/concepts/time-travel/) | Resume from checkpoints |
| [Durable Execution](https://langchain-ai.github.io/langgraph/concepts/durable_execution/) | Save progress, resume |
| [Subgraphs](https://langchain-ai.github.io/langgraph/concepts/subgraphs/) | Modular graph composition |
| [Multi-Agent](https://langchain-ai.github.io/langgraph/concepts/multi_agent/) | Supervisor, swarm, hierarchical |
| [Functional API](https://langchain-ai.github.io/langgraph/concepts/functional_api/) | @entrypoint and @task decorators |
| [Tools](https://langchain-ai.github.io/langgraph/concepts/tools/) | Tool calling patterns |

---

## How-To Guides

### Graph API
| Guide | URL |
|-------|-----|
| [Graph API Overview](https://langchain-ai.github.io/langgraph/how-tos/graph-api/) | State, nodes, control flow |
| [Streaming](https://langchain-ai.github.io/langgraph/how-tos/streaming/) | Sync and async streaming |
| [Persistence](https://langchain-ai.github.io/langgraph/how-tos/persistence/) | Memory implementations |
| [Memory Management](https://langchain-ai.github.io/langgraph/how-tos/memory/) | Trimming, summarizing, deleting |
| [Tool Calling](https://langchain-ai.github.io/langgraph/how-tos/tool-calling/) | Tools with error handling |
| [Subgraphs](https://langchain-ai.github.io/langgraph/how-tos/subgraph/) | Shared/different state schemas |
| [Multi-Agent](https://langchain-ai.github.io/langgraph/how-tos/multi_agent/) | Agent handoffs |
| [Functional API](https://langchain-ai.github.io/langgraph/how-tos/use-functional-api/) | Retry, caching, HITL |

### Human-in-the-Loop
| Guide | URL |
|-------|-----|
| [Add HITL](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/add-human-in-the-loop/) | interrupt() function |
| [Breakpoints](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/breakpoints/) | Static and dynamic |
| [Time Travel](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/time-travel/) | Debugging, exploration |

---

## Agent Development

| Resource | URL |
|----------|-----|
| [Agents Overview](https://langchain-ai.github.io/langgraph/agents/overview/) | Prebuilt components |
| [Running Agents](https://langchain-ai.github.io/langgraph/agents/run_agents/) | Sync/async execution |
| [Streaming](https://langchain-ai.github.io/langgraph/agents/streaming/) | Progress, tokens, updates |
| [Models](https://langchain-ai.github.io/langgraph/agents/models/) | Tool calling, providers |
| [Tools](https://langchain-ai.github.io/langgraph/agents/tools/) | Defining, error handling |
| [MCP Integration](https://langchain-ai.github.io/langgraph/agents/mcp/) | Model Context Protocol |
| [Context](https://langchain-ai.github.io/langgraph/agents/context/) | Config, State, Long-Term Memory |
| [Memory](https://langchain-ai.github.io/langgraph/agents/memory/) | Short and long-term |
| [Human-in-the-Loop](https://langchain-ai.github.io/langgraph/agents/human-in-the-loop/) | Tool call approval |
| [Multi-Agent](https://langchain-ai.github.io/langgraph/agents/multi-agent/) | Supervisor and swarm |
| [Evaluation](https://langchain-ai.github.io/langgraph/agents/evals/) | LangSmith testing |
| [Deployment](https://langchain-ai.github.io/langgraph/agents/deployment/) | Local and production |
| [Agent UI](https://langchain-ai.github.io/langgraph/agents/ui/) | Chat UI integration |

---

## Platform & Deployment

### LangGraph Platform
| Resource | URL |
|----------|-----|
| [Platform Overview](https://langchain-ai.github.io/langgraph/concepts/langgraph_platform/) | Streaming, background runs, memory |
| [Components](https://langchain-ai.github.io/langgraph/concepts/langgraph_components/) | Server, CLI, Studio, SDKs |
| [Server](https://langchain-ai.github.io/langgraph/concepts/langgraph_server/) | API for agent applications |
| [Application Structure](https://langchain-ai.github.io/langgraph/concepts/application_structure/) | Config file, dependencies, graphs |
| [Deployment Options](https://langchain-ai.github.io/langgraph/concepts/deployment_options/) | Cloud, Self-Hosted, Standalone |

### Deployment Guides
| Guide | URL |
|-------|-----|
| [Local Server](https://langchain-ai.github.io/langgraph/tutorials/langgraph-platform/local-server/) | CLI and Studio |
| [Cloud Deployment](https://langchain-ai.github.io/langgraph/cloud/deployment/cloud/) | GitHub repos |
| [Self-Hosted Data Plane](https://langchain-ai.github.io/langgraph/cloud/deployment/self_hosted_data_plane/) | Kubernetes, ECS |
| [Self-Hosted Control Plane](https://langchain-ai.github.io/langgraph/cloud/deployment/self_hosted_control_plane/) | Full self-management |
| [Standalone Container](https://langchain-ai.github.io/langgraph/cloud/deployment/standalone_container/) | Docker, Helm |
| [Custom Docker](https://langchain-ai.github.io/langgraph/cloud/deployment/custom_docker/) | Dockerfile via langgraph.json |
| [RemoteGraph](https://langchain-ai.github.io/langgraph/how-tos/use-remote-graph/) | Connect to deployed graphs |

### CLI & Studio
| Resource | URL |
|----------|-----|
| [CLI Overview](https://langchain-ai.github.io/langgraph/concepts/langgraph_cli/) | Build and run API server |
| [Studio Overview](https://langchain-ai.github.io/langgraph/concepts/langgraph_studio/) | Visual debugging IDE |
| [Studio Quick Start](https://langchain-ai.github.io/langgraph/cloud/how-tos/studio/quick_start/) | Connect to deployments |

---

## Tools & Integrations

| Resource | URL |
|----------|-----|
| [SDK Overview](https://langchain-ai.github.io/langgraph/concepts/sdk/) | Python and JS SDKs |
| [MCP Server](https://langchain-ai.github.io/langgraph/concepts/server-mcp/) | Model Context Protocol |
| [Webhooks](https://langchain-ai.github.io/langgraph/cloud/concepts/webhooks/) | Event-driven integration |
| [Cron Jobs](https://langchain-ai.github.io/langgraph/cloud/concepts/cron_jobs/) | Scheduled execution |
| [Authentication](https://langchain-ai.github.io/langgraph/concepts/auth/) | Auth vs authorization |
| [Custom Auth](https://langchain-ai.github.io/langgraph/how-tos/auth/custom_auth/) | Implementation guide |

---

## Troubleshooting

| Error | URL |
|-------|-----|
| [Error Index](https://langchain-ai.github.io/langgraph/troubleshooting/errors/index/) | Common errors |
| [GRAPH_RECURSION_LIMIT](https://langchain-ai.github.io/langgraph/troubleshooting/errors/GRAPH_RECURSION_LIMIT/) | Recursion limits |
| [INVALID_CONCURRENT_GRAPH_UPDATE](https://langchain-ai.github.io/langgraph/troubleshooting/errors/INVALID_CONCURRENT_GRAPH_UPDATE/) | Concurrent state conflicts |
| [INVALID_GRAPH_NODE_RETURN_VALUE](https://langchain-ai.github.io/langgraph/troubleshooting/errors/INVALID_GRAPH_NODE_RETURN_VALUE/) | Node return validation |
| [INVALID_CHAT_HISTORY](https://langchain-ai.github.io/langgraph/troubleshooting/errors/INVALID_CHAT_HISTORY/) | Message format |
| [Studio Issues](https://langchain-ai.github.io/langgraph/troubleshooting/studio/) | Connection, routing |

---

## Tutorials

| Tutorial | URL |
|----------|-----|
| [Workflows & Agents](https://langchain-ai.github.io/langgraph/tutorials/workflows/) | Agentic patterns |
| [Agentic RAG](https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_agentic_rag/) | Retrieval systems |
| [Agent Supervisor](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/agent_supervisor/) | Multi-agent orchestration |
| [SQL Agent](https://langchain-ai.github.io/langgraph/tutorials/sql-agent/) | Database queries |
| [Auth Getting Started](https://langchain-ai.github.io/langgraph/tutorials/auth/getting_started/) | Token-based auth |
| [Deployment Tutorial](https://langchain-ai.github.io/langgraph/tutorials/deployment/) | Local and cloud |

---

## GitHub Repositories

| Repository | URL |
|------------|-----|
| **langgraph** | https://github.com/langchain-ai/langgraph |
| **langgraph-supervisor** | https://github.com/langchain-ai/langgraph-supervisor-py |
| **langgraph-swarm** | https://github.com/langchain-ai/langgraph-swarm-py |
| **langgraph-studio** | https://github.com/langchain-ai/langgraph-studio |
| **langgraphjs** | https://github.com/langchain-ai/langgraphjs |

---

## Fetching Latest Docs

For always up-to-date documentation, fetch the llms.txt index:

```python
# Using WebFetch or requests
url = "https://langchain-ai.github.io/langgraph/llms.txt"
# Returns structured list of all documentation URLs with descriptions
```
