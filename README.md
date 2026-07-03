<div align="center">

# autodev

**Autonomous multi-agent development system driven by opencode + DeepSeek**


</div>

An 8-agent roster (planner, contractor, coder, designer, tester, researcher, arbiter, acceptor) collaborates through opencode + DeepSeek V4 to research market demand, decompose features into tasks, implement code, and verify against acceptance criteria. A FastAPI orchestrator ticks the two primary agents (planner, contractor) and exposes an MCP tool layer through which they dispatch the sub-agents; every agent runs in an ephemeral docker sandbox. State is persisted in SQLite and surfaced in a React UI with a visual dataflow graph editor for agent configuration.

## ■ Features

- ❖ **8 agents** — planner (market research), contractor (decomposition), coder, designer (UI review), tester (black-box), researcher, arbiter (goal check), acceptor (review)
- ❖ **Orchestrator** — state machine ticks only the primary agents: `idle -> planning -> contracting -> idle`; sub-agents are invoked over MCP
- ❖ **MCP tool layer** — `autodev-agents` server exposes `call` / `call_async` / `read_db` / `write_db` / `git_commit_push`, gated by per-agent permissions derived from the graph
- ❖ **Sandboxed runs** — each agent run launches a fresh docker container with the project bind-mounted and opencode-serve inside
- ❖ **React dataflow editor** — @xyflow/react canvas with agent nodes plus db/ticker/text/git meta-nodes, drag-create edges, undo/redo
- ❖ **Agent inspector** — edit role, model (Pro/Flash), tools, temperature, thinking mode, system prompt, prefill prompt, input/output format
- ❖ **Live UI** — WebSocket pub/sub pushes every CRUD mutation for instant TanStack Query invalidation
- ❖ **Run audit trail** — transcript renderer with tool calls, reasoning, token + cost tracking per run
- ❖ **Stats dashboard** — cost/tokens line charts, runs bar chart, 14-day zero-filled window
- ❖ **Auto-commit** — contractor commits + pushes via the `git_commit_push` MCP tool after the acceptor accepts a backlog item, using a per-project bot git identity

## ■ Stack

<div align="center">

| Component | Technology |
|-----------|------------|
| Backend | FastAPI + SQLAlchemy 2.0 async + aiosqlite |
| Frontend | React 18 + Vite 5 + TypeScript + TanStack Query + @xyflow/react |
| Agent runtime | opencode (sandboxed, image `opencode-runner:1.14.25`) |
| Agent comms | MCP server `autodev-agents` (FastMCP, streamable HTTP) |
| LLM | DeepSeek V4 (per-agent Pro/Flash, selectable in the inspector) |
| Web search | Serper.dev |
| Database | SQLite (projects, backlog, tasks, runs, agent sessions) |
| Migrations | Alembic |
| Design | Catppuccin Mocha (CSS variables) |

</div>

## ■ How It Works

```
1. Browser connects to FastAPI :8765 via REST/WebSocket; the orchestrator ticks planner and contractor per project in a state machine loop (idle → planning → contracting → idle).
2. Planner and contractor agents run inside ephemeral docker sandboxes (opencode-serve) and communicate back to the host through the MCP autodev-agents server at /mcp/mcp.
3. Primary agents dispatch sub-agents (coder, designer, tester, researcher, arbiter, acceptor) via MCP call / call_async tools, with permissions derived from the dataflow graph.
4. Each agent calls DeepSeek V4; results, transcripts, and token costs are stored in SQLite and pushed to the React UI instantly via WebSocket pub/sub.
5. Once the acceptor approves a backlog item, the contractor triggers git_commit_push to commit and push changes using a per-project bot git identity.
```

## ■ Screenshots

<div align="center">

![Screenshot](screenshots/main.png)

*Main interface showing the agent dataflow graph editor and project dashboard*

</div>

## ■ Usage

```bash
# Backend
make install
make serve              # uvicorn :8765

# Frontend
make web-install        # pnpm install
make web-dev            # vite :5174

# Orchestrator
make orchestrate        # continuous tick loop
make run-once           # single tick

# Sandbox runtime (docker)
make sandbox-build
make sandbox-up

# Migrations / quality
make migrate-up         # alembic upgrade head
make test               # pytest
make lint               # ruff
```

## ■ License

MIT © [pluttan](https://github.com/pluttan)
