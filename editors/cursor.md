# Context Engineering for Cursor

> Cursor has the deepest native support for context engineering of any editor. This guide covers the full stack: `.cursor/rules/`, Memory Bank phases, MCP servers, and the CLAUDE.md equivalent.

---

## What Cursor Supports Natively

| Feature | How |
|---------|-----|
| Project rules | `.cursor/rules/` — MDC files with `applyTo` |
| Global rules | Cursor Settings → Rules for AI |
| MCP servers | `.cursor/mcp.json` |
| File references | `@file`, `@folder`, `@codebase` in chat |
| Sub-agent workflows | Agent mode (Composer) |
| Memory Bank | Via `.cursor/rules/` phase files |

---

## Layer 1 — Foundational Memory

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**Hindsight (production agents that need to learn):**  
Editor-agnostic — deploy as a Docker service, integrate via client SDK. In Cursor workflows, call `retain` to store decisions and patterns, `recall` to retrieve them, and `reflect` at phase transitions to synthesize abstract understanding. See [tools/memory/hindsight.md](../tools/memory/hindsight.md).

---

## Layer 2 — Documentation Context

**Context7:**
```json
// .cursor/mcp.json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

**code-review-graph — structural context selection:**  
A Claude Code plugin that uses AST dependency graphs to identify the blast radius of any change. Install in Claude Code: `claude plugin marketplace add tirth8205/code-review-graph`. Achieves avg 6.8x token reduction. In Cursor, replicate the pattern manually with `@file` references scoped to files in the dependency graph of your change. See [tools/docs/code-review-graph.md](../tools/docs/code-review-graph.md).

---

## Layer 3 — Session & Project Memory

**ConPort for Cursor** (ConPort provides a Cursor-specific strategy file in its source repo):
```json
// .cursor/mcp.json
{
  "mcpServers": {
    "conport": {
      "command": "npx",
      "args": ["-y", "context-portal", "serve"]
    }
  }
}
```

**Basic Memory:**
```json
{
  "mcpServers": {
    "basic-memory": {
      "command": "uvx",
      "args": ["basic-memory", "mcp", "--path", "./memory"]
    }
  }
}
```

---

## Layer 4 — Workflow Orchestration

### `.cursor/rules/` — the MDC system

Cursor rules are MDC (Markdown with frontmatter) files. The `applyTo` glob pattern controls automatic loading.

```markdown
---
description: Backend patterns for Express/TypeScript
globs: backend/src/**/*.ts
alwaysApply: false
---

# Backend Rules

## Error Handling
All controllers: asyncHandler wrapper.
Errors: throw new AppError(msg, statusCode)

## Database
Never spread Prisma results. Always destructure explicitly.
```

**`alwaysApply: true`** — loads for every agent interaction (use for universal rules only)  
**`alwaysApply: false` with `globs`** — loads automatically when editing matching files  
**Neither** — loads only when you reference it manually

### The Memory Bank Phase System in Cursor

The [Cursor Memory Bank](../tools/workflow/memory-bank.md) was designed for Cursor. Set it up:

**Install:**
```bash
git clone https://github.com/vanzan01/cursor-memory-bank
cp cursor-memory-bank/.cursor/rules/* .cursor/rules/
cp cursor-memory-bank/memory-bank/ ./memory-bank/
```

This gives you pre-built rule files:
```
.cursor/rules/
├── isolation_rules/
│   ├── visual-maps/
│   ├── Core_Workflows/
│   └── Phases/
│       ├── van-rules.md        ← VAN phase rules
│       ├── plan-rules.md       ← PLAN phase rules
│       ├── creative-rules.md   ← CREATIVE phase rules
│       ├── implement-rules.md  ← BUILD phase rules
│       └── reflect-archive.md  ← REFLECT + ARCHIVE rules
└── main.mdc                    ← entry point
```

**Usage:**
```
# Start a task:
"Enter VAN mode for: [task description]"

# The agent reads van-rules.md, assesses complexity, announces phase
# Then you confirm progression through phases
```

---

## Full `.cursor/mcp.json`

```json
{
  "mcpServers": {
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

## Recommended File Structure for Cursor Projects

```
project-root/
├── .cursor/
│   ├── rules/
│   │   ├── main.mdc                    ← hub (alwaysApply: true, short)
│   │   ├── backend-rules.mdc           ← globs: backend/src/**
│   │   ├── frontend-rules.mdc          ← globs: frontend/src/**
│   │   ├── payments-rules.mdc          ← globs: **/payment*,**/settlement*
│   │   └── phases/                     ← Memory Bank phase files
│   │       ├── van-rules.md
│   │       ├── plan-rules.md
│   │       ├── build-rules.md
│   │       └── reflect-rules.md
│   └── mcp.json
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

## Session Start Checklist (Cursor)

```
1. Open Composer (Cmd+I / Ctrl+I)
2. @memory-bank/activeContext.md @tasks/HANDOFF.md
3. "Enter VAN mode for: [task]"
4. Agent assesses complexity and enters the appropriate phase
```

---

## Cursor-Specific Tips

- **Use Composer for multi-file tasks.** The inline editor is for single-file changes; Composer is for agent-mode work.
- **`.cursor/rules/main.mdc` should be your routing table**, not a full dump. `alwaysApply: true` files load for every message — keep them under 200 tokens.
- **Use `alwaysApply: false` with `globs`** for domain rules. This is the Cursor equivalent of Copilot's `applyTo`.
- **Combine Memory Bank with ConPort.** Memory Bank handles the workflow phases; ConPort handles the persistent knowledge graph.
- **Commit `.cursor/rules/`** to version control so the whole team benefits.
