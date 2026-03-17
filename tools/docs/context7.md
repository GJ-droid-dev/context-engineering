# Context7

> Real-time library documentation pulled directly into your agent's context via MCP. No more hallucinated APIs from stale training data.

**Source:** [upstash/context7](https://github.com/upstash/context7) — 49.5k stars  
**Layer:** 2 — Documentation Context  
**WISC:** S (just-in-time selection)

---

## The Problem It Solves

LLMs are trained on a snapshot of the internet. That snapshot is months or years old. When your agent writes code using a library that changed its API since training cutoff, it produces plausible-looking but broken code. You don't find out until the test suite fails.

Context7 intercepts the prompt and injects the *actual current documentation* for whatever library your agent is about to use.

---

## How It Works

Context7 runs as an MCP server. When your editor sends a prompt, the server:

1. Detects which libraries are referenced (by import, by name, or by your `/use context7` instruction)
2. Fetches current docs from the Context7 library catalog (49k+ libraries indexed)
3. Injects the relevant documentation segment into the prompt before the LLM sees it

The agent then answers from real docs, not from training memory.

---

## Setup

**VS Code / GitHub Copilot:**
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

**One-liner install (global):**
```bash
npx ctx7 setup
```

**Cursor:**
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

---

## Usage

Add `use context7` to your prompt to trigger doc injection:

```
# Agent prompt
Use context7. Implement a rate limiter using the express-rate-limit library.
```

The agent will receive the current `express-rate-limit` docs automatically — including the correct `windowMs` and `max` options for whatever version is current.

---

## What It Covers

- 49,000+ libraries and frameworks indexed
- Auto-updates as libraries release new versions
- Works with any MCP-compatible editor (VS Code, Cursor, Windsurf, Zed, Claude Desktop)
- Open catalog — you can submit libraries that aren't yet indexed

---

## Limitations

- Requires network access at query time
- Coverage depends on the catalog — very niche or internal libraries won't be in it
- For internal APIs and proprietary docs, use [Context Hub](context-hub.md) instead

---

## In This Repo's Architecture

Context7 is the **live docs layer**. It covers the long tail of public libraries. Pair it with [Context Hub](context-hub.md) for internal or curated documentation sets.

**Relevant patterns:** [Project Brief Bootstrap](../../patterns/project-brief-bootstrap.md)
