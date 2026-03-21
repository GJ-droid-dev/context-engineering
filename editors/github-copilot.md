# Context Engineering for GitHub Copilot

> A practical guide to implementing the full 5-layer context architecture in VS Code with GitHub Copilot. Covers what's natively supported, what requires MCP, and what you emulate manually.

---

## What Copilot Supports Natively

Before installing anything, understand what's already available:

| Feature | How |
|---------|-----|
| Project-wide instructions | `.github/copilot-instructions.md` |
| File-specific instructions | `.instructions.md` files with `applyTo` frontmatter |
| Chat context | `#file:`, `#folder:`, `#codebase` references |
| MCP servers | `.vscode/mcp.json` (VS Code 1.99+) |
| Memory (via MCP) | Any MCP-compatible memory server |
| Agent mode | Copilot Chat → Agent mode |
| Custom agents | `.github/agents/` (Copilot agent customization) |
| Prompt files | `.github/prompts/` (reusable prompt templates) |

---

## Layer 1 — Foundational Memory

**Option A: MCP Memory Server (Anthropic reference)**
```json
// .vscode/mcp.json
{
  "servers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```
Gives the agent `create_entities`, `create_relations`, `search_nodes`, `open_nodes` tools.

**Option B: mem0 (if you're building a product with user memory)**  
Use the mem0 TypeScript SDK in your application backend. Not directly in Copilot — mem0 is for production agents, not developer tooling.

**Option C: Hindsight (production agents that need to learn over time)**  
Deploy Hindsight as a Docker service and connect via the Python or TypeScript client. Adds `retain`, `recall`, and `reflect` operations. The key differentiator is `reflect`: it synthesizes accumulated memory into abstract understanding at phase transitions, not just stores raw facts. See [tools/memory/hindsight.md](../tools/memory/hindsight.md).

---

## Layer 2 — Documentation Context

**Context7 — live library docs via MCP:**
```json
// .vscode/mcp.json
{
  "servers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```
Add `use context7` to any prompt to inject current docs for the library you're using.

**Context Hub — internal and curated docs:**
```bash
npm install -g context-hub
context-hub init
context-hub add --name "internal-api" --file ./docs/api.md
context-hub serve --mcp
```
```json
// .vscode/mcp.json
{
  "servers": {
    "context-hub": { "command": "context-hub", "args": ["serve", "--mcp"] }
  }
}
```

**code-review-graph — structural context selection (Claude Code plugin):**  
Installs as a Claude Code plugin (`claude plugin marketplace add tirth8205/code-review-graph`), not a VS Code MCP server. If you use Claude Code alongside Copilot, add it there for 6.8x average token reduction via blast-radius dependency graph analysis. The pattern — load only files in the change’s dependency graph — is manually achievable in Copilot via targeted `#file:` references. See [tools/docs/code-review-graph.md](../tools/docs/code-review-graph.md).

---

## Layer 3 — Session & Project Memory

**Option A: ConPort (recommended for teams)**
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
Agent gets `log_decision`, `log_system_pattern`, `search_decisions_fts`, etc.

**Option B: Basic Memory (recommended for individuals)**
```json
// .vscode/mcp.json
{
  "servers": {
    "basic-memory": {
      "command": "uvx",
      "args": ["basic-memory", "mcp", "--path", "./memory"]
    }
  }
}
```

**Option C: File-based (no MCP needed)**  
Use the [File Memory Structure](../patterns/file-memory-structure.md) pattern. Manually load files via `#file:` in chat.

---

## Layer 4 — Workflow Orchestration

### `.github/copilot-instructions.md` — the hub

This file loads automatically for every Copilot interaction project-wide:

```markdown
<!-- .github/copilot-instructions.md -->
# Project Context Hub

## What This Is
[1-paragraph summary]

## Tech Stack
[bullet list]

## Key Rules (always apply)
- [3-5 non-negotiable rules]

## For Specific Domains
Load the relevant instructions file for your task:
- Backend: `.github/instructions/backend.instructions.md`
- Frontend: `.github/instructions/frontend.instructions.md`
- Payments: `.github/instructions/payments.instructions.md`
```

### Domain-specific instructions with `applyTo`

```markdown
---
applyTo: "backend/src/**"
---
<!-- .github/instructions/backend.instructions.md -->

# Backend Rules

- All controllers: asyncHandler wrapper required
- Errors: throw new AppError(msg, code) — never res.json() directly
- Auth: every protected route needs authenticate + authorize middleware
- DB: never spread Prisma results — always explicit field picks
```

Files with `applyTo` patterns load **automatically** when you open or edit matching files — no manual loading needed.

### Prompt files for reusable workflows

```markdown
<!-- .github/prompts/implement-endpoint.prompt.md -->
# Implement API Endpoint

Load: #file:.github/instructions/backend.instructions.md

Implement the following endpoint following all backend rules:
- Route: {{route}}
- Method: {{method}}  
- Description: {{description}}
- Request body: {{requestBody}}
- Expected response: {{response}}

Use asyncHandler, AppError, and sendSuccess/sendCreated from @/utils/response.
```

---

## Layer 5 — Compression

**Practical compression for Copilot without RTK:**

1. Keep all instruction files under 500 tokens
2. Use tables and bullet lists instead of prose
3. Structure rules as code examples, not descriptions
4. Keep `copilot-instructions.md` as a routing table, not a dump
5. For large context files, split by domain and use `applyTo`

---

## Full .vscode/mcp.json (all layers)

```json
{
  "servers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "conport": {
      "command": "npx",
      "args": ["-y", "context-portal", "serve"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

---

## Recommended File Structure for Copilot Projects

```
project-root/
├── .github/
│   ├── copilot-instructions.md          ← hub file (always loads)
│   ├── instructions/
│   │   ├── backend.instructions.md      ← applyTo: backend/src/**
│   │   ├── frontend.instructions.md     ← applyTo: frontend/src/**
│   │   ├── payments.instructions.md     ← applyTo: **/payment*
│   │   └── testing.instructions.md      ← applyTo: **/tests/**
│   └── prompts/
│       ├── implement-endpoint.prompt.md
│       ├── write-tests.prompt.md
│       └── review-changes.prompt.md
├── .vscode/
│   └── mcp.json                         ← MCP servers
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

## Session Start Checklist (Copilot)

```
1. Open a new Copilot Chat
2. Reference: #file:memory-bank/activeContext.md #file:tasks/HANDOFF.md
3. Say: "Read these files and confirm you understand the current state."
4. The agent is now oriented. Start your task.
```

---

## Practical Tips

- **`applyTo` is the most underused feature.** Set it up for every domain and your agent will automatically have the right rules without you doing anything.
- **One prompt file per workflow.** "Implement endpoint", "Write tests", "Review diff", "Run migration" — each is a prompt file.
- **Keep `copilot-instructions.md` short.** If it's over 600 tokens, split it. It loads for every interaction.
- **Use agent mode for multi-file tasks.** Chat mode is for Q&A. Agent mode is for implementation work spanning multiple files.
- **Context files rot if you append.** `activeContext.md`, `todo.md`, and `HANDOFF.md` are state snapshots — rewrite them, don't grow them. See [Context Pruning](../patterns/context-pruning.md) for the size thresholds and per-file rules.
