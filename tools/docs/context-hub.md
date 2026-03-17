# Context Hub

> Curated, versioned, annotatable documentation bundles for your AI agent. Where Context7 covers the public long tail, Context Hub covers your internal and hand-selected docs.

**Source:** [andrewyng/context-hub](https://github.com/andrewyng/context-hub) — 8.8k stars  
**Layer:** 2 — Documentation Context  
**WISC:** S + W (curated selection, human-authored)

---

## The Problem It Solves

Context7 is excellent for public libraries. But most enterprises also have:

- Internal APIs that aren't in any public catalog
- Third-party services where the official docs are noisy and you want a curated subset
- Legacy systems where you want to document conventions that live nowhere
- Specific versions of docs you want pinned (not auto-updated)

Context Hub is a CLI that lets you create, version, and annotate documentation bundles — and serve them via an MCP server to any compatible agent.

---

## How It Works

Context Hub maintains a local store of documentation bundles. You add docs from URLs, files, or raw text. You can annotate them with gotchas, notes, and version pin information. The MCP server exposes these bundles to your agent on request.

```
context-hub
├── bundles/
│   ├── internal-payment-api.md      ← your proprietary API docs
│   ├── stripe-webhooks-v3.md        ← pinned to v3, not auto-updated
│   └── company-coding-standards.md  ← conventions not in any linter
```

---

## Setup

```bash
npm install -g context-hub
context-hub init
```

**Add documentation:**
```bash
# From a URL
context-hub add --name "stripe-webhooks" --url "https://stripe.com/docs/webhooks"

# From a local file
context-hub add --name "internal-api" --file ./docs/internal-api.md

# With annotations
context-hub add --name "legacy-auth" --file ./docs/auth.md \
  --note "This uses the old session system. JWT migration is in progress."
```

**Start the MCP server:**
```bash
context-hub serve
```

**Connect to VS Code:**
```json
// .vscode/mcp.json
{
  "servers": {
    "context-hub": {
      "command": "context-hub",
      "args": ["serve", "--mcp"]
    }
  }
}
```

---

## Key Features

- **Versioning:** Pin a doc bundle to a specific state; it won't auto-update
- **Annotations:** Add human notes to docs ("this method is deprecated but still used in prod")
- **Open format:** All content is stored as markdown — inspect and edit directly
- **Team sharing:** Commit your context-hub bundles to version control so the whole team's agent uses the same docs

---

## Context7 vs. Context Hub

| | Context7 | Context Hub |
|---|----------|-------------|
| Coverage | 49k+ public libraries | Whatever you add |
| Updates | Automatic, live | Manual, versioned |
| Internal docs | No | Yes |
| Annotations | No | Yes |
| Setup | MCP config only | CLI + MCP |
| Best for | React, Stripe, Next.js, etc. | Internal APIs, pinned versions, annotated docs |

**Use both.** They address different problems.

---

## In This Repo's Architecture

Context Hub is the **stable curated docs layer** — the human-authored complement to Context7's machine-indexed public catalog. It also serves double duty as a W-layer tool: the docs you author and annotate here are persistent knowledge that survives session resets.

**Relevant patterns:** [Project Brief Bootstrap](../../patterns/project-brief-bootstrap.md)
