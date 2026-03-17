# ConPort (Context Portal)

> A SQLite-backed project knowledge graph for AI agents. Explicitly log decisions, system patterns, and progress — then let the agent retrieve exactly what's relevant via semantic search.

**Source:** [GreatScottyMac/context-portal](https://github.com/GreatScottyMac/context-portal) — 755 stars  
**Layer:** 3 — Session & Project Memory  
**WISC:** W + S (structured logging; semantic retrieval)

---

## The Problem It Solves

Sessions end, context is lost. A developer re-explains the same architectural decisions every new conversation. An agent reverses a choice made two days ago because it has no memory of the rationale.

ConPort is a structured "project brain" — a SQLite database where decisions, patterns, and progress are explicitly logged with linked relationships. The agent queries it at session start to re-orient, and queries it mid-task to retrieve relevant past decisions.

---

## How It Works

ConPort exposes a set of MCP tools that create structured entries in a SQLite database:

- **Decisions** — architectural choices with rationale and consequences
- **System Patterns** — recurring technical approaches used in the codebase
- **Progress** — what's been completed, what's next
- **Custom Data** — any structured facts you want persisted

Entries can be explicitly linked to each other. The agent can search by full-text or semantic similarity.

---

## Setup

**Install via npx:**
```bash
npx -y context-portal init
```

**VS Code MCP config:**
```json
// .vscode/mcp.json
{
  "servers": {
    "conport": {
      "command": "npx",
      "args": ["-y", "context-portal", "serve"]
    }
  }
}
```

**Per-project database** — ConPort creates a `.conport/` directory in your project root. Commit it to version control so the whole team shares the same knowledge graph.

---

## Core MCP Tools

### Logging (agent writes to the knowledge graph)

```
log_decision(
  summary: "Use Prisma over TypeORM",
  rationale: "Better TypeScript support; schema-first approach matches our workflow",
  alternatives_considered: ["TypeORM", "Drizzle"],
  tags: ["database", "orm"]
)
```

```
log_system_pattern(
  name: "asyncHandler wrapper",
  description: "All Express controllers wrapped in asyncHandler from @/utils/asyncHandler",
  tags: ["error-handling", "express"]
)
```

```
log_progress(
  status: "COMPLETED",
  description: "Payment collection flow with Razorpay",
  linked_item_type: "DECISION",
  linked_item_id: 12
)
```

### Retrieval (agent reads from the knowledge graph)

```
search_decisions_fts(query: "database")
get_decisions(tags: ["architecture"])
get_system_patterns()
get_recent_activity(hours: 48)
```

### Linking

```
link_conport_items(
  source_item_type: "DECISION",
  source_item_id: 5,
  target_item_type: "SYSTEM_PATTERN",
  target_item_id: 3,
  relationship_type: "IMPLEMENTS"
)
```

---

## Session Start Pattern

Add this to your system prompt or CLAUDE.md to make the agent re-orient at the start of every session:

```markdown
At the start of every session:
1. Call `get_recent_activity(hours: 72)` to see what changed recently
2. Call `get_system_patterns()` to recall established conventions
3. Load `activeContext.md` from the project root
```

---

## ConPort vs. Basic Memory

| | ConPort | Basic Memory |
|---|---------|--------------|
| Structure | Explicit, schema-driven | Emergent, free-form Markdown |
| Storage | SQLite | Markdown files + SQLite index |
| Who writes | Primarily the agent during task | Agent writes during conversation |
| Linking | Explicit typed relations | Wikilink-style `[[Entity]]` |
| Human editability | Via MCP tools | Direct file editing |
| Best for | Teams wanting structured, auditable knowledge | Flexible, exploratory knowledge building |

---

## Editor Compatibility

ConPort provides strategy files for multiple editors in the source repo:
- VS Code / GitHub Copilot
- Cursor
- Cline
- Windsurf
- Roo Code

---

## In This Repo's Architecture

ConPort is the **explicit structured knowledge graph** in Layer 3. It pairs well with ICM (which captures *what happened this session*) and complements Basic Memory (which is less structured but more flexible). Projects that need auditable, team-shared decision logs should prefer ConPort.

**Relevant patterns:** [Progressive Loading](../../patterns/progressive-loading.md), [File Memory Structure](../../patterns/file-memory-structure.md)
