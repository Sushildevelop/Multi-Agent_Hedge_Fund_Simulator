# Dependency Guide — Multi-Agent Stock Investment Advisor (Python 3.13)

This document explains the `requirements.txt`, `requirements-dev.txt`, and `requirements-prod.txt` files: what's in them, why, how they were verified for Python 3.13, and how to manage them going forward.

> **A note on currency.** This ecosystem moves fast — LangChain and LangGraph both shipped a major `1.x` rewrite in the last year, and packages like `torch`, `transformers`, and `qdrant-client` release new minor versions monthly. The versions pinned here were confirmed against PyPI at the time this was generated. Before you lock these for production, run a resolution pass (`uv lock` or `pip-compile --generate-hashes`) to confirm the exact latest compatible patch — do not treat these pins as permanently correct.

---

## 1. `requirements.txt`

See the generated file. It contains pinned, mutually-compatible versions for every package category below, verified to install cleanly on **Python 3.13.x**.

---

## 2. Categorized Packages

### Web Framework
`fastapi`, `uvicorn`, `starlette`, `python-multipart`

### AI / Agent Framework
`langchain`, `langchain-core`, `langchain-text-splitters`, `langchain-community`, `langchain-openai`, `langgraph`, `langgraph-checkpoint`, `langgraph-sdk`, `langsmith`

### LLM Providers
`groq`, `openai`

### Embeddings
`langchain-huggingface`, `sentence-transformers`, `transformers`, `tokenizers`, `huggingface-hub`, `torch`

### Vector Database
`qdrant-client`, `langchain-qdrant`

### Database (MongoDB)
`beanie`, `motor`, `pymongo`

### Queue / Caching
`redis`, `hiredis`, `celery`

### Financial APIs
`yfinance`, `alpha_vantage`, `newsapi-python`

### Document Processing
`pymupdf`, `unstructured`, `pypdf`, `python-docx`

### Data / Scientific
`pandas`, `numpy`, `scikit-learn`

### Config & Validation
`pydantic`, `pydantic-settings`, `python-dotenv`

### Auth & Security
`pyjwt`, `passlib`, `bcrypt`

### HTTP / Async
`httpx`, `aiohttp`

### Utilities
`orjson`, `tenacity`, `tiktoken`

### Logging
`structlog`, `loguru`

### Monitoring / Observability
`opentelemetry-api`, `opentelemetry-sdk`, `opentelemetry-instrumentation-fastapi`, `opentelemetry-exporter-otlp`, `prometheus-client`

### Testing (dev only)
`pytest`, `pytest-asyncio`, `pytest-cov`, `pytest-mock`, `locust`, `mongomock-motor`, `fakeredis`

---

## 3. Every Package Explained

