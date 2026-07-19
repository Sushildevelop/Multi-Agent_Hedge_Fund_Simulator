# multi-agent-hedge-fund

Project scaffold for a multi-agent hedge fund system.

# 📈 Multi-Agent Stock Investment Advisor
### *Production-Grade Multi-Agent Financial Intelligence Platform powered by Agentic RAG*

**An explainable, multi-agent financial research system that reasons like a real investment desk — not a single black-box model.**

*A multi-agent, retrieval-augmented system that turns raw financial data into explainable Buy / Hold / Sell recommendations.*


---

## Table of Contents

1. [Cover](#-multi-agent-stock-investment-advisor)
2. [Project Overview](#2-project-overview)
3. [Features](#3-features)
4. [Folder Structure](#4-folder-structure)
5. [Architecture](#5-architecture)
6. [Agent Details](#6-agent-details)
7. [RAG Pipeline](#7-rag-pipeline)
8. [Smart Search](#8-smart-search)
9. [Why Qdrant](#9-why-qdrant)
10. [Why Groq](#10-why-groq)
11. [Installation](#11-installation)
12. [Environment Variables](#12-environment-variables)
13. [API Endpoints](#13-api-endpoints)
14. [Data Flow](#14-data-flow)
15. [Challenges](#15-challenges)
16. [Smart Engineering Decisions](#16-smart-engineering-decisions)
17. [Scalability](#17-scalability)
18. [Monitoring](#18-monitoring)
19. [Security](#19-security)
20. [Testing](#20-testing)
21. [Packages](#21-packages)
22. [Future Improvements](#22-future-improvements)
23. [Resume Summary](#23-resume-summary)
24. [Interview Questions](#24-interview-questions)
25. [License](#25-license)
26. [Contribution Guide](#26-contribution-guide)
27. [Roadmap](#27-roadmap)

---

## 2. Project Overview

### The Problem

Retail and even professional investors are drowning in data — 10-Ks, earnings call transcripts, candlestick patterns, breaking news, interest rate decisions — but starved of **synthesized, explainable judgment**. Traditional "AI stock picker" tools fall into two failure modes:

- **Single-LLM chatbots** that hallucinate numbers, cannot cite sources, and collapse fundamentally different types of reasoning (accounting analysis vs. chart pattern recognition vs. macro policy reading) into one undifferentiated prompt.
- **Black-box quant models** that produce a score with no narrative, no audit trail, and no way for a human analyst to challenge the reasoning.

Neither approach reflects how real investment decisions are made. A hedge fund does not ask one generalist to read the balance sheet, chart the RSI, skim the news wire, *and* track Fed policy in a single pass — it convenes a desk of specialists and a portfolio manager who weighs their conflicting views.

### Why Agentic AI

This project treats a stock recommendation as the output of a **deliberative process**, not a single forward pass. Agentic AI — where autonomous, tool-using LLM agents each own a narrow slice of the problem — lets us:

- Decompose an ambiguous task ("should I buy AAPL?") into well-defined sub-tasks with clear inputs and outputs.
- Give each agent a focused prompt, a focused toolset, and a focused retrieval corpus, which dramatically reduces hallucination compared to one mega-prompt.
- Let a **Decision Agent** perform explicit second-order reasoning over first-order agent outputs, including modeling disagreement between agents as a signal rather than noise.

### Why Multiple Agents (Not One Bigger Prompt)

Stuffing fundamentals, technicals, news, and macro context into a single prompt does not scale: context windows fill up, unrelated signals interfere with each other during generation, and it becomes impossible to attribute *why* the model said "Buy." By splitting responsibilities across a **Fundamental Agent**, **Technical Agent**, **News Agent**, **Macro Agent**, and **Decision Agent**, each running inside a LangGraph state machine, the system gains:

- **Separation of concerns** — each agent has a narrow, testable contract.
- **Parallelism** — independent agents can execute concurrently, cutting end-to-end latency.
- **Debuggability** — a wrong recommendation can be traced back to a single agent's output rather than an opaque monolith.

### Why RAG

LLMs do not know today's earnings report, this morning's news, or a company's actual filed financials — and they should never be trusted to "remember" numbers that determine real financial decisions. Retrieval-Augmented Generation grounds every agent's reasoning in **retrieved, source-attributed documents** (10-Ks, 10-Qs, news articles, price/volume history) rather than parametric memory, which is the difference between an assistant that is *directionally interesting* and one that is *defensible*.

### Why Explainability Matters

A recommendation with no reasoning trail is not usable in any serious financial context — not for a retail investor deciding where to put their savings, and certainly not for any workflow that might face compliance review. Every recommendation this system produces includes:

- The **retrieved evidence** each agent used.
- Each agent's **independent verdict and confidence score**.
- The Decision Agent's **synthesis logic**, including where agents disagreed and how that disagreement was resolved.

This is explainability by construction, not a summary bolted on after the fact.

---

## 3. Features

| Category | Feature | Description |
|---|---|---|
| Reasoning | **Multi-Agent Reasoning** | Specialized agents collaborate via a LangGraph state machine instead of one monolithic prompt. |
| Reasoning | **Explainable AI** | Every recommendation ships with evidence, per-agent verdicts, and confidence scores. |
| Retrieval | **Financial RAG** | Retrieval pipeline grounded in filings, transcripts, and price data. |
| Retrieval | **Hybrid Retrieval** | Combines dense vector search with sparse/keyword filtering for precision. |
| Retrieval | **Semantic Search** | Embedding-based similarity search over financial documents. |
| Retrieval | **Metadata Filtering** | Filter retrieval by ticker, document type, date range, and source. |
| Analysis | **Fundamental Analysis** | Ratio analysis, revenue/earnings trend extraction from filings. |
| Analysis | **Technical Analysis** | Indicator-driven pattern recognition (RSI, MACD, moving averages). |
| Analysis | **News Analysis** | Sentiment and event extraction from live news feeds. |
| Analysis | **Macro Analysis** | Interest rate, inflation, and macro-policy contextualization. |
| Analysis | **Confidence Scoring** | Every agent emits a calibrated confidence score alongside its verdict. |
| Performance | **Streaming Responses** | Token-level streaming of the final recommendation over SSE/WebSocket. |
| Performance | **Async Processing** | Fully async FastAPI + async agent execution graph. |
| Performance | **Redis Queue** | Background job queue for long-running ingestion/analysis tasks. |
| Product | **Portfolio Tracking** | Persisted portfolio state with historical recommendation tracking. |
| Ops | **Monitoring** | Prometheus + Grafana dashboards for latency, cost, and token usage. |
| Ops | **Logging** | Structured, correlation-ID-based logging across the agent graph. |

---

## 4. Folder Structure

```
multi-agent-stock-advisor/
├── app/
│   ├── main.py                     # FastAPI application entrypoint
│   ├── api/
│   │   ├── v1/
│   │   │   ├── routes_analysis.py  # POST /analysis, streaming endpoints
│   │   │   ├── routes_portfolio.py # Portfolio CRUD endpoints
│   │   │   ├── routes_history.py   # Historical recommendation retrieval
│   │   │   └── routes_health.py    # Liveness/readiness probes
│   │   └── deps.py                 # Shared FastAPI dependencies (auth, db session)
│   │
│   ├── agents/
│   │   ├── base_agent.py           # Abstract agent interface
│   │   ├── fundamental_agent.py    # Financial statement analysis agent
│   │   ├── technical_agent.py      # Price/indicator analysis agent
│   │   ├── news_agent.py           # News sentiment/event agent
│   │   ├── macro_agent.py          # Macroeconomic context agent
│   │   └── decision_agent.py       # Synthesis + confidence aggregation agent
│   │
│   ├── graph/
│   │   ├── state.py                # LangGraph shared state schema
│   │   ├── build_graph.py          # Graph wiring, edges, conditional routing
│   │   └── nodes.py                # Node wrappers around each agent
│   │
│   ├── rag/
│   │   ├── loaders/                # PDF/HTML/API document loaders
│   │   ├── chunking.py             # Chunking strategies per document type
│   │   ├── embeddings.py           # Embedding model client wrapper
│   │   ├── retrievers.py           # Hybrid, MMR, self-query, multi-query retrievers
│   │   └── reranker.py             # Cross-encoder reranking layer
│   │
│   ├── vectorstore/
│   │   └── qdrant_client.py        # Qdrant collection management
│   │
│   ├── data_sources/
│   │   ├── yahoo_finance.py        # Price/fundamentals client
│   │   ├── alpha_vantage.py        # Secondary fundamentals client
│   │   └── news_api.py             # News ingestion client
│   │
│   ├── models/
│   │   ├── db_models.py            # SQLAlchemy ORM models
│   │   └── schemas.py              # Pydantic request/response schemas
│   │
│   ├── core/
│   │   ├── config.py                # Settings via pydantic-settings
│   │   ├── security.py              # JWT issuance/validation
│   │   ├── logging.py               # Structured logging setup
│   │   └── telemetry.py             # OpenTelemetry instrumentation
│   │
│   ├── workers/
│   │   ├── celery_app.py            # Redis-backed task queue app
│   │   └── tasks.py                 # Ingestion + analysis background tasks
│   │
│   └── db/
│       ├── session.py                # Async SQLAlchemy session factory
│       └── migrations/               # Alembic migration scripts
│
├── tests/
│   ├── unit/                        # Per-agent, per-module unit tests
│   ├── integration/                 # End-to-end graph execution tests
│   └── load/                        # Locust load-test scenarios
│
├── infra/
│   ├── docker/
│   │   ├── Dockerfile.api
│   │   └── Dockerfile.worker
│   ├── docker-compose.yml
│   └── k8s/                         # Kubernetes manifests / Helm chart
│
├── monitoring/
│   ├── prometheus.yml
│   └── grafana/dashboards/
│
├── scripts/
│   ├── seed_vectorstore.py
│   └── run_migrations.sh
│
├── .env.example
├── pyproject.toml
├── requirements.txt
└── README.md
```

Each top-level folder maps to a single architectural concern (API surface, agent logic, orchestration graph, retrieval pipeline, persistence, background work, infrastructure) so the codebase stays navigable as the number of agents and data sources grows.

---

## 5. Architecture

At a high level, a user query flows through a FastAPI ingress layer into a LangGraph-orchestrated agent graph, which pulls grounded context from Qdrant, reasons over it with Groq-hosted LLMs, and converges on a single explainable recommendation.

```
                        ┌───────────────────┐
                        │      Client        │
                        └─────────┬─────────┘
                                  │  HTTPS / JWT
                                  ▼
                        ┌───────────────────┐
                        │      FastAPI       │
                        │   (API Gateway)    │
                        └─────────┬─────────┘
                                  ▼
                        ┌───────────────────┐
                        │     LangGraph       │
                        │ (Agent Orchestrator)│
                        └─────────┬─────────┘
                 ┌────────────────┼────────────────┐
                 ▼                ▼                ▼
        ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
        │ Fundamental  │  │  Technical   │  │    News      │
        │    Agent     │  │    Agent     │  │    Agent     │
        └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
               │                 │                 │
               └────────┬────────┴────────┬────────┘
                         ▼                 ▼
                  ┌─────────────┐  ┌─────────────┐
                  │    Macro     │  │   Qdrant     │
                  │    Agent     │  │ (Vector DB)  │
                  └──────┬───────┘  └──────┬───────┘
                         │                 │
                         └────────┬────────┘
                                  ▼
                        ┌───────────────────┐
                        │    Groq LLM(s)      │
                        │ (Llama 3 / DeepSeek)│
                        └─────────┬─────────┘
                                  ▼
                        ┌───────────────────┐
                        │  Decision Agent     │
                        │ (Synthesis + Score)│
                        └─────────┬─────────┘
                                  ▼
                        ┌───────────────────┐
                        │ Final Recommendation│
                        │   (Buy/Hold/Sell)   │
                        └───────────────────┘
```

### Mermaid Architecture Diagram

```mermaid
flowchart TD
    A[Client Request] -->|HTTPS + JWT| B[FastAPI Gateway]
    B --> C[LangGraph Orchestrator]
    C --> D[Fundamental Agent]
    C --> E[Technical Agent]
    C --> F[News Agent]
    C --> G[Macro Agent]
    D --> H[(Qdrant Vector Store)]
    E --> H
    F --> H
    G --> H
    H --> I[Groq LLM Inference]
    D --> J[Decision Agent]
    E --> J
    F --> J
    G --> J
    J --> K[PostgreSQL: Persist Recommendation]
    J --> L[Final Explainable Recommendation]
    L --> B
    B --> A
```

The graph is intentionally **fan-out / fan-in**: the four analysis agents run independently (and can execute concurrently, since none depends on another's output), then converge on the Decision Agent, which is the only node that reasons over *all* agent outputs simultaneously.

---

## 6. Agent Details

Each agent is implemented as a LangGraph node wrapping a narrow, single-responsibility LLM call plus its own retrieval and tool access. None of the analysis agents can see another agent's output — that separation is what makes the Decision Agent's synthesis meaningful rather than circular.

### Fundamental Agent

| | |
|---|---|
| **Responsibilities** | Parse financial statements (income statement, balance sheet, cash flow), compute key ratios (P/E, ROE, debt-to-equity, free cash flow yield), identify multi-year trends. |
| **Input** | Ticker symbol, latest 10-K/10-Q filings, historical financial statement data. |
| **Output** | Structured verdict (Buy/Hold/Sell), ratio summary, trend narrative, confidence score (0–1). |
| **Tools** | `yahoo_finance` fundamentals client, `alpha_vantage` financials client, PDF filing parser. |
| **Prompt Strategy** | Few-shot ratio-interpretation examples + strict output schema (Pydantic-validated JSON) to prevent free-form hallucination of numbers. |
| **RAG Source** | Chunked, embedded 10-K/10-Q filings and earnings call transcripts in the `fundamentals` Qdrant collection. |

### Technical Agent

| | |
|---|---|
| **Responsibilities** | Compute and interpret technical indicators (RSI, MACD, Bollinger Bands, moving average crossovers), identify chart patterns and momentum signals. |
| **Input** | OHLCV price history (configurable lookback window, e.g. 200 trading days). |
| **Output** | Verdict, indicator readout table, momentum narrative, confidence score. |
| **Tools** | `pandas`/`ta-lib`-style indicator computation, price history client. |
| **Prompt Strategy** | Indicators are computed deterministically in code first; the LLM is only asked to *interpret* precomputed numeric indicators, not calculate them — this eliminates arithmetic hallucination entirely. |
| **RAG Source** | Not retrieval-heavy; primarily consumes structured numeric input, with optional retrieval of historical technical-analysis commentary for pattern-naming context. |

### News Agent

| | |
|---|---|
| **Responsibilities** | Retrieve recent news for the ticker/sector, extract sentiment and material events (earnings surprises, litigation, management changes, M&A). |
| **Input** | Ticker, company name, lookback window (e.g. last 7–30 days). |
| **Output** | Verdict, sentiment score, ranked list of material events with source citations, confidence score. |
| **Tools** | `news_api` client, sentiment classification prompt. |
| **Prompt Strategy** | Retrieval-then-summarize with mandatory source attribution per claim; the agent is instructed to discount low-credibility sources and flag conflicting headlines rather than averaging them away. |
| **RAG Source** | Rolling `news` Qdrant collection, re-embedded on ingestion with a short TTL so stale articles age out of retrieval relevance. |

### Macro Agent

| | |
|---|---|
| **Responsibilities** | Contextualize the stock within current interest-rate policy, inflation trends, sector-level macro exposure, and business-cycle positioning. |
| **Input** | Sector/industry classification, current macro indicator snapshot. |
| **Output** | Verdict, macro tailwind/headwind summary, confidence score. |
| **Tools** | Macro data client (rates, CPI, GDP growth), sector-exposure lookup. |
| **Prompt Strategy** | Chain-of-thought reasoning constrained to a fixed macro checklist (rates, inflation, cycle stage, sector sensitivity) to keep output consistent and comparable across tickers. |
| **RAG Source** | `macro_context` Qdrant collection containing recent central bank statements and macro research notes. |

### Decision Agent

| | |
|---|---|
| **Responsibilities** | Synthesize the four independent verdicts into one final recommendation, explicitly reasoning about agreement/disagreement and weighting confidence scores. |
| **Input** | All four agent outputs (verdict, narrative, confidence) — never raw documents directly. |
| **Output** | Final Buy/Hold/Sell recommendation, aggregate confidence score, structured explanation referencing each contributing agent. |
| **Tools** | None (pure reasoning over structured agent outputs); optional calibration function to combine confidence scores. |
| **Prompt Strategy** | Explicitly instructed to surface disagreement rather than silently averaging it away — e.g. "Fundamental Agent says Buy (0.82 confidence), Technical Agent says Sell (0.65 confidence) — reconcile and explain." |
| **RAG Source** | None directly; consumes only the structured outputs of upstream agents, which is what preserves traceability of the final answer back to specific evidence. |

---

## 7. RAG Pipeline

The retrieval pipeline is what keeps every agent grounded in real, sourced documents instead of parametric memory.

1. **Document Loading** — 10-K/10-Q filings (PDF via PyMuPDF/`unstructured`), earnings transcripts, and news articles are pulled from source APIs and normalized into a common document schema.
2. **Chunking** — Filings are chunked by semantic section (e.g. "Risk Factors," "MD&A," "Financial Statements") rather than fixed token windows, since financial documents have strong structural signal that naive fixed-size chunking destroys. News articles use smaller, sentence-aware chunks.
3. **Embedding** — Chunks are embedded with `BAAI/bge-large-en-v1.5` (or optionally `text-embedding-3-large`) and tagged with metadata (ticker, doc type, filing date, source).
4. **Vector Storage** — Embeddings are upserted into ticker- and doc-type-partitioned Qdrant collections with full payload metadata for downstream filtering.
5. **Retrieval** — Each agent issues a scoped query (hybrid dense + metadata-filtered) against only the collection(s) relevant to its responsibility.
6. **Prompt Injection** — Retrieved chunks are inserted into the agent's prompt template with explicit source markers so every generated claim can be traced back to a chunk.
7. **Generation** — The agent LLM (Groq-hosted Llama 3 / DeepSeek) generates a structured, schema-validated response grounded in the injected context.
8. **Post Processing** — Output is validated against a Pydantic schema, confidence scores are clipped to `[0, 1]`, and citations are cross-checked against the retrieved chunk IDs before the result is persisted.

```mermaid
flowchart LR
    A[Raw Documents<br/>Filings / News / Transcripts] --> B[Loaders<br/>PyMuPDF / Unstructured]
    B --> C[Chunking<br/>Section-aware]
    C --> D[Embedding<br/>bge-large-en-v1.5]
    D --> E[(Qdrant<br/>Vector Store)]
    E --> F[Retrieval<br/>Hybrid + Metadata Filter]
    F --> G[Prompt Injection]
    G --> H[Groq LLM Generation]
    H --> I[Post-Processing<br/>Schema Validation + Citation Check]
    I --> J[Agent Output]
```

---

## 8. Smart Search

Retrieval quality is the single biggest lever on hallucination rate, so the system layers several complementary retrieval strategies rather than relying on plain top-k similarity search.

- **Vector Similarity** — Baseline dense retrieval using cosine similarity over `bge-large-en-v1.5` embeddings.
- **Metadata Filtering** — Every query is scoped by ticker, document type, and date range at the Qdrant payload level *before* similarity scoring, which avoids cross-ticker contamination (e.g. never letting a Tesla 10-K influence an Apple analysis).
- **Hybrid Search** — Dense vector search combined with sparse keyword matching (BM25-style) so exact terms like ticker symbols, dollar figures, and named executives aren't lost to pure semantic drift.
- **MMR (Maximal Marginal Relevance) Retrieval** — Used when retrieving from large news corpora to reduce redundancy — five near-duplicate wire summaries of the same event are worse than three diverse ones.
- **Context Compression** — Retrieved chunks are compressed to the sentences actually relevant to the query before injection, keeping prompts small and signal-dense.
- **Self-Query Retriever** — Translates natural-language filter intent (e.g. "news from the last two weeks") into structured Qdrant metadata filters automatically.
- **Multi-Query Retriever** — Generates several paraphrased versions of the retrieval query to widen recall for ambiguous financial terminology.
- **Parent Document Retriever** — Retrieves small, precise child chunks for matching but returns their larger parent section for generation, balancing retrieval precision with generation context.
- **Compression Retriever** — A LangChain `ContextualCompressionRetriever` wraps the base retriever to strip irrelevant spans post-retrieval.
- **Cross-Encoder Reranking** — Top-k candidates from the first-pass retriever are rescored with a cross-encoder model for a more accurate final ranking before injection, since bi-encoder similarity alone is a noisy proxy for true relevance.

Together, this stack means the Fundamental Agent retrieves only the specific balance-sheet sections it needs (not the entire 10-K), and the News Agent retrieves diverse, deduplicated, recent coverage rather than five copies of the same headline.

---

## 9. Why Qdrant

| Capability | How it's used here |
|---|---|
| **Collections** | One collection per document type (`fundamentals`, `news`, `macro_context`), keeping embedding spaces and payload schemas cleanly separated. |
| **Payload** | Rich metadata (ticker, filing date, source, doc type) stored alongside vectors, enabling filter-then-search instead of search-then-filter. |
| **Filtering** | Payload filtering happens at the index level (via Qdrant's filterable HNSW), not as a post-hoc Python filter over returned results — critical for scoping retrieval per-agent without wasting recall budget. |
| **Cosine Similarity** | Default distance metric, matched to the normalized embedding space of `bge-large-en-v1.5`. |
| **HNSW Indexing** | Approximate nearest-neighbor search with tunable recall/latency tradeoff (`ef_construct`, `ef_search`) suited to sub-100ms retrieval at scale. |
| **Scalability** | Horizontal sharding and replication support growth from a handful of tickers to a full market universe without a storage-layer rewrite. |
| **Snapshots** | Point-in-time collection backups, used for reproducible RAG evaluation runs. |
| **Replication** | Multi-node replication for read availability during high query load (e.g. market-open traffic spikes). |
| **Performance** | Written in Rust with a gRPC API, giving materially lower query latency than pure-Python vector stores under concurrent load. |

**Why Qdrant over FAISS or Chroma for production:** FAISS is a library, not a service — it has no built-in filtering, no persistence layer, no replication, and no multi-tenant collection management, all of which this project needs from day one. Chroma is a reasonable prototyping choice but lacks Qdrant's production-grade filterable HNSW index and clustering story at the scale this system is designed to reach (multi-ticker, continuously ingesting news). Qdrant gives payload-filtered ANN search, horizontal scaling, and operational tooling (snapshots, replication) as first-class features rather than things to build on top.

---

## 10. Why Groq

| Factor | Why it matters here |
|---|---|
| **Low Latency** | Groq's LPU inference engine serves tokens at speeds that make a 4-agent sequential/parallel pipeline feasible within an interactive request-response budget. |
| **Open-Source LLM Support** | First-class hosting for Llama 3 and DeepSeek means the system isn't locked to a single closed-source vendor. |
| **Fast Inference** | Deterministic, hardware-accelerated inference reduces the tail latency that would otherwise dominate a multi-agent fan-out. |
| **Reasoning Quality** | Llama 3 and DeepSeek both perform well on structured financial reasoning tasks when combined with the schema-constrained prompting used here. |
| **Streaming** | Native token streaming lets the Decision Agent's final explanation stream to the client instead of blocking on full generation. |
| **Cost Effectiveness** | Open-weight models on Groq's inference pricing are materially cheaper per token than frontier closed models, which matters when four agents run per single user query. |

**Comparison with OpenAI:** OpenAI's models remain a strong optional fallback (`OPENAI_API_KEY` is supported for exactly this reason) and are sometimes preferable for tasks needing maximum reasoning depth on ambiguous macro questions. But for the bulk of this pipeline's calls — structured extraction, indicator interpretation, sentiment classification — Groq-hosted open models deliver comparable task performance at a fraction of the latency and cost, which is the deciding factor for a system that fans out to multiple LLM calls per request.

---

## 11. Installation

### Prerequisites

- Python 3.11+
- Docker & Docker Compose
- A Groq API key (and optionally an OpenAI API key)
- API keys for Alpha Vantage and NewsAPI

### Step 1 — Clone the Repository

```bash
git clone https://github.com/your-org/multi-agent-stock-advisor.git
cd multi-agent-stock-advisor
```

### Step 2 — Create a Virtual Environment

```bash
python3.11 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
```

### Step 3 — Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Step 4 — Configure Environment Variables

```bash
cp .env.example .env
# then edit .env with your API keys and connection strings
```

### Step 5 — Start Redis

```bash
docker run -d --name redis -p 6379:6379 redis:7-alpine
```

### Step 6 — Start Qdrant

```bash
docker run -d --name qdrant -p 6333:6333 -p 6334:6334 qdrant/qdrant:latest
```

### Step 7 — Run Database Migrations

```bash
alembic upgrade head
```

### Step 8 — Run the FastAPI Server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Step 9 — Run Background Workers

```bash
celery -A app.workers.celery_app worker --loglevel=info
```

### Step 10 — Run the Scheduler (Periodic Ingestion)

```bash
celery -A app.workers.celery_app beat --loglevel=info
```

### Step 11 — Or Run Everything via Docker Compose

```bash
docker compose -f infra/docker-compose.yml up --build
```

The API will be available at `http://localhost:8000`, with interactive docs at `http://localhost:8000/docs`.

---

## 12. Environment Variables

```env
# --- Application ---
APP_ENV=development
APP_HOST=0.0.0.0
APP_PORT=8000
SECRET_KEY=change-me-to-a-random-64-char-string
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60

# --- LLM Providers ---
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL_PRIMARY=llama3-70b-8192
GROQ_MODEL_SECONDARY=deepseek-r1-distill-llama-70b
OPENAI_API_KEY=your_openai_api_key_optional
USE_OPENAI_FALLBACK=false

# --- Embeddings ---
EMBEDDING_PROVIDER=bge
EMBEDDING_MODEL=BAAI/bge-large-en-v1.5
OPENAI_EMBEDDING_MODEL=text-embedding-3-large

# --- Vector Database ---
QDRANT_HOST=localhost
QDRANT_PORT=6333
QDRANT_GRPC_PORT=6334
QDRANT_API_KEY=

# --- PostgreSQL ---
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=stock_advisor
POSTGRES_USER=stock_advisor
POSTGRES_PASSWORD=change-me
DATABASE_URL=postgresql+asyncpg://stock_advisor:change-me@localhost:5432/stock_advisor

# --- Redis ---
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
CELERY_BROKER_URL=redis://localhost:6379/1
CELERY_RESULT_BACKEND=redis://localhost:6379/2

# --- Financial Data APIs ---
YAHOO_FINANCE_ENABLED=true
ALPHA_VANTAGE_API_KEY=your_alpha_vantage_key
NEWS_API_KEY=your_newsapi_key

# --- Observability ---
PROMETHEUS_ENABLED=true
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
LOG_LEVEL=INFO

# --- Rate Limiting ---
RATE_LIMIT_PER_MINUTE=60
```

---

## 13. API Endpoints

Base URL: `http://localhost:8000/api/v1`

### `POST /analysis`

Trigger a full multi-agent analysis for a ticker.

**Request**

```json
{
  "ticker": "AAPL",
  "lookback_days": 90,
  "include_macro": true
}
```

**Response**

```json
{
  "ticker": "AAPL",
  "recommendation": "Buy",
  "confidence": 0.78,
  "agents": {
    "fundamental": { "verdict": "Buy", "confidence": 0.82, "summary": "..." },
    "technical":   { "verdict": "Hold", "confidence": 0.55, "summary": "..." },
    "news":        { "verdict": "Buy", "confidence": 0.71, "summary": "..." },
    "macro":       { "verdict": "Buy", "confidence": 0.68, "summary": "..." }
  },
  "explanation": "Fundamental and news signals outweigh a neutral technical setup...",
  "generated_at": "2026-07-19T10:15:00Z"
}
```

### `POST /analysis/stream`

Same as `/analysis`, but streams the Decision Agent's explanation token-by-token over Server-Sent Events.

### `GET /portfolio`

Returns the authenticated user's tracked portfolio positions and latest recommendations.

### `POST /portfolio`

Add a ticker to the tracked portfolio.

### `GET /history`

Returns historical recommendations for a ticker, paginated.

| Query Param | Type | Description |
|---|---|---|
| `ticker` | string | Required. Ticker symbol. |
| `limit` | int | Page size, default 20. |
| `offset` | int | Pagination offset. |

### `GET /health`

Liveness/readiness probe used by Docker/Kubernetes.

```json
{ "status": "ok", "qdrant": "ok", "postgres": "ok", "redis": "ok" }
```

### `POST /auth/token`

Exchange credentials for a JWT access token.

---

## 14. Data Flow

```
User Query ("Analyze TSLA")
        │
        ▼
FastAPI  ── validates request, issues correlation ID, authenticates JWT
        │
        ▼
LangGraph ── initializes shared state, fans out to 4 analysis agents
        │
        ▼
Agents ── each independently retrieves from Qdrant and calls Groq
        │
        ▼
Qdrant ── returns ticker-scoped, reranked context chunks per agent
        │
        ▼
Groq ── generates structured, schema-validated per-agent verdicts
        │
        ▼
Decision Agent ── synthesizes verdicts, resolves disagreement, scores confidence
        │
        ▼
PostgreSQL ── persists the recommendation, agent outputs, and evidence trail
        │
        ▼
Response ── streamed/returned to the client with full explanation
```

Every hop carries the same correlation ID, which is what makes it possible to reconstruct, after the fact, exactly which document chunks and which agent reasoning produced any given recommendation.

---

## 15. Challenges

Building a production multi-agent financial system surfaces engineering problems that a single-prompt demo never encounters:

- **API Rate Limits** — Alpha Vantage, NewsAPI, and Yahoo Finance all impose per-minute/per-day caps; the ingestion layer uses token-bucket rate limiting and request batching to stay under quota during market-open spikes.
- **Hallucinations** — Mitigated primarily by forcing numeric data (ratios, indicators) to be computed in code rather than generated by the LLM, and by schema-validating every agent output.
- **Agent Disagreement** — Not a bug to eliminate but a signal to model explicitly; the Decision Agent is prompted to surface, not silently smooth over, conflicting verdicts.
- **Latency** — A four-agent fan-out plus a synthesis call is inherently slower than a single LLM call; addressed via concurrent agent execution and Groq's low-latency inference.
- **Embedding Drift** — Model or provider upgrades to the embedding model invalidate prior vectors; the system versions its collections by embedding model name to avoid silently mixing incompatible vector spaces.
- **Vector Duplication** — Repeated ingestion of the same filing/article can duplicate vectors; content-hash-based upsert keys prevent duplicate entries in Qdrant.
- **Large PDFs** — 10-K filings can exceed hundreds of pages; section-aware chunking and streaming PDF parsing avoid loading entire documents into memory at once.
- **Context Limits** — Even with chunking, aggregate retrieved context can exceed model context windows; contextual compression and reranking trim injected context to what's actually relevant.
- **Queue Failures** — Background ingestion tasks can fail mid-run; Celery task retries with exponential backoff and dead-letter queues prevent silent data gaps.
- **Network Failures** — External API outages (news, price data) are handled with circuit breakers and cached last-known-good fallbacks rather than failing the entire analysis.
- **Caching** — Redis caches recent analysis results and retrieval results to avoid redundant Groq calls for repeated queries within a short window.
- **Prompt Injection** — Retrieved news/document content is treated as untrusted input; agent prompts explicitly instruct the model to treat retrieved text as data, not instructions, and structured output parsing rejects unexpected control tokens.
- **Security** — Financial recommendation systems are a natural target for abuse; addressed via JWT auth, rate limiting, and input validation (see [Security](#19-security)).
- **Cost Optimization** — Four agent calls per query adds up; addressed via Groq's low per-token cost, response caching, and routing simpler sub-tasks (e.g. indicator interpretation) to smaller models.

---

## 16. Smart Engineering Decisions

| Decision | Why |
|---|---|
| **LangGraph over a plain LangChain chain** | Explicit state machine semantics (nodes, edges, conditional routing) make the fan-out/fan-in agent topology inspectable and testable in a way a linear chain cannot express. |
| **Qdrant over FAISS/Chroma** | Production-grade filtering, replication, and snapshotting without building that operational layer in-house (see [Why Qdrant](#9-why-qdrant)). |
| **Groq over a single OpenAI-only setup** | Latency and cost profile suited to a multi-call-per-request architecture, with OpenAI kept available as an optional fallback. |
| **Redis for both queueing and caching** | One operational dependency instead of two; Redis's data structures cleanly serve both Celery's broker needs and simple TTL-based result caching. |
| **PostgreSQL for persistence** | Recommendations, portfolios, and audit trails are relational and benefit from PostgreSQL's transactional guarantees and JSONB columns for flexible agent-output storage. |
| **FastAPI over Flask/Django** | Native async support, Pydantic-based request/response validation, and automatic OpenAPI docs match a service that is fundamentally I/O-bound (LLM calls, vector search, external APIs). |
| **Docker/Docker Compose for local dev, Kubernetes for prod** | Compose keeps local onboarding to a single command; Kubernetes manifests give the same topology horizontal scalability and self-healing in production. |

---

## 17. Scalability

- **Horizontal Scaling** — The FastAPI service is stateless; additional replicas can be added behind a load balancer with no session affinity required.
- **Load Balancing** — A Kubernetes Service/Ingress (or an external ALB) distributes traffic across API pod replicas.
- **Redis Queues** — Long-running ingestion and re-embedding jobs are offloaded to Celery workers so API request latency stays decoupled from background workload.
- **Caching** — Redis-backed response caching absorbs repeated-query load without re-invoking the full agent graph.
- **Kubernetes** — Deployment manifests define separate scalable pools for the API, Celery workers, and the scheduler, each with independent HPA (Horizontal Pod Autoscaler) targets.
- **Async Processing** — End-to-end async I/O (FastAPI, async SQLAlchemy, async Qdrant client) maximizes concurrency per pod without needing a thread per request.
- **Worker Nodes** — Celery worker pools scale independently of the API tier based on ingestion queue depth.
- **Vector DB Scaling** — Qdrant supports sharded collections and read replicas as the document corpus and query volume grow.
- **Database Sharding** — PostgreSQL can be partitioned by ticker or by time range for very large recommendation-history datasets.
- **Connection Pooling** — `asyncpg`/SQLAlchemy connection pools and a Qdrant gRPC channel pool prevent connection exhaustion under load.
- **CDN** — Static assets (docs UI, dashboards) are served through a CDN to keep the API tier focused on dynamic traffic.
- **Rate Limiting** — Per-user and per-IP request limits protect both the API and downstream paid LLM/data APIs from abuse-driven cost spikes.
- **Circuit Breakers** — Wrapping calls to external financial APIs and the LLM provider so a downstream outage degrades gracefully instead of cascading into full request failures.

---

## 18. Monitoring

- **Prometheus** — Scrapes custom metrics (request latency, per-agent execution time, token usage, cache hit rate) exposed via a `/metrics` endpoint.
- **Grafana** — Pre-built dashboards visualize latency percentiles, error rates, queue depth, and per-agent confidence score distributions over time.
- **Logging** — Structured JSON logs, correlated by request ID across the API, LangGraph nodes, and Celery workers, shipped to a centralized log store.
- **OpenTelemetry** — Distributed tracing spans wrap each agent's execution and each external call (Qdrant query, Groq call, financial API call), making it possible to see exactly where time is spent within a single request.
- **Tracing** — End-to-end traces let engineers pinpoint whether latency in a slow request came from retrieval, LLM inference, or an external data API.
- **Metrics** — Business metrics (recommendations issued per ticker, agent agreement rate) are tracked alongside infrastructure metrics.
- **Alerting** — Alertmanager rules fire on elevated error rates, queue backlog growth, or anomalous token-usage spikes (a common signal of prompt-injection attempts or runaway loops).
- **Token Usage** — Per-agent, per-request token accounting feeds directly into cost dashboards and per-user rate limiting.
- **Latency** — P50/P95/P99 latency tracked per endpoint and per agent, since the four-agent fan-out makes tail latency the primary UX risk.

---

## 19. Security

- **JWT** — Stateless bearer-token authentication with short-lived access tokens and configurable expiry.
- **Rate Limiting** — Per-user and per-IP limits enforced at the API gateway layer to prevent abuse and control downstream API/LLM cost.
- **Secrets Manager** — Production deployments load API keys and database credentials from a secrets manager (e.g. AWS Secrets Manager/Vault) rather than plaintext environment files.
- **HTTPS** — All external traffic is terminated over TLS; internal service-to-service traffic runs inside a private network.
- **Prompt Injection Protection** — Retrieved document content is explicitly marked as untrusted data in agent prompts, and structured output parsing rejects any response that doesn't conform to the expected schema.
- **API Validation** — Every request/response is validated against Pydantic schemas, rejecting malformed or unexpected payloads before they reach business logic.
- **Role-Based Access** — Portfolio and history endpoints are scoped to the authenticated user; admin-only endpoints (e.g. manual re-ingestion triggers) require an elevated role claim.
- **SQL Injection** — All database access goes through parameterized SQLAlchemy queries; no raw string-interpolated SQL.
- **XSS** — API is a pure JSON service (no server-rendered HTML with user input), removing the primary XSS vector; any admin dashboard sanitizes rendered output.
- **CORS** — Explicit allow-list of origins configured per environment rather than a wildcard.

---

## 20. Testing

- **Unit Testing** — Each agent, retriever, and utility function has isolated `pytest` unit tests with mocked LLM and Qdrant responses.
- **Integration Testing** — End-to-end graph execution tests run the full LangGraph pipeline against a test Qdrant instance and a stubbed Groq client to verify node wiring and state propagation.
- **Load Testing** — `Locust` scenarios simulate concurrent `/analysis` requests to measure throughput and latency degradation under load.
- **Stress Testing** — Sustained load beyond expected peak traffic to identify the system's breaking point and validate circuit-breaker behavior.
- **Agent Testing** — Golden-output regression tests for each agent, checking that verdicts and confidence scores stay within expected bounds for known input fixtures.
- **Prompt Testing** — Versioned prompt templates are tested against a fixed evaluation set whenever a prompt changes, to catch regressions before deployment.
- **Evaluation** — RAG-specific evaluation (retrieval precision/recall, faithfulness of generated claims to retrieved context) run against a curated evaluation dataset of tickers with known-good analyst outputs.

---

## 21. Packages

| Package | Role | Why it's used |
|---|---|---|
| **FastAPI** | Web framework | Async-native, Pydantic-integrated, auto-generates OpenAPI docs — ideal for an I/O-bound multi-agent service. |
| **LangGraph** | Agent orchestration | Explicit graph-based state machine for coordinating fan-out/fan-in agent execution with conditional routing. |
| **LangChain** | RAG utilities | Provides retriever abstractions (MMR, self-query, multi-query, compression) used across the retrieval layer. |
| **Qdrant Client** | Vector DB access | gRPC/REST client for collection management, upserts, and filtered similarity search. |
| **Groq SDK** | LLM inference | Low-latency access to hosted Llama 3 / DeepSeek models. |
| **Redis (redis-py / Celery)** | Queueing & caching | Backs both the Celery task broker and short-TTL response caching. |
| **SQLAlchemy (async)** | ORM | Async database access with connection pooling for PostgreSQL. |
| **Alembic** | Migrations | Versioned schema migrations for PostgreSQL. |
| **PyMuPDF** | PDF parsing | Fast, reliable text/layout extraction from large 10-K/10-Q filings. |
| **Unstructured** | Document parsing | Handles heterogeneous document formats and layout-aware chunking. |
| **pypdf** | PDF fallback parsing | Lightweight fallback parser for simpler PDF documents. |
| **Pydantic** | Schema validation | Enforces strict, typed contracts on every API request/response and every agent output. |
| **Celery** | Task queue | Executes background ingestion, embedding, and scheduled re-analysis jobs. |
| **Pytest** | Testing | Unit and integration test framework across the codebase. |
| **Locust** | Load testing | Simulates concurrent user traffic against the API for performance testing. |
| **Prometheus client** | Metrics | Exposes application metrics for scraping. |
| **OpenTelemetry SDK** | Tracing | Instruments requests and agent calls for distributed tracing. |
| **python-jose / PyJWT** | Auth | JWT issuance and verification. |
| **Docker / Docker Compose** | Containerization | Reproducible local and CI environments; single-command startup. |

---

## 22. Future Improvements

- **Real-Time Streaming** — Push live token-level agent reasoning to the client as each agent completes, not just the final Decision Agent output.
- **Portfolio Optimization** — Extend beyond single-ticker recommendations to portfolio-level allocation suggestions (mean-variance or risk-parity informed).
- **Auto Retraining / Re-Evaluation** — Scheduled re-evaluation of agent prompt performance against a growing golden dataset, with automatic rollback on regression.
- **Long-Term Memory** — Persistent per-user memory of past recommendations and outcomes to personalize future analysis tone and risk framing.
- **MCP (Model Context Protocol) Integration** — Expose the agent toolset via MCP so external agent frameworks can call into this system's specialized financial agents.
- **Voice Assistant Interface** — Voice-driven query and streamed spoken explanation of recommendations.
- **Multi-Modal Support** — Chart-image understanding (candlestick pattern recognition directly from rendered charts) alongside numeric indicator interpretation.
- **Real-Time Stock WebSocket** — Live price streaming to trigger re-analysis on significant intraday moves rather than only on-demand.
- **Knowledge Graph** — Entity-relationship graph linking companies, executives, sectors, and macro factors to support relationship-aware retrieval.
- **Graph RAG** — Combine the knowledge graph with vector retrieval so agents can reason over explicit entity relationships, not just semantic similarity.

---

## 23. Resume Summary

- Designed and built a production-grade multi-agent Agentic RAG platform in FastAPI + LangGraph, orchestrating five specialized LLM agents (fundamental, technical, news, macro, decision) to generate explainable stock recommendations.
- Engineered a hybrid retrieval pipeline (dense + sparse + MMR + cross-encoder reranking) over Qdrant, improving retrieval precision on financial filings and news corpora versus naive top-k vector search.
- Implemented an async, horizontally scalable architecture using Redis-backed Celery queues, PostgreSQL, and Kubernetes, supporting concurrent multi-agent execution under production load.
- Reduced LLM hallucination risk by constraining numeric computation (financial ratios, technical indicators) to deterministic code paths, reserving LLM calls strictly for interpretation and synthesis.
- Instrumented the full request lifecycle with Prometheus, Grafana, and OpenTelemetry, enabling per-agent latency, token-cost, and confidence-score observability across the pipeline.

---

## 24. Interview Questions

**System Design & Architecture**

1. **Why did you choose a multi-agent architecture instead of a single LLM prompt for stock analysis?**
   A single prompt conflates fundamentally different reasoning types (accounting, charting, sentiment, macro) into one context window, which increases hallucination and makes it impossible to attribute why a conclusion was reached. Separate agents give each task a focused prompt, focused retrieval scope, and independently testable/auditable output.

2. **How does LangGraph differ from a standard LangChain sequential chain?**
   LangGraph models the workflow as an explicit graph with nodes and (conditional) edges over a shared state object, supporting branching, fan-out/fan-in, and cycles — a linear chain can only express a fixed sequence of steps.

3. **Why is the agent graph fan-out / fan-in rather than sequential?**
   The four analysis agents have no data dependency on each other, so running them concurrently minimizes end-to-end latency; only the Decision Agent needs all four outputs simultaneously.

4. **How would you scale this system to handle 10,000 concurrent analysis requests?**
   Horizontally scale stateless FastAPI pods behind a load balancer, scale Celery worker pools independently based on queue depth, shard/replicate Qdrant collections, add read replicas for PostgreSQL, and apply aggressive response caching for repeated ticker queries within a short window.

5. **What happens if the News API is down during a request?**
   A circuit breaker trips after repeated failures, the News Agent falls back to the most recent cached articles (with a staleness flag surfaced in the output) rather than blocking or failing the entire analysis.

**RAG & Retrieval**

6. **Why chunk 10-K filings by section instead of fixed token windows?**
   Financial filings have strong structural signal (Risk Factors, MD&A, Financial Statements); fixed-size chunking can split a single financial table or argument across chunks, destroying retrieval coherence. Section-aware chunking preserves semantic units.

7. **What's the difference between MMR retrieval and plain top-k similarity search?**
   Top-k similarity can return near-duplicate chunks that all say the same thing; MMR explicitly balances relevance against diversity so the retrieved set covers more distinct information.

8. **Why use a cross-encoder reranker on top of vector search?**
   Bi-encoder similarity (the basis of vector search) is a fast but noisy relevance proxy; a cross-encoder jointly encodes the query and candidate, producing a materially more accurate relevance ranking for the smaller top-k candidate set.

9. **What is a self-query retriever and why is it useful here?**
   It translates natural-language filter intent (e.g. "recent news only") into structured metadata filters automatically, rather than requiring the calling agent to manually construct Qdrant filter syntax.

10. **How do you prevent one ticker's documents from leaking into another ticker's analysis?**
    Metadata filtering (ticker field) is applied at the Qdrant payload-index level before similarity scoring, not as a post-hoc filter over returned results, guaranteeing scoped retrieval.

11. **Why embed with BAAI/bge-large-en-v1.5 instead of a general-purpose embedding model?**
    `bge-large-en-v1.5` performs strongly on retrieval benchmarks for dense passage retrieval and is open-weight, avoiding per-call embedding API cost and vendor lock-in, while `text-embedding-3-large` remains available as an optional higher-fidelity alternative.

12. **How does the parent document retriever differ from standard chunk retrieval?**
    It matches against small, precise child chunks for retrieval accuracy but returns the larger parent section to the generation step, balancing precise matching with sufficient context for the LLM.

13. **What is context compression and why apply it after retrieval?**
    It strips retrieved chunks down to only the sentences relevant to the query, reducing prompt size and noise before generation — important when injecting multiple agents' retrieved context into token-limited prompts.

14. **How would you detect embedding drift after upgrading the embedding model?**
    Version Qdrant collections by embedding model name; mixing vectors from two different embedding spaces in one collection silently corrupts similarity scores, so a model upgrade requires a full re-embedding into a new versioned collection.

15. **How do you evaluate RAG quality beyond "it looks right"?**
    Track retrieval precision/recall against a labeled evaluation set, and faithfulness — whether generated claims are actually supported by the retrieved context — rather than only checking final-answer correctness.

**Agents & Prompting**

16. **Why is the Technical Agent not allowed to compute indicators itself?**
    LLMs are unreliable at exact arithmetic; indicators (RSI, MACD, moving averages) are computed deterministically in code, and the LLM's role is constrained to interpreting precomputed numbers, eliminating a whole class of numeric hallucination.

17. **How does the Decision Agent handle disagreement between agents?**
    It's explicitly prompted to surface disagreement (e.g. Fundamental says Buy, Technical says Sell) and reason about it in the final explanation, rather than silently averaging conflicting verdicts into an uninformative middle ground.

18. **Why does each agent output a confidence score, and how is it used?**
    Confidence scores let the Decision Agent weight noisier or less certain agent verdicts appropriately during synthesis, and give end users a calibrated sense of how strong the evidence behind a recommendation actually was.

19. **How do you prevent prompt injection from a malicious news article?**
    Retrieved content is explicitly framed as untrusted data within the prompt template (not as instructions), and agent outputs are strictly schema-validated, so injected text that tries to alter output format is rejected at the parsing layer.

20. **Why use structured/schema-validated LLM outputs instead of free-form text?**
    Free-form output is brittle to parse and easy to silently corrupt; Pydantic-validated structured output makes every downstream step (persistence, aggregation, API response) deterministic and catches malformed generations immediately.

**Data & Infrastructure**

21. **Why PostgreSQL instead of a NoSQL store for recommendation history?**
    Recommendations, users, and portfolios are inherently relational with transactional requirements (e.g. atomic portfolio updates); PostgreSQL's JSONB columns still allow flexible storage of variable agent-output structures where needed.

22. **Why use Redis for both queueing and caching instead of separate systems?**
    Reduces operational surface area — one dependency serves Celery's broker/result-backend needs and simple TTL-based response caching, rather than running e.g. RabbitMQ plus a separate cache layer.

23. **How do you keep background ingestion from falling behind during market hours?**
    Celery worker pools scale independently of the API tier based on queue depth (via Kubernetes HPA), and rate-limited API clients queue rather than drop requests when external API limits are hit.

24. **What's the purpose of correlation IDs across the request lifecycle?**
    They let engineers reconstruct, after the fact, the full trace of a single recommendation — which document chunks were retrieved, which agent produced which verdict — across FastAPI, LangGraph, and Celery logs.

25. **How would you handle a Qdrant node failure in production?**
    Qdrant's replication ensures reads can be served from a surviving replica; the client is configured with retry/failover logic, and periodic snapshots provide a recovery point if a full cluster restore is needed.

**Security & Reliability**

26. **How is JWT used here, and what are its tradeoffs versus session-based auth?**
    JWTs are stateless bearer tokens, avoiding server-side session storage and simplifying horizontal scaling, at the cost of harder immediate revocation — mitigated here with short expiry windows.

27. **How do you prevent the system from being used to leak another user's portfolio data?**
    Every portfolio/history query is scoped server-side to the authenticated user's ID extracted from the validated JWT claim, never accepted as a client-supplied parameter.

28. **What's a circuit breaker, and where is one used in this system?**
    A pattern that stops calling a failing downstream dependency after a failure threshold, failing fast instead of piling up timeouts; used here around external financial/news APIs and the LLM provider.

29. **How would you control runaway LLM cost from a single misbehaving client?**
    Per-user and per-IP rate limiting at the gateway, combined with per-request token-usage tracking that feeds alerting on anomalous spikes.

30. **How do you test a non-deterministic system like an LLM agent pipeline?**
    Golden-output regression tests check verdicts/confidence stay within expected bounds (not exact string match) for fixed input fixtures, combined with RAG-specific faithfulness evaluation and integration tests that stub the LLM to verify graph wiring independent of model output variance.

---

## 25. License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

```
MIT License

Copyright (c) 2026 Multi-Agent Stock Advisor Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

## 26. Contribution Guide

Contributions are welcome — this project follows a standard fork-and-PR workflow.

### Getting Started

1. Fork the repository and clone your fork locally.
2. Create a feature branch: `git checkout -b feature/your-feature-name`.
3. Set up the local environment as described in [Installation](#11-installation).
4. Make your changes, following the code style enforced by `ruff`/`black` (`pre-commit run --all-files` before committing).

### Commit Conventions

Use [Conventional Commits](https://www.conventionalcommits.org/) style messages, e.g.:

```
feat(agents): add sector-relative valuation to fundamental agent
fix(rag): correct off-by-one in chunk overlap calculation
docs(readme): clarify environment variable defaults
```

### Pull Request Checklist

- [ ] All new code has unit test coverage.
- [ ] `pytest` passes locally, including integration tests.
- [ ] Any new/changed agent prompt includes an update to its evaluation fixture.
- [ ] Documentation (README, docstrings) updated for any user-facing change.
- [ ] No secrets or API keys committed — check `.env` is git-ignored.

### Code of Conduct

Be respectful, assume good intent, and keep discussion focused on the technical merits of a change. Disagreements about architecture or approach are welcome in PR discussion — just as this project's Decision Agent is designed to surface disagreement rather than hide it, so should code review.

### Reporting Issues

Please include: reproduction steps, expected vs. actual behavior, relevant logs (with secrets redacted), and environment details (Python version, OS, Docker vs. local).

---

## 27. Roadmap

### Version 1.0 — Foundation (Current)

- Core five-agent LangGraph pipeline (Fundamental, Technical, News, Macro, Decision)
- Qdrant-backed RAG over filings and news
- FastAPI service with JWT auth, portfolio tracking, and recommendation history
- Docker Compose local development environment
- Prometheus/Grafana monitoring baseline

### Version 2.0 — Scale & Depth

- Kubernetes production deployment with autoscaling
- Cross-encoder reranking and multi-query retrieval fully in production path
- Portfolio-level (multi-ticker) optimization recommendations
- Expanded evaluation harness with continuous prompt regression testing

### Version 3.0 — Intelligence & Reach

- Knowledge Graph + Graph RAG for entity-relationship-aware reasoning
- Real-time WebSocket price feeds triggering event-driven re-analysis
- MCP-based exposure of individual agents as composable tools for external systems
- Multi-modal chart pattern recognition alongside numeric indicators

### Beyond

- Voice-driven interaction
- Personalized long-term memory of user risk preferences and past outcomes
- Community-contributed agent plugins (e.g. ESG Agent, Options-Flow Agent)

---


**Built as a demonstration of production-grade Agentic RAG system design.**

*Not financial advice. For educational and portfolio-demonstration purposes.*


---

## Appendix A: Troubleshooting Guide

**`qdrant.http.exceptions.UnexpectedResponse: 404`**
Usually means the target collection hasn't been created yet. Run the seed script before starting the API:
```bash
python scripts/seed_vectorstore.py
```

**Agents return `Hold` with very low confidence for every ticker**
Almost always a retrieval problem, not a model problem — check that ingestion actually populated the relevant Qdrant collection for that ticker (`GET /collections/{name}` on the Qdrant REST API) before assuming the LLM prompt is at fault.

**`GROQ_API_KEY` errors on startup but the key is set in `.env`**
Confirm the `.env` file is actually being loaded — Docker Compose requires the `env_file` directive per service; a `.env` in the repo root is not automatically picked up by containers unless explicitly wired in `docker-compose.yml`.

**Celery workers start but never process ingestion tasks**
Check that `CELERY_BROKER_URL` points at the same Redis instance/DB index the API is publishing tasks to — a common misconfiguration is pointing the worker at `redis://localhost:6379/0` while the API publishes to `/1`.

**High P99 latency only on `/analysis`, not other endpoints**
Check OpenTelemetry traces first — in most cases this is either the News Agent blocking on a slow upstream NewsAPI response (add/lower the circuit-breaker timeout) or the Decision Agent's synthesis call being routed to a larger, slower model than intended.

**Duplicate near-identical chunks show up in retrieved context**
Confirm content-hash-based upsert keys are actually being used during ingestion — re-running ingestion without idempotent keys is the most common cause of vector duplication in Qdrant.

**JWT validation fails immediately after deploying a new environment**
`SECRET_KEY` must be identical across all API replicas; if it's generated randomly at container startup instead of being read from a shared secret, tokens issued by one replica will fail validation on another.

---

## Appendix B: Frequently Asked Questions

**Is this system giving real financial advice?**
No. This is an educational/portfolio-demonstration project showcasing production-grade Agentic RAG system design. Nothing it outputs should be treated as licensed financial advice.

**Can I swap Groq for a different inference provider?**
Yes — the LLM client is abstracted behind a thin interface in `app/agents/base_agent.py`; adding a new provider means implementing that interface and wiring it into `core/config.py`.

**Why five agents and not, say, three or seven?**
Five was chosen as the smallest decomposition that cleanly separates the major categories of investment reasoning (fundamentals, technicals, news, macro, synthesis) without over-fragmenting into agents so narrow they'd have nothing meaningful to disagree about.

**Does the system retrain the LLMs on financial data?**
No — it uses off-the-shelf hosted models (Llama 3, DeepSeek, optionally GPT-family models) grounded via RAG rather than fine-tuning, which keeps the system provider-agnostic and avoids the operational burden of managing custom model weights.

**How current is the data behind a recommendation?**
Filings are as current as the last scheduled ingestion run; news is refreshed on a short interval (configurable, default hourly) via the Celery beat scheduler. The `generated_at` timestamp on every response reflects when the analysis itself was run, not when underlying documents were last ingested — both are logged for full traceability.

**Can this run entirely offline / self-hosted?**
Vector storage (Qdrant), the database (PostgreSQL), and the queue (Redis) can all run fully self-hosted. LLM inference currently depends on Groq's hosted API; a fully offline setup would require swapping in a locally hosted open-weight model server behind the same agent interface.

**What happens if two agents return the exact same confidence score but opposite verdicts?**
The Decision Agent doesn't just average — it's prompted to reason qualitatively about *why* two equally confident agents disagree (e.g. strong fundamentals vs. deteriorating short-term technical momentum) and reflects that tension explicitly in the final explanation rather than resolving it arithmetically.

---

## Appendix C: Best Practices for Extending This Project

- **New agents should own a genuinely independent signal.** Before adding an agent, confirm it retrieves from a distinct data source and can meaningfully disagree with existing agents — an agent that just re-derives another agent's conclusion adds latency without adding information.
- **Never let an LLM compute a number that code can compute deterministically.** This is the single highest-leverage hallucination-prevention rule in the entire system.
- **Version everything that touches the embedding space.** Collection names, embedding model versions, and chunking strategy versions should all be encoded in Qdrant collection naming to avoid silent vector-space corruption.
- **Treat every prompt change as a code change.** Prompt edits should go through the same PR review and regression-evaluation process as application code, not be edited ad hoc in production.
- **Keep agent contracts schema-first.** Define the Pydantic output schema before writing the prompt — this constrains prompt design toward outputs that are actually parseable and preventing free-form drift over time.
- **Log evidence, not just conclusions.** Every persisted recommendation should retain enough of the retrieval trail to answer "why did it say this?" months later, not just the final verdict.




