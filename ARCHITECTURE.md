# Architecture: Context Engineering for AI Coding Agents

This document maps all 16 source tools onto a unified 5-layer context architecture. Use it to understand why each tool exists, where it sits in the stack, and how layers combine.

---

## The Core Mental Model: WISC

Before diving into layers, the organizing framework for this entire repo is **WISC** — four dimensions that describe *how* context enters a session:

| Letter | Dimension | Question It Answers |
|--------|-----------|---------------------|
| **W** | **Write** | What persistent content do we pre-author for the agent? |
| **I** | **Isolate** | Which sub-agents or phases get only their relevant slice? |
| **S** | **Select** | Which context files get loaded for *this* task, not all tasks? |
| **C** | **Compress** | How do we reduce token cost of what must be included? |

Every tool in this repo addresses at least one WISC dimension. Most address two.

Full WISC breakdown: [framework/wisc.md](framework/wisc.md)

---

## The 5-Layer Stack

```
╔═══════════════════════════════════════════════════════════════════╗
║  LAYER 5 — COMPRESSION                                           ║
║  Reduce the token cost of content that must be in context        ║
║                                                                   ║
║  Tools:  RTK                                                      ║
║  WISC:   C                                                        ║
╠═══════════════════════════════════════════════════════════════════╣
║  LAYER 4 — WORKFLOW ORCHESTRATION                                 ║
║  Phase-gate what context loads; isolate sub-agents               ║
║                                                                   ║
║  Tools:  Cursor Memory Bank, ECC, WISC framework                 ║
║  WISC:   I + S                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║  LAYER 3 — SESSION & PROJECT MEMORY                              ║
║  Persist decisions, patterns, and progress across sessions       ║
║                                                                   ║
║  Tools:  ConPort, Basic Memory, ICM                              ║
║  WISC:   W + S                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║  LAYER 2 — DOCUMENTATION CONTEXT                                  ║
║  Real-time, versioned, correct library docs in every prompt      ║
║                                                                   ║
║  Tools:  Context7, Context Hub                                    ║
║  WISC:   S (just-in-time selection)                               ║
╠═══════════════════════════════════════════════════════════════════╣
║  LAYER 1 — FOUNDATIONAL MEMORY                                    ║
║  Application-level memory infrastructure for production agents   ║
║                                                                   ║
║  Tools:  mem0, MCP Memory Server, LlamaIndex                     ║
║  WISC:   W (write + retrieve)                                     ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## Layer 1 — Foundational Memory

**What it is:** The memory infrastructure that production AI agents use to remember users, sessions, and its own past reasoning across API calls.

**Why it exists:** Without this layer, every agent request is stateless. The agent can't remember that a user prefers TypeScript, that a payment flow was redesigned last sprint, or that a particular approach was abandoned with a reason.

**Tools:**

| Tool | What It Does | Best For |
|------|--------------|----------|
| [mem0](tools/memory/mem0.md) | TypeScript/Python SDK; three-tier memory (User / Session / Agent); +26% LOCOMO benchmark | Production apps; agents that need user-level memory |
| [MCP Memory Server](https://github.com/modelcontextprotocol/servers) | Anthropic's reference implementation; knowledge graph via MCP protocol | Connecting memory to any MCP-compatible editor |
| [LlamaIndex](https://github.com/run-llama/llama_index) | RAG framework; hybrid BM25 + vector search | Large document corpora; semantic retrieval |
| [Hindsight](tools/memory/hindsight.md) | Biomimetic memory (world facts / experiences / mental models); retain + recall + reflect; SOTA LongMemEval | Production agents that need to learn, not just remember |

**Key pattern:** [File Memory Structure](patterns/file-memory-structure.md)

---

## Layer 2 — Documentation Context

**What it is:** Mechanism for pulling correct, current library documentation directly into prompts — not from training data, but from the live source.

**Why it exists:** LLMs hallucinate API signatures from outdated training data. The fix is not hope — it's injecting real docs at inference time. An agent using Context7 calling a React hook will see the *actual* current API, not a pre-training snapshot.

**Tools:**

| Tool | What It Does | Best For |
|------|--------------|----------|
| [Context7](tools/docs/context7.md) | MCP server; `npx ctx7 setup`; pulls docs from 49k+ libraries in real-time | Any modern library or framework in active development |
| [Context Hub](tools/docs/context-hub.md) | CLI; curated, versioned, annotatable doc bundles; MCP server | Internal APIs; niche libraries not in Context7's catalog |
| [code-review-graph](tools/docs/code-review-graph.md) | Tree-sitter AST graph; blast-radius analysis; computes minimal review set from static analysis; 6.8x avg token reduction | Code review; impact analysis; refactoring in large codebases |

**Key insight:** These tools are complementary. Context7 covers external library docs; Context Hub covers internal API docs; code-review-graph covers your own codebase structure. Context7 covers the long tail of public libraries. Context Hub covers internal, proprietary, or carefully curated sets.

**Key pattern:** [Project Brief Bootstrap](patterns/project-brief-bootstrap.md)

---

## Layer 3 — Session & Project Memory

**What it is:** Tools that persist project-specific knowledge — decisions made, patterns established, progress tracked — across multiple sessions and agents.

**Why it exists:** Every new session forgets everything. A developer spends 20 minutes re-explaining the project architecture, the tech stack constraints, the decisions already made. Layer 3 tools eliminate that tax by writing structured knowledge to queryable stores between sessions.

**Tools:**

| Tool | What It Does | Best For |
|------|--------------|----------|
| [Basic Memory](tools/memory/basic-memory.md) | Bi-directional: LLM reads AND writes Markdown files; SQLite + hybrid search | Letting the agent build its own knowledge base from conversation |
| [ConPort](tools/memory/conport.md) | SQLite project brain; `log_decision`, `log_progress`, `log_system_pattern` | Explicit, structured project knowledge graphs |
| [ICM](tools/memory/icm.md) | MCP-native; episodic + semantic session memory; VS Code `icm init` | Developer-session-level memory separate from project knowledge |

**Key distinction:**
- **ConPort** = the agent explicitly logs structured facts (you define the schema)
- **Basic Memory** = the agent freely writes observations and relations in Markdown (emergent structure)
- **ICM** = captures what happened *in this session* so the next session can resume

**Key patterns:** [Bidirectional Memory](patterns/bidirectional-memory.md), [Progressive Loading](patterns/progressive-loading.md)

---

## Layer 4 — Workflow Orchestration

**What it is:** Systems that control *when* context is loaded, *which* agent gets *which* slice, and *what phase* the work is in.

**Why it exists:** Loading everything at once is the naive approach and it fails. An agent writing a React component doesn't need the database schema. An agent doing discovery doesn't need the implementation rules. Orchestration is surgical context delivery.

**Measured result:** Cursor Memory Bank demonstrated ~70% token reduction by switching from load-everything to hierarchical lazy loading by phase.

**Tools:**

| Tool | What It Does | Best For |
|------|--------------|----------|
| [Cursor Memory Bank](tools/workflow/memory-bank.md) | 6-phase workflow (VAN→PLAN→CREATIVE→BUILD→REFLECT→ARCHIVE); complexity levels 1–4; hierarchical rule loading | Project workflows requiring strict phase control |
| [ECC (everything-claude-code)](tools/workflow/ecc.md) | 108 skills, 25 agents, 57 commands; CLAUDE.md hub; sub-agent orchestration | Full agent harness systems; skills-as-files pattern |
| [GSD-2](tools/workflow/gsd.md) | TypeScript agent runtime; fresh context window per task; .gsd/ disk-state machine; per-phase model routing; crash recovery | Autonomous long-running agent sessions; reference implementation of sub-agent isolation + phase-gating |

**Key patterns:** [Phase-Gated Context](patterns/phase-gated-context.md), [Sub-Agent Isolation](patterns/sub-agent-isolation.md)

---

## Layer 5 — Compression

**What it is:** Techniques and tools that reduce the token cost of content that *must* be in context, without reducing its information density.

**Why it exists:** Some context is mandatory — it must be present. Compression doesn't remove it; it shrinks it. At 60–90% reduction, the compression layer can double or triple how much useful context fits in a fixed window.

**Tools:**

| Tool | What It Does | Best For |
|------|--------------|----------|
| [RTK](tools/compression/rtk.md) | CLI proxy; rewrites verbose LLM I/O into compact form; 60–90% token reduction | High-frequency agent calls; large codebase analysis tasks |

**Key pattern:** [Progressive Loading](patterns/progressive-loading.md)

---

## Full Tool → Layer → WISC Mapping

| Tool | Layer | WISC | Notes |
|------|-------|------|-------|
| mem0 | 1 | W | Application memory SDK; 3 memory tiers |
| MCP Memory Server | 1 | W | Universal integration via MCP protocol |
| LlamaIndex | 1 | W | RAG + hybrid retrieval foundation |
| Context7 | 2 | S | Just-in-time doc injection via MCP |
| Context Hub | 2 | S + W | Curated doc bundles, versionable |
| Basic Memory | 3 | W + S | Bi-directional; LLM writes + reads |
| ConPort | 3 | W + S | Explicit structured logging; SQLite |
| ICM | 3 | W + S | Session-level episodic memory |
| Cursor Memory Bank | 4 | I + S | Phase-gating; ~70% token reduction |
| ECC | 4 | I + W | Skills-as-files; agent orchestration |
| WISC Framework | 4 | W+I+S+C | The organizing philosophy |
| GSD-2 | 4 | I + S + W | Reference implementation; fresh context per task |
| RTK | 5 | C | 60–90% I/O token compression |
| Hindsight | 1 | W | Biomimetic memory; retain/recall/reflect operations |
| code-review-graph | 2 | S | Blast-radius graph; structural context selection |

---

## What Good Context Architecture Looks Like

A well-engineered context stack for a typical enterprise project would combine:

```
1. mem0 or MCP Memory Server       ← foundational user/session memory
2. Context7 + Context Hub          ← always-current docs
3. ConPort or Basic Memory + ICM   ← project + session continuity
4. ECC pattern with skills files   ← task-specific context selection
5. RTK or manual compression       ← token budget management
```

Not all projects need all layers. The [editor guides](editors/) shows which layers are achievable in each editor today.

---

## Design Principles

1. **Curation beats completeness.** A 500-token, perfectly relevant context window beats a 4000-token dump with 80% noise.
2. **Write beats recall.** Explicitly authored context (W) outperforms hoping the model remembers. Write important things down.
3. **Isolation beats broadcasting.** Sub-agents with focused context outperform one agent with all context.
4. **Loading is a decision.** Every context file is a cost. Load files by task, not by default.
5. **Compression is additive.** Add the compression layer last, after the other four are working.