| Package | Why it's needed | Used by | Required? |
|---|---|---|---|
| `fastapi` | Async web framework; request/response validation via Pydantic; auto-generated OpenAPI docs | `app/main.py`, all `app/api/**` routes | Required |
| `uvicorn[standard]` | ASGI server that actually runs the FastAPI app; `[standard]` extra pulls in `uvloop`/`httptools` for performance | Entrypoint / Docker CMD | Required |
| `starlette` | FastAPI's underlying ASGI toolkit (middleware, routing primitives); pinned explicitly to avoid resolver ambiguity | Transitive, via FastAPI | Required (transitive, pin for reproducibility) |
| `python-multipart` | Enables `multipart/form-data` parsing (file uploads — e.g. uploading a filing PDF via the API) | Upload endpoints | Required if any endpoint accepts file uploads |
| `langchain` | High-level chains, prompt templates, and retriever abstractions used across the RAG pipeline | `app/rag/**` | Required |
| `langchain-core` | Core interfaces (Runnable, messages, output parsers) that both `langchain` and `langgraph` build on | Everywhere agent/RAG code touches LangChain types | Required |
| `langchain-text-splitters` | Section-aware and recursive text splitters for chunking filings, transcripts, news | `app/rag/chunking.py` | Required |
| `langchain-community` | Community-maintained loaders/integrations (e.g. certain document loaders) not yet promoted to standalone packages. **Note:** LangChain has announced this package is being sunset in favor of standalone partner packages — see Section 4. | Loader utilities in `app/rag/loaders/` | Required short-term; plan migration |
| `langchain-openai` | Optional OpenAI chat/embedding model integration for LangChain's interface | `app/agents/*` when `USE_OPENAI_FALLBACK=true` | Optional |
| `langgraph` | The state-machine orchestration engine wiring the five agents into a fan-out/fan-in graph | `app/graph/**` | Required |
| `langgraph-checkpoint` | Checkpointer implementation giving the graph durable state/memory across steps (agent memory requirement) | `app/graph/build_graph.py` | Required |
| `langgraph-sdk` | Client SDK for interacting with a deployed LangGraph runtime/API (used if you run LangGraph Platform/CLI locally) | Dev tooling, optional runtime client | Optional |
| `langsmith` | Tracing/observability for LangChain and LangGraph runs; pairs with OpenTelemetry for full request tracing | Agent execution tracing | Optional (strongly recommended for debugging) |
| `groq` | Official Groq SDK for low-latency Llama 3 / DeepSeek inference | All agent LLM calls | Required |
| `openai` | Official OpenAI SDK, used both directly and as the transitive dependency of `langchain-openai` | Optional fallback path | Optional |
| `langchain-huggingface` | LangChain-compatible wrapper around HuggingFace embedding/chat models | `app/rag/embeddings.py` | Required |
| `sentence-transformers` | Runs `BAAI/bge-large-en-v1.5` locally for dense embedding generation | `app/rag/embeddings.py` | Required |
| `transformers` | Underlying model-loading library `sentence-transformers` depends on | Transitive + direct tokenizer use | Required |
| `tokenizers` | Fast Rust-backed tokenization used by `transformers` | Transitive | Required (transitive, pinned for reproducibility) |
| `huggingface-hub` | Downloads/caches model weights from the Hub | Transitive, embedding model loading | Required |
| `torch` | Tensor backend required to actually run `sentence-transformers`/`transformers` models locally | Embedding generation | Required (unless you go 100% OpenAI-embeddings) |
| `qdrant-client` | Official Qdrant SDK — collection management, upserts, filtered vector search | `app/vectorstore/qdrant_client.py` | Required |
| `langchain-qdrant` | LangChain-compatible retriever wrapper around Qdrant (enables MMR/self-query/multi-query retriever abstractions) | `app/rag/retrievers.py` | Required |
| `beanie` | Async ODM for MongoDB, built on Pydantic — models portfolios, recommendation history, users | `app/models/db_models.py` | Required |
| `motor` | Async MongoDB driver Beanie runs on top of | Transitive (Beanie dependency) + direct use for raw aggregations | Required |
| `pymongo` | Underlying MongoDB driver both Motor and Beanie depend on | Transitive | Required (transitive, pinned) |
| `redis` | Async Redis client for caching and as the Celery broker/result backend | `app/core/cache.py`, `app/workers/celery_app.py` | Required |
| `hiredis` | C-accelerated Redis protocol parser, drastically reduces Redis client overhead | Transitive perf dependency of `redis` | Optional but strongly recommended |
| `celery` | Distributed task queue running background ingestion/re-analysis jobs against the Redis broker | `app/workers/**` | Required |
| `yfinance` | Yahoo Finance client for price history and basic fundamentals | `app/data_sources/yahoo_finance.py` | Required |
| `alpha_vantage` | Secondary fundamentals/financial statement data source | `app/data_sources/alpha_vantage.py` | Required |
| `newsapi-python` | NewsAPI client for the News Agent's raw article ingestion | `app/data_sources/news_api.py` | Required |
| `pymupdf` | Fast native PDF text/layout extraction for large 10-K/10-Q filings | `app/rag/loaders/pdf_loader.py` | Required |
| `unstructured` | Layout-aware parsing across heterogeneous document formats (PDF, DOCX, HTML) with section detection | `app/rag/loaders/**` | Required |
| `pypdf` | Lightweight pure-Python PDF fallback parser for simple documents where PyMuPDF/`unstructured` are overkill | `app/rag/loaders/pdf_loader.py` (fallback path) | Required (fallback) |
| `python-docx` | Reads/writes `.docx` files (e.g. ingesting research notes supplied as Word docs) | `app/rag/loaders/docx_loader.py` | Required if DOCX ingestion is enabled |
| `pandas` | Tabular manipulation of price history and financial statement data; indicator computation | `app/agents/technical_agent.py`, data pipeline | Required |
| `numpy` | Numeric backend for pandas and indicator math | Transitive + direct use | Required |
| `scikit-learn` | Utility functions for confidence-score calibration / simple statistical helpers | `app/agents/decision_agent.py` (calibration) | Optional (only if calibration logic is implemented) |
| `pydantic` | Schema definition and validation for every API request/response and every agent's structured output | Everywhere | Required |
| `pydantic-settings` | Typed, environment-variable-driven application configuration | `app/core/config.py` | Required |
| `python-dotenv` | Loads `.env` files in local development | `app/core/config.py` (via `pydantic-settings`) | Required for local dev |
| `pyjwt[crypto]` | JWT issuance/verification for API authentication; `[crypto]` extra enables RS256/ES256 algorithms | `app/core/security.py` | Required |
| `passlib[bcrypt]` | Password hashing abstraction layer | `app/core/security.py` | Required |
| `bcrypt` | The actual bcrypt hashing implementation Passlib wraps | Transitive + pinned directly (Passlib's bcrypt backend detection is version-sensitive) | Required |
| `httpx` | Async HTTP client used for most outbound calls (LLM SDKs, some data source clients) | Widely used | Required |
| `aiohttp` | Async HTTP client required by some financial/news API client libraries that don't support `httpx` | Data source clients | Required (transitive for some clients) |
| `orjson` | Fast JSON serialization, used to speed up FastAPI response encoding for large recommendation payloads | FastAPI response class override | Optional but recommended |
| `tenacity` | Retry/backoff decorator wrapping flaky external API calls (rate limits, transient network errors) | `app/data_sources/**`, LLM call wrappers | Required |
| `tiktoken` | Token counting for prompt-size budgeting before injecting retrieved context | `app/rag/chunking.py`, prompt assembly | Required |
| `structlog` | Structured, correlation-ID-friendly logging | `app/core/logging.py` | Required |
| `loguru` | Simpler ergonomic logging used in scripts/CLI tooling alongside structlog for app logs | `scripts/**` | Optional |
| `opentelemetry-api` / `-sdk` | Distributed tracing instrumentation core | `app/core/telemetry.py` | Required |
| `opentelemetry-instrumentation-fastapi` | Auto-instruments FastAPI routes for tracing | App startup | Required |
| `opentelemetry-exporter-otlp` | Ships traces to an OTLP collector (e.g. feeding Grafana Tempo/Jaeger) | App startup | Required |
| `prometheus-client` | Exposes the `/metrics` endpoint Prometheus scrapes | `app/api/v1/routes_health.py` or dedicated metrics route | Required |
| `pytest` / `pytest-asyncio` | Test framework + async test support (the entire codebase is async) | `tests/**` | Required (dev) |
| `pytest-cov` | Coverage reporting | CI pipeline | Required (dev) |
| `pytest-mock` | Convenient mocking fixtures for LLM/Qdrant/API stubs in unit tests | `tests/unit/**` | Required (dev) |
| `locust` | Load-testing framework simulating concurrent `/analysis` traffic | `tests/load/**` | Required (dev) |
| `mongomock-motor` | In-memory MongoDB mock compatible with Motor's async API, for fast unit tests without a real MongoDB | `tests/unit/**` | Required (dev) |
| `fakeredis` | In-memory Redis mock for testing caching/queue logic without a real Redis instance | `tests/unit/**` | Required (dev) |
| `ruff` | Fast linter (replaces flake8 + many plugins) | Pre-commit, CI | Required (dev) |
| `black` | Code formatter | Pre-commit, CI | Required (dev) |
| `isort` | Import sorting (configured to be Black-compatible) | Pre-commit, CI | Required (dev) |
| `mypy` | Static type checking | CI | Required (dev) |
| `pre-commit` | Git hook runner wiring the above tools into every commit | Local dev | Required (dev) |

---

## 4. Package Conflicts & Python 3.13 Notes

- **`torch` and Python 3.13** — `torch` officially ships `cp313` wheels as of the 2.x series used here (`torch==2.13.0`); confirm this on install since PyTorch's Python-version support historically trails new CPython releases by a few months. If you ever need Python 3.14, note that PyPI has already dropped free-threaded `3.13t` wheels in favor of `3.14t` — don't target the free-threaded build for this project; use the standard (GIL-enabled) `cp313` interpreter.
- **`torch` install size/index** — the default PyPI `torch` wheel now bundles CUDA 13.0 by default on Linux. If you're deploying to a CPU-only container (typical for this project, since inference is delegated to Groq/OpenAI and Torch is only used locally for embeddings), install from the CPU-only index instead to avoid multi-gigabyte CUDA dependencies:
  ```bash
  pip install torch==2.13.0 --index-url https://download.pytorch.org/whl/cpu
  ```
- **`langchain-community` sunset notice** — LangChain has publicly signaled that `langchain-community` is being phased out in favor of standalone partner packages and `langchain-classic`. It still installs and works today, but treat any loader you pull from it as a migration candidate; check whether a standalone package (e.g. a dedicated loader package) already exists before adding new `langchain-community` usage.
- **LangChain / LangGraph major version alignment** — `langchain`, `langchain-core`, `langgraph`, and `langchain-openai` all moved through a `1.0` rewrite. Do **not** mix a `0.x` version of any of these with `1.x` versions of the others — pin all of them to their `1.x` lines together, as done in this file, or you will hit import-path breakage (e.g. `langchain.schema` → `langchain_core.outputs` relocations).
- **`beanie` Python ceiling** — `beanie` currently declares `Requires-Python <3.14, >=3.10`, i.e. it explicitly supports 3.13 but **not yet 3.14**. Do not upgrade the interpreter past 3.13 without checking Beanie's compatibility first.
- **`pydantic` v2 everywhere** — `beanie`, `fastapi`, `pydantic-settings`, and `qdrant-client` all depend on Pydantic v2. Never pin any transitive dependency to a Pydantic v1-only version — this is the single most common cause of cryptic `ValidationError`/import errors in this stack.
- **`bcrypt` + `passlib` version sensitivity** — Passlib's bcrypt backend probes `bcrypt.__about__` at import time, which newer `bcrypt` releases removed; pinning `bcrypt` explicitly (as done here) avoids `AttributeError` on startup. If you upgrade `bcrypt` independently, re-verify Passlib's backend still loads.
- **`unstructured` heavy optional dependencies** — the base `unstructured` package is lightweight, but format-specific extras (`unstructured[pdf,docx]`, as pinned) pull in additional native dependencies (e.g. `libmagic`). On Debian-based Docker images install `libmagic1` at the OS package level, not via pip.
- **`hiredis` C extension** — requires a compiler toolchain at install time on platforms without prebuilt wheels; on slim Docker base images, install build-essential (or use a wheel-only install with `--only-binary=:all:`) to avoid a failed build.
- **`grpcio` (transitive via `qdrant-client`)** — `grpcio` wheels for brand-new CPython minor versions sometimes lag by a few weeks after release; if `qdrant-client` fails to resolve on a fresh Python 3.13 image, that's almost always the cause — check for a `grpcio` wheel matching your exact platform before falling back to a source build.
- **No known direct conflicts** between `celery`, `redis`, and `fastapi`'s async stack, but note `celery` itself is a sync framework — the app's task functions in `app/workers/tasks.py` should not call async agent code directly; either wrap with `asyncio.run()` inside the task or use a sync-safe client path.

---

## 5. Recommended Installation Order

Installing in this order avoids the most common resolver backtracking issues in this stack (large packages with many transitive pins should go first so the resolver locks their sub-dependencies before smaller packages add conflicting constraints):

```bash
# 1. Create and activate the environment
python3.13 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip

# 2. Core numeric/ML stack first (heaviest, most constrained dependency trees)
pip install numpy pandas scikit-learn
pip install torch==2.13.0 --index-url https://download.pytorch.org/whl/cpu

# 3. HuggingFace stack (depends on torch)
pip install huggingface-hub tokenizers transformers sentence-transformers

# 4. LangChain / LangGraph ecosystem (pin all 1.x packages together)
pip install langchain-core langchain-text-splitters langchain langchain-community langchain-openai
pip install langgraph langgraph-checkpoint langgraph-sdk langsmith
pip install langchain-huggingface langchain-qdrant

# 5. Vector DB + Database clients
pip install qdrant-client
pip install pymongo motor beanie

# 6. Web framework + async infra
pip install "fastapi[standard]" uvicorn starlette python-multipart
pip install redis hiredis celery

# 7. Remaining application dependencies
pip install -r requirements.txt   # resolves/confirms everything above plus the rest
```

Or, more simply, once the file is finalized and version-locked:

```bash
pip install -r requirements.txt
```

The staged order above is primarily useful when you're debugging a resolver conflict — for normal use, a single `pip install -r requirements.txt` (or, better, `uv pip install -r requirements.txt`) is sufficient and faster.

---

## 6. Optional Packages

| Package | Purpose | When to include |
|---|---|---|
| `openai` / `langchain-openai` | Optional GPT-family fallback for LLM calls and/or `text-embedding-3-large` embeddings | Include if `USE_OPENAI_FALLBACK=true` or you want higher-fidelity embeddings than `bge-large-en-v1.5` |
| `langgraph-sdk` | Client for a deployed LangGraph Platform/CLI server | Only if running LangGraph's managed runtime locally or in staging |
| `langsmith` | Tracing/eval platform integration | Strongly recommended for debugging agent behavior, but not required for the app to function |
| `scikit-learn` | Confidence-score calibration helpers | Only if the Decision Agent's calibration logic uses it; otherwise drop |
| `loguru` | Simpler logging for scripts | Optional alongside `structlog` |
| `orjson` | Faster JSON responses | Optional performance improvement, safe to drop for a marginally slower default JSON encoder |
| `hiredis` | Faster Redis parsing | Optional, safe to drop on platforms where the C extension won't build |

---

## 7. Development Packages

See `requirements-dev.txt`. Install alongside the base requirements:

```bash
pip install -r requirements.txt -r requirements-dev.txt
pre-commit install
```

---

## 8. Production Packages

See `requirements-prod.txt` — the same functional set as `requirements.txt` minus all testing/linting/type-checking tooling. Use this file (or better, a hash-locked export of it) as the base for your production Docker image so build tooling never ships into the runtime image.

---

## 9. Security-Related Package Recommendations

| Concern | Package(s) | Notes |
|---|---|---|
| **JWT** | `pyjwt[crypto]` | The `[crypto]` extra pulls in `cryptography`, required for RS256/ES256; skip it only if you're certain you'll stay on HS256 forever. |
| **Password hashing** | `passlib[bcrypt]`, `bcrypt` | Never hand-roll hashing; keep `bcrypt` pinned per the note in Section 4. |
| **CORS** | Built into `starlette`/`fastapi` (`from fastapi.middleware.cors import CORSMiddleware`) — no extra package needed | Configure an explicit origin allow-list per environment; never `allow_origins=["*"]` alongside `allow_credentials=True`. |
| **Environment variables** | `python-dotenv`, `pydantic-settings` | `pydantic-settings` gives typed, validated config with clear errors on missing required secrets at startup, rather than silent `None`s. |
| **Secrets management (production)** | No pip package required — use your cloud provider's secrets manager (AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault) and inject as environment variables at deploy time | Do not commit `.env` to version control; `.env.example` only. |
| **Dependency vulnerability scanning** | `pip-audit` (recommend adding to CI, not runtime) | Run `pip-audit -r requirements-prod.txt` in CI on every PR. |
| **Rate limiting** | Implement via `slowapi` or a Redis-backed token-bucket using the already-included `redis` package | Not included by default above — add `slowapi` if you want a drop-in FastAPI rate-limit middleware instead of hand-rolling one against Redis. |

---

## 10. Best Practices

- **Use `uv` (or Poetry) instead of raw `pip` for day-to-day dependency management.** `uv` resolves this stack significantly faster than pip and produces a proper lock file:
  ```bash
  uv init
  uv add -r requirements.txt
  uv lock
  uv sync
  ```
- **Commit a lock file** (`uv.lock` or `poetry.lock`), not just `requirements.txt`. The pinned versions in `requirements.txt` fix *direct* dependencies; a lock file additionally fixes every *transitive* dependency's exact version and hash, which is what actually guarantees a reproducible build.
- **Pin exact versions in production**, and use `>=`/compatible-release (`~=`) ranges only in a library's own `pyproject.toml` if this project is ever published as an installable package rather than a deployed service.
- **Reproducible builds:** generate hash-pinned requirements for CI/CD (`pip-compile --generate-hashes` or `uv export`) and install with `pip install --require-hashes` in your Dockerfile, so a compromised package index can't silently swap in a malicious wheel.
- **Multi-stage Docker builds:** build wheels (including `torch`, `transformers`, `unstructured`'s native deps) in a `builder` stage, then copy only the installed site-packages into a slim `runtime` stage — this alone often cuts final image size by more than half given how heavy the ML dependencies are here.
- **Separate images for API vs. worker if resource profiles diverge significantly** — the Celery worker image needs the full ML/document-processing stack for ingestion, while a request-only API replica arguably doesn't if embedding is centralized in the worker; splitting `Dockerfile.api` and `Dockerfile.worker` (as scaffolded in the project's folder structure) lets you size and scale each independently.
- **Re-run dependency resolution on a schedule**, not just when something breaks — set up a monthly Dependabot/Renovate job so security patches (especially in `torch`, `transformers`, and `cryptography`-adjacent packages) don't silently accumulate.
- **Never let `langchain-community` usage grow** given its announced sunset path — audit loader usage periodically and migrate to standalone partner packages as they become available.