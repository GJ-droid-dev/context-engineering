# Context Engineering for Cline

> Cline is a VS Code extension with deep agentic capabilities and a native `.clinerules` system. It supports MCP servers, multi-file edits, and is designed around the principle that context engineering should be explicit and controllable.

---

## What Cline Supports Natively

| Feature | How |
|---------|-----|
| Project rules | `.clinerules/` directory (per-project) |
| Global rules | Cline Settings → Custom Instructions |
| MCP servers | Cline Settings → MCP Servers tab (GUI) or `.cline/mcp.json` |
| File references | `@url`, `@file`, `@folder` in chat |
| Sub-agent workflows | Agent mode (autonomous multi-step) |
| Browser integration | Built-in browser tool |
| Memory | Via MCP servers (ConPort, Basic Memory, etc.) |

---

## The `.clinerules/` Directory

This is Cline's defining context feature: a directory of markdown files that define project-specific rules. Cline reads these files and applies them to every interaction.

```
.clinerules/
├── project.md          ← core project context (always loaded)
├── backend.md          ← backend-specific rules
├── frontend.md         ← frontend-specific rules
├── payments.md         ← payment domain rules
└── testing.md          ← test patterns and commands
```

Unlike `applyTo` filtering in Copilot, Cline loads `.clinerules/` files based on the task context. You can use a single flat file or split by domain.

**A minimal `.clinerules/project.md`:**

```markdown
# Project Context

## What This Is
[product summary]

## Tech Stack
- Backend: Node.js/Express/TypeScript/Prisma
- Database: PostgreSQL
- Frontend: Next.js 14, MUI

## Key Rules
- asyncHandler wrapper on all controllers
- Never spread Prisma results
- Use @/ path aliases always

## Key Locations
- Schema: backend/prisma/schema.prisma
- Auth: backend/src/middlewares/auth.ts
```

---

## Layer 1 — Foundational Memory

**Set up via Cline's MCP GUI:**  
Cline Settings → MCP Servers → Add Server

Or add directly:
```json
{
  "memory": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-memory"]
  }
}
```

**Hindsight (production agents that need to learn):**  
Editor-agnostic — deploy as a Docker service, integrate via client SDK. In Cline workflows, call `retain` after significant decisions, `recall` at task start to retrieve relevant context, and `reflect` at phase transitions to synthesize abstract understanding across sessions. See [tools/memory/hindsight.md](../tools/memory/hindsight.md).

---

## Layer 2 — Documentation Context

**Context7:**
Add via Cline MCP GUI or config:
```json
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp@latest"]
  }
}
```

Usage: add `use context7` to your prompt. Cline will inject current docs for the library referenced.

**code-review-graph — structural context selection:**  
A Claude Code plugin (`claude plugin marketplace add tirth8205/code-review-graph`) that computes the blast-radius dependency graph of any change. Cline’s agentic mode pairs well with this — load only the files actually affected by a change rather than the full codebase. Achieves avg 6.8x token reduction. See [tools/docs/code-review-graph.md](../tools/docs/code-review-graph.md).

---

## Layer 3 — Session & Project Memory

**ConPort for Cline** (ConPort includes a Cline strategy file):
```json
{
  "conport": {
    "command": "npx",
    "args": ["-y", "context-portal", "serve"]
  }
}
```

**Basic Memory:**
```json
{
  "basic-memory": {
    "command": "uvx",
    "args": ["basic-memory", "mcp", "--path", "./memory"]
  }
}
```

**Instruct Cline to use it:**
Add to `.clinerules/project.md`:
```markdown
## Memory Protocol
After completing any significant task or discovering a non-obvious fact:
- Write an Observation to basic-memory about what was learned
- Tag it with the relevant domain

At session start:
- Call `recall_recent(hours: 72)` via basic-memory
```

---

## Layer 4 — Workflow Orchestration

### Phase structure in `.clinerules/`

Implement a lightweight phase system using Cline's custom instructions:

```markdown
<!-- .clinerules/workflow.md -->

## Task Phases
Before implementing anything, declare the phase:

PLAN phase: Analyze the task. Output a written plan (files to change, approach, steps).
BUILD phase: Implement the plan. Load only relevant rule files.
REVIEW phase: Verify the implementation meets all patterns and tests pass.

Do not enter BUILD without an approved plan.
Do not mark complete without running verification.
```

### ECC-style skills in Cline

Create domain skill files and reference them by name:

```
.clinerules/
├── skills/
│   ├── backend-patterns.md
│   ├── payment-domain.md
│   └── migration-workflow.md
```

Reference in chat:
```
Load the backend-patterns skill from .clinerules/skills/backend-patterns.md
Then implement the following endpoint...
```

---

## Full `.cline/mcp.json` (if used)

```json
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp@latest"]
  },
  "conport": {
    "command": "npx",
    "args": ["-y", "context-portal", "serve"]
  },
  "basic-memory": {
    "command": "uvx",
    "args": ["basic-memory", "mcp", "--path", "./memory"]
  }
}
```

---

## Recommended File Structure for Cline Projects

```
project-root/
├── .clinerules/
│   ├── project.md              ← core context (always included)
│   ├── backend.md              ← backend domain rules
│   ├── frontend.md             ← frontend domain rules
│   ├── payments.md             ← payment domain rules
│   ├── testing.md              ← test patterns + commands
│   ├── workflow.md             ← phase protocol
│   └── skills/
│       ├── backend-patterns.md
│       └── migration-workflow.md
├── memory-bank/
│   ├── projectBrief.md
│   ├── systemPatterns.md
│   ├── activeContext.md
│   └── progress.md
└── tasks/
    ├── todo.md
    ├── lessons.md
    └── HANDOFF.md
```

---

## Session Start Checklist (Cline)

```
1. Open Cline chat
2. @file:memory-bank/activeContext.md @file:tasks/HANDOFF.md
3. "Read these files. Recall recent activity from basic-memory. 
    Then: [task description]. Start in PLAN phase."
4. Agent plans and waits for your approval before building.
```

---

## Cline-Specific Tips

- **`.clinerules/` is your most powerful lever.** Invest in structuring it well. Everything here is automatically available.
- **Cline's autonomous mode is aggressive.** Use the phase protocol to add checkpoints, especially for migrations or auth changes.
- **`@url` is underused.** You can pass library docs directly from a URL: `@url:https://orm.drizzle.team/docs/select` — a lightweight alternative to Context7 for one-off docs.
- **Basic Memory suits Cline well** because Cline generates a lot of conversation artifacts that are worth preserving as knowledge graph nodes.
- **Commit `.clinerules/`** — it's the shared team context. New team members get a fully oriented agent by cloning.
- **Context files rot if you append.** `activeContext.md`, `todo.md`, and `HANDOFF.md` are state snapshots — rewrite them, don't grow them. See [Context Pruning](../patterns/context-pruning.md) for the size thresholds and per-file rules.
