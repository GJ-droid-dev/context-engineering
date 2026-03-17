# mem0

> Application-level memory infrastructure for production AI agents. Three-tier memory architecture (User / Session / Agent) backed by academic research showing +26% accuracy improvement on the LOCOMO benchmark.

**Source:** [mem0ai/mem0](https://github.com/mem0ai/mem0) — 50.2k stars  
**Layer:** 1 — Foundational Memory  
**WISC:** W (write and retrieve across agent lifecycle)

---

## The Problem It Solves

Production AI agents need to remember users across API calls. "What's this user's timezone preference?" "What approach did this user's agent use last sprint?" "What did this customer say about payment preferences last month?" Stateless LLM calls can't answer these questions. mem0 can.

mem0 operates at the **application layer** — it's the memory infrastructure you embed in your AI product, not in your editor. It's for agents you deploy, not agents you converse with in VS Code.

---

## Architecture: Three Memory Tiers

```
┌─────────────────────────────────────────────┐
│  USER MEMORY                                │
│  Persistent: preferences, history, facts    │
│  Scope: per user, permanent until cleared   │
├─────────────────────────────────────────────┤
│  SESSION MEMORY                             │
│  Transient: current conversation context   │
│  Scope: single session lifecycle            │
├─────────────────────────────────────────────┤
│  AGENT MEMORY                               │
│  Procedural: learned behaviors, patterns    │
│  Scope: per agent instance                  │
└─────────────────────────────────────────────┘
```

---

## Research Backing

mem0's paper (published with YC S24 batch) ran the **LOCOMO benchmark** — a conversational memory benchmark testing whether agents can answer questions about previous conversations. Result: mem0 agents scored **+26% higher** than agents without memory.

The key technique: mem0 doesn't store raw conversation history (expensive, noisy). It extracts and stores semantic *facts* from conversations, then retrieves the most relevant facts at inference time.

---

## Setup

```bash
npm install mem0ai
# or
pip install mem0ai
```

**TypeScript/Node.js:**
```typescript
import { Memory } from 'mem0ai';

const memory = new Memory({
  llm: { provider: 'openai', config: { model: 'gpt-4o-mini' } },
  vector_store: { provider: 'qdrant', config: { host: 'localhost', port: 6333 } }
});

// Store a memory
await memory.add(
  [{ role: 'user', content: 'I prefer TypeScript over JavaScript' }],
  { user_id: 'user-123' }
);

// Retrieve relevant memories
const memories = await memory.search('coding preferences', { user_id: 'user-123' });
```

**Managed cloud (no infrastructure setup):**
```typescript
import { MemoryClient } from 'mem0ai';
const client = new MemoryClient({ api_key: process.env.MEM0_API_KEY });
```

---

## Key Operations

```typescript
// Add memory from conversation
await memory.add(messages, { user_id, session_id, agent_id });

// Search for relevant memories
const relevant = await memory.search(query, { user_id, limit: 5 });

// Get all memories for a user
const all = await memory.getAll({ user_id });

// Delete a specific memory
await memory.delete(memory_id);

// Update a memory
await memory.update(memory_id, { data: 'Updated fact' });
```

---

## Storage Options

mem0 supports multiple vector stores and LLMs:

**Vector stores:** Qdrant, Pinecone, Chroma, Weaviate, PostgreSQL/pgvector, FAISS  
**LLMs for extraction:** OpenAI, Anthropic, Google, Ollama (local), Groq  
**Graph memory:** Neo4j (for relation-aware memory)

---

## mem0 vs. Developer-Tooling Memory (ConPort, ICM, Basic Memory)

| Dimension | mem0 | ConPort / ICM / Basic Memory |
|-----------|------|------------------------------|
| Who uses it | Your deployed AI product | You, the developer in VS Code |
| Memory owner | End users of your app | The developer and their project |
| Scale | Millions of users | One developer, one project |
| Deployment | Production server | Local dev environment |
| Integration | SDK call in your agent code | MCP server in your editor |

These tools solve different problems at different layers. A project could use mem0 in production for user memory *and* ConPort in development for project memory.

---

## In This Repo's Architecture

mem0 is in Layer 1 because it's foundational infrastructure — it goes *under* the application, not *around* the developer. It provides the theoretical model (User / Session / Agent memory tiers) and the LOCOMO benchmark result that validates structured memory extraction over raw history storage.

**Relevant patterns:** [File Memory Structure](../../patterns/file-memory-structure.md)
