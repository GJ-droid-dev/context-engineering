# Hindsight

> Agent memory that learns. Hindsight uses biomimetic data structures — world facts, experiences, and mental models — to give agents memory that improves over time, not just history that accumulates.

**Source:** [vectorize-io/hindsight](https://github.com/vectorize-io/hindsight) — 5.1k stars  
**Layer:** 1 — Foundational Memory  
**WISC:** W (write, retrieve, and synthesize across agent lifecycle)

---

## The Problem It Solves

Most agent memory systems are retrieval pipes: store conversations, retrieve relevant chunks, inject into context. This solves recall but not learning. An agent that recalls facts is still stateless in a meaningful sense — it cannot form higher-order understanding from what it has seen.

Hindsight addresses three limitations of standard RAG-based memory:

- **Recall without synthesis:** The agent retrieves what it was told, but never derives new understanding from it
- **Flat fact storage:** No distinction between "a user told me X" (raw input) and "I now understand X is true in this domain" (mental model)
- **Temporal blindness:** Standard vector search treats a 2-year-old fact identically to a fact from this session

---

## Architecture: Biomimetic Memory Organization

Hindsight stores memories in three distinct pathways, mirroring human memory architecture:

```
┌─────────────────────────────────────────────┐
│  MENTAL MODELS                              │
│  Synthesized understanding; formed by       │
│  reflect() operations on raw memories       │
├─────────────────────────────────────────────┤
│  EXPERIENCES                                │
│  Agent's own interactions and observations  │
│  Scope: what happened to this agent         │
├─────────────────────────────────────────────┤
│  WORLD FACTS                                │
│  Verified facts about the world/domain      │
│  Scope: stable truths, not session-bound    │
└─────────────────────────────────────────────┘
```

When content is pushed via `retain()`, Hindsight uses an LLM to extract entities, relationships, and temporal data, then routes the result to the appropriate pathway. The result is a structured memory graph rather than a flat vector index.

---

## Three Core Operations

### retain — Store information

```python
from hindsight_client import Hindsight

client = Hindsight(base_url="http://localhost:8888")

# Store a fact
client.retain(bank_id="project", content="The payment API uses idempotency keys on all POST requests")

# Store with context and timestamp
client.retain(
    bank_id="project",
    content="Team decided to move from REST to GraphQL for the client API",
    context="architecture decision",
    timestamp="2026-03-01T10:00:00Z"
)
```

Behind the scenes, `retain` extracts entities, relationships, and temporal markers, then normalizes and indexes them across semantic, keyword, and graph representations.

### recall — Retrieve memories

```python
# recall runs 4 retrieval strategies in parallel:
# 1. Semantic  — vector similarity
# 2. Keyword   — BM25 exact matching
# 3. Graph     — entity/temporal/causal links
# 4. Temporal  — time range filtering

results = client.recall(bank_id="project", query="What API conventions do we use?")

# With time filter
results = client.recall(bank_id="project", query="recent architecture decisions", after="2026-01-01")
```

The four results are merged via reciprocal rank fusion, then reranked by a cross-encoder model, then trimmed to fit the token budget. This consistently outperforms single-strategy retrieval.

### reflect — Synthesize understanding

```python
# reflect does not retrieve — it generates new insights from existing memories
client.reflect(bank_id="project", query="What are the key patterns in how this team makes decisions?")

# Use cases:
# - "What risks should I flag for this project?"
# - "What approach has worked well in this codebase before?"
# - "Why have certain outreach messages gotten responses?"
```

`reflect` is the mechanism by which the agent forms mental models. It's distinct from recall: recall retrieves what exists; reflect synthesizes what can be derived. See [Reflective Memory](../../patterns/reflective-memory.md) for when and how to use it.

---

## Performance

Hindsight holds state-of-the-art performance on the **LongMemEval benchmark** (widely used to assess agent memory across conversational AI scenarios) as of January 2026. Results were independently reproduced by Virginia Tech's Sanghani Center for AI and Data Analytics and The Washington Post.

---

## Setup

```bash
# Server (Docker, recommended)
export OPENAI_API_KEY=sk-xxx
docker run --rm -it --pull always -p 8888:8888 -p 9999:9999 \
  -e HINDSIGHT_API_LLM_API_KEY=$OPENAI_API_KEY \
  -v $HOME/.hindsight-docker:/home/hindsight/.pg0 \
  ghcr.io/vectorize-io/hindsight:latest

# Client
pip install hindsight-client
# or
npm install @vectorize-io/hindsight-client
```

**Supported LLM providers:** OpenAI, Anthropic, Gemini, Groq, Ollama, LMStudio, MiniMax

Also available embedded (no server required):

```python
pip install hindsight-all

from hindsight import HindsightServer, HindsightClient

with HindsightServer(llm_provider="openai", llm_model="gpt-4o-mini", llm_api_key=...) as server:
    client = HindsightClient(base_url=server.url)
    client.retain(bank_id="my-bank", content="...")
```

---

## Hindsight vs. Other Layer 1 Memory Tools

| Dimension | Hindsight | mem0 | Basic Memory | ConPort |
|-----------|-----------|------|--------------|---------|
| Target | Production agents | Production apps | Developer sessions | Dev project knowledge |
| Memory types | World / Experiences / Mental Models | User / Session / Agent | Markdown notes + relations | Decisions / Progress / Patterns |
| Recall strategies | Semantic + keyword + graph + temporal | Vector + semantic extraction | Hybrid BM25 + vector | SQLite queries |
| Synthesis (reflect) | ✅ Native | ❌ | ❌ | ❌ |
| Deployment | Docker / embedded Python | SDK / managed cloud | MCP server | MCP server |
| Best for | Agents that need to learn, not just remember | User-personalized production apps | Developer knowledge base | Explicit project knowledge logging |

---

## In This Repo's Architecture

Hindsight sits in **Layer 1 — Foundational Memory** alongside mem0. The key distinction: mem0 provides the standard three-tier memory model for production apps; Hindsight adds the `reflect` operation that allows agents to accumulate *understanding* rather than just *facts*.

The `reflect` operation is documented as a standalone pattern in [Reflective Memory](../../patterns/reflective-memory.md).

**Related patterns:** [File Memory Structure](../../patterns/file-memory-structure.md), [Reflective Memory](../../patterns/reflective-memory.md)
