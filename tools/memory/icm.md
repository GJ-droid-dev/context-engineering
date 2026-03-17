# ICM (Intelligent Context Memory)

> MCP-native session memory for developer workflows. Captures what happens in a session — code written, decisions made, approaches tried — so the next session can pick up exactly where you left off.

**Source:** [rtk-ai/icm](https://github.com/rtk-ai) — 70 stars  
**Layer:** 3 — Session & Project Memory  
**WISC:** W + S (session-level write and retrieval)

---

## The Problem It Solves

Sessions end. Context vanishes. The next day you open a new chat and spend the first 15 minutes re-establishing what you were doing, what you tried, what didn't work, and what the next step was. For a 60-day project, that's roughly 15 hours of re-orientation overhead.

ICM captures session memory episodically — like a journal — and retrieves the relevant slice when you need it. You stop explaining. You start working.

---

## How It Works

ICM is an MCP server that persists session memories across conversations. It stores two types:

- **Episodic memory** — timestamped records of what happened ("fixed the rate limiter bug by adjusting windowMs")  
- **Semantic memory** — facts and patterns derived from sessions ("this codebase uses asyncHandler on all controllers")

At session start, you query ICM for recent or relevant memory. At session end (or mid-session), the agent logs what it learned.

---

## Setup

**Install:**
```bash
npm install -g @rtk-ai/icm
```

**Initialize for a project:**
```bash
icm init
```
This creates `.icm/` in your project root and generates the MCP config automatically for VS Code.

**Manual VS Code config:**
```json
// .vscode/mcp.json
{
  "servers": {
    "icm": {
      "command": "npx",
      "args": ["-y", "@rtk-ai/icm", "serve"]
    }
  }
}
```

---

## Key MCP Tools

### Writing memory

```
store_memory(
  type: "EPISODIC",
  content: "Debugged the payment webhook signature mismatch — Razorpay sends raw body, must not parse before verification",
  tags: ["payment", "razorpay", "debugging"]
)
```

```
store_memory(
  type: "SEMANTIC",
  content: "All protected routes require both authenticate() and authorize() middleware in that order",
  tags: ["auth", "pattern"]
)
```

### Retrieving memory

```
recall_recent(hours: 24)
recall_relevant(query: "payment webhook", limit: 5)
search_memory(query: "razorpay signature", type: "EPISODIC")
```

---

## Recommended Session Pattern

**Session start:**
```
recall_recent(hours: 72)
recall_relevant(query: "[current task]", limit: 3)
```

**Session end (or after completing a significant task):**
```
store_memory(type: "EPISODIC", content: "Completed [task]. Key finding: [finding]")
store_memory(type: "SEMANTIC", content: "[Any new pattern or rule discovered]")
```

---

## ICM vs. ConPort vs. Basic Memory

| Dimension | ICM | ConPort | Basic Memory |
|-----------|-----|---------|--------------|
| Memory type | Episodic (what happened) | Structured (decisions, patterns) | Graph (entities + relations) |
| Scope | Developer session | Project lifetime | Project lifetime |
| Structure | Tagged free text | Explicit schema | Markdown + inferred graph |
| Best for | "What was I doing?" | "Why did we choose X?" | "What does Y relate to?" |
| Versioned | No (ephemeral sessions) | Yes (commit `.conport/`) | Yes (commit `./memory/`) |

**They are complementary.** ICM captures the *session narrative*; ConPort captures the *project facts*; Basic Memory captures the *knowledge graph*.

---

## In This Repo's Architecture

ICM is the **session memory layer** — the episodic journal that turns session boundaries from a context cliff into a bookmark. It sits alongside ConPort and Basic Memory in Layer 3, covering the temporal dimension neither of them focuses on.

**Relevant patterns:** [Progressive Loading](../../patterns/progressive-loading.md)
