# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a workshop/teaching repo of self-contained example agents built on Google's Agent
Development Kit (ADK, `google-adk`). It is not a single application — it's a curriculum of ~50
small, independent agent folders grouped into numbered lesson directories (`01-agent` through
`09-model`), each demonstrating one ADK concept (basic agents, sessions/memory, tools, MCP,
workflow agents, callbacks, deployment, structured output, model providers).

## Setup and commands

- Package manager is `uv` (Python >= 3.12). The single `pyproject.toml`/`uv.lock` at the repo
  root govern the whole repo — there is no per-lesson `pyproject.toml` despite what some nested
  READMEs imply.
  ```
  uv venv --python 3.12
  source .venv/bin/activate
  uv sync
  ```
- Each runnable example needs Google credentials/model config in a `.env` file. **`.env` is
  placed in the parent lesson directory** (e.g. `03-tools/.env`), not inside the individual
  example folder — one `.env` is shared by every example under that lesson directory. Minimum
  variables:
  ```
  GOOGLE_GENAI_USE_VERTEXAI=TRUE|FALSE
  GOOGLE_GENAI_MODEL="gemini-2.5-flash"
  # Vertex AI:
  GOOGLE_CLOUD_PROJECT="..."
  GOOGLE_CLOUD_LOCATION="..."
  # or AI Studio:
  GOOGLE_API_KEY="..."
  ```
  Some examples add extra keys documented in their own `README.md` (e.g. `STOCK_API_KEY`,
  `GOOGLE_MAPS_API_KEY`).
- Authenticate to GCP when using Vertex AI: `gcloud auth application-default login`.
- Run an example via the ADK CLI from inside its lesson directory (ADK auto-discovers any
  subfolder that exposes a `root_agent`):
  ```
  cd <lesson-dir>            # e.g. cd 03-tools
  adk web                    # interactive web UI, pick the example folder from the dropdown
  adk run <example_folder>   # run one example directly from the CLI
  ```
- A few examples are plain scripts instead of `adk web` targets and are run directly, e.g.
  `01-agent/runtime/runner.py`, `07-deploy/agent_engine/run.py`, `07-deploy/operation/*.py`.
- There is no configured lint/format tooling and no real automated test suite. `pytest` and
  `pytest-asyncio` are listed as dependencies but the only "test"-named file
  (`09-model/ollama_agent/ollama_unittest.py`) is an ad-hoc script, not a pytest test — don't
  assume `pytest` will discover anything meaningful.

## Architecture

Every leaf example directory follows the same shape, which is what makes it discoverable by the
`adk` CLI:
- `__init__.py` — always just `from . import agent`, so ADK can import the package.
- `agent.py` — defines the module-level `root_agent = Agent(...)` (or a workflow agent variant).
  This exact name (`root_agent`) is the contract ADK's CLI/web UI looks for.
- `sub_agent.py` (optional) — sub-agents referenced by `agent.py` via `sub_agents=[...]` or as
  `AgentTool`s, for multi-agent examples.
- `callback.py` / `runner.py` (optional) — callback functions wired into an agent's
  `before_/after_*_callback` params, or a standalone script that builds an
  `InMemorySessionService` + `Runner` and drives the agent from the terminal instead of `adk web`.
- `README.md` — per-example setup notes (env vars, run command, sample prompt).

Multi-agent composition happens **within a single process**: a root agent routes to sub-agents
or tools, but there's no cross-process orchestration in these examples (see `01-agent/README.md`
for the ADK component/hierarchy diagrams).

Lesson directories, in the order the curriculum builds on itself:
- `01-agent` — agent basics: minimal agent, sub-agents, runtime/streaming, observability.
- `02-context` — session, state, memory (`memory_bank`), and event handling across turns.
- `03-tools` — tool integration: function calling, Google Search, code execution, BigQuery,
  Vertex AI Search/RAG Engine, LangChain (Tavily) tools, agent-as-tool.
- `04-mcp` — Model Context Protocol: stdio/Streamable HTTP MCP servers and clients, MCP Toolbox,
  filesystem/Google Maps MCP tools.
- `05-workflow` — workflow agent types: sequential, parallel, loop, general, and a custom
  critic-aggregation workflow.
- `06-callback` — before/after callbacks at the agent, model, and tool level (READMEs in Korean).
- `07-deploy` — packaging/deploying to Vertex AI Agent Engine and post-deploy operations
  (create/update/execute) (READMEs in Korean).
- `08-output` — structured output via Pydantic schemas (`output_schema`).
- `09-model` — swapping model providers: LiteLLM and local Ollama models instead of Gemini.

Each numbered directory has its own `README.md` summarizing its subfolders — check it before
adding a new example, since it should be kept in sync with the folder list.
