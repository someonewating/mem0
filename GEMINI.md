# Mem0 - The Memory Layer for Personalized AI

## Project Overview

Mem0 ("mem-zero") is an intelligent memory layer that enhances AI assistants and agents with persistent, personalized memory capabilities. It enables AI systems to remember user preferences, adapt to individual needs, and continuously learn over time.

This is a **polyglot monorepo** containing Python and TypeScript SDKs, CLIs, a self-hosted server, plugins, documentation, and evaluation tooling.

### Main Technologies
- **Python:** Core SDK (`mem0ai`), FastAPI server, Typer CLI.
- **TypeScript:** Node SDK (`mem0ai`), Node CLI (`@mem0/cli`), Vercel AI provider.
- **Data Stores:** Qdrant (default vector store), PostgreSQL/pgvector, Neo4j (graph memory).
- **LLMs:** OpenAI (default), Anthropic, Gemini, Groq, Ollama, etc.
- **Build Tools:** Hatch (Python), pnpm (TypeScript), Docker Compose.

---

## Repository Structure

| Directory | Description |
|-----------|-------------|
| `mem0/` | Core Python SDK (`mem0ai` on PyPI) — memory, LLMs, embeddings, vector stores, graphs, rerankers. |
| `mem0-ts/` | TypeScript SDK (`mem0ai` on npm) — client + OSS memory. |
| `cli/python/` | Python CLI (`mem0-cli` on PyPI) — Typer-based, entry point `mem0`. |
| `cli/node/` | Node CLI (`@mem0/cli` on npm) — Commander-based, entry point `mem0`. |
| `server/` | FastAPI REST server for self-hosted Mem0 (Docker: FastAPI + PostgreSQL/pgvector + Neo4j). |
| `openmemory/` | Self-hosted memory platform (Sunset in favor of `server/`). |
| `vercel-ai-sdk/` | `@mem0/vercel-ai-provider` — Vercel AI SDK memory provider. |
| `openclaw/` | `@mem0/openclaw-mem0` — OpenClaw plugin for Claude Code / AI editors. |
| `skills/` | Claude Code skill definitions. |
| `docs/` | Documentation site (Mintlify). |
| `tests/` | Python SDK tests (pytest). |
| `evaluation/` | Benchmarking framework (LOCOMO evals). |
| `examples/` | Sample projects and demo apps. |

---

## Building and Running

### Development Setup

- **Python:** 3.9+ (3.10+ for CLI). Uses `hatch` for environment management.
- **Node.js:** v18+ (v20 or v22 recommended). Uses `pnpm` v10+.
- **Docker:** Required for `server/` development.

#### Python SDK (`mem0/`)
```bash
# Install and setup environment
hatch shell dev_py_3_11

# Linting and formatting
make lint                          # ruff check
make format                        # ruff format
make sort                          # isort mem0/

# Tests
make test                          # pytest tests/
```

#### TypeScript SDK (`mem0-ts/`)
```bash
cd mem0-ts
pnpm install
pnpm run build                     # tsup build
pnpm run test                      # run jest tests
```

#### Self-Hosted Server (`server/`)
```bash
cd server
make bootstrap                     # starts stack, creates admin, issues first API key
# or
make up                            # starts stack for browser-based wizard setup
```

---

## Development Conventions

### Coding Style
- **Python:**
  - **Linter/Formatter:** Ruff (line length **120** for core, **100** for CLI).
  - **Import Sorting:** isort (`profile = "black"`).
  - **Models:** Pydantic v2 for all data models and configuration.
  - **Pattern:** Provider pattern for LLMs, Embeddings, Vector Stores, etc. (inheriting from `base.py`).
- **TypeScript:**
  - **Strict Mode:** Enabled across all packages.
  - **Linter:** Package-specific (Biome for `cli/node`, ESLint for `vercel-ai-sdk`).
  - **Formatter:** Prettier (or Biome where applicable).
  - **Import Syntax:** Always use ES module `import`.

### Testing Practices
- **Python:** `pytest` is used for unit and integration tests.
- **Node/TS:** `jest` is used for `mem0-ts` and `vercel-ai-sdk`; `vitest` is used for `cli/node` and `openclaw`.
- **Naming:**
  - Python tests: `tests/**/test_<module>.py`
  - TS tests: `**/*.test.ts`

### Contribution Guidelines
- **Git:** Use [Conventional Commits](https://www.conventionalcommits.org/) (e.g., `feat:`, `fix:`, `docs:`).
- **PRs:** Every PR must link to an issue and pass all CI checks (linting, tests, build).
- **Pre-commit:** Run `pre-commit install` to enable automatic linting on commit.
- **Dependencies:** Avoid adding core Python dependencies; use optional groups in `pyproject.toml` for new providers.

---

## Key Core APIs

### Python `Memory` (Self-Hosted)
- `add(messages, user_id=...)`: Create memories from conversations.
- `search(query, user_id=...)`: Semantic search across memories.
- `get_all(user_id=...)`: List all memories for a user.
- `update(memory_id, data)`: Update specific memory.
- `delete(memory_id)`: Delete specific memory.

### TypeScript `MemoryClient` (Hosted Platform)
- `add(messages, { user_id })`
- `search(query, { user_id })`
- `getAll({ user_id })`

---

## Architecture: Provider Pattern

The SDK uses a consistent plugin architecture. To add a new provider:
1. Create `mem0/<category>/<provider_name>.py`.
2. Inherit from the abstract base class in `mem0/<category>/base.py`.
3. Add configuration to `mem0/<category>/configs.py`.
4. Register the provider in `mem0/<category>/__init__.py`.
5. Add tests in `tests/<category>/<provider_name>/`.
6. Add new dependencies to optional groups in `pyproject.toml`.
