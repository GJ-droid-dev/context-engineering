# code-review-graph

> A local knowledge graph for your codebase. Builds a persistent structural map from Tree-sitter ASTs — functions, classes, imports, call edges — so your agent reads only the files that matter for a given change.

**Source:** [tirth8205/code-review-graph](https://github.com/tirth8205/code-review-graph) — 1.6k stars  
**Layer:** 2 — Documentation Context  
**WISC:** S (structural selection — load by dependency graph, not by heuristic)

---

## The Problem It Solves

Context7 and Context Hub solve the *library documentation* problem: inject the right API docs so the agent knows what it's calling. code-review-graph solves a different problem: inject the right *codebase files* so the agent understands what it's modifying.

Without structural context selection, agents either:
1. **Read too little:** Miss the callers of a function they're changing, break things they didn't know they'd affect
2. **Read too much:** Load the entire repo, burn tokens on files unrelated to the change

code-review-graph computes the **blast radius** of any change — the minimal set of files the agent actually needs to read — from a pre-built static analysis graph.

**Measured results:**
- 6.8x average token reduction on code review tasks
- 14.1x average on live coding tasks
- 49x peak on monorepo changes

---

## How It Works

The repository is parsed into an AST with Tree-sitter, stored as a graph of nodes (functions, classes, imports) and edges (calls, inheritance, test coverage), then queried at review time:

```
Repository → Tree-sitter Parser → SQLite Graph → Blast Radius Analysis → Minimal Review Set
```

1. **Initial build:** ~10 seconds for a 500-file project; stored in `.code-review-graph/` (SQLite, local, no cloud)
2. **Incremental updates:** Re-parses only changed files; subsequent updates complete in under 2 seconds
3. **Auto-update:** Graph updates on every file edit and git commit via hooks — no manual intervention

---

## Supported Languages

Python, TypeScript, JavaScript, Go, Rust, Java, C#, Ruby, Kotlin, Swift, PHP, C/C++

---

## Setup

**Claude Code plugin (recommended):**
```bash
claude plugin marketplace add tirth8205/code-review-graph
claude plugin install code-review-graph@code-review-graph
# Restart Claude Code, then:
# "Build the code review graph for this project"
```

**pip:**
```bash
pip install code-review-graph
code-review-graph install
# Restart Claude Code
```

Requires Python 3.10+ and `uv`. Works as an MCP tool — Claude Code calls the graph query tools automatically during review and coding tasks.

---

## What the Agent Gets

When reviewing a change, instead of loading all potentially related files, the agent receives the computed blast radius:

```
Changed: backend/src/services/payment.service.ts

Blast radius (computed):
  Direct callers:    backend/src/routes/payment.routes.ts
                     backend/src/controllers/checkout.controller.ts
  Inheritors:        (none)
  Test coverage:     tests/payment.service.test.ts

  Minimal review set: 4 files (vs. 47 in naive approach)
```

---

## code-review-graph vs. Other Layer 2 Tools

| Dimension | code-review-graph | Context7 | Context Hub |
|-----------|------------------|----------|-------------|
| Context type | Your codebase structure | External library docs | Curated internal doc bundles |
| Selection method | Dependency graph (blast radius) | Semantic query | Keyword / versioned bundle |
| Storage | Local SQLite in `.code-review-graph/` | MCP server (remote) | Local CLI + MCP server |
| Update mechanism | Incremental on file change | Real-time from live sources | Manual curation |
| Best for | Code review, refactoring, impact analysis | Using external libraries | Internal APIs, niche libraries |

These tools are complementary: Context7 injects external library docs; Context Hub injects internal API docs; code-review-graph injects the right files from your own codebase.

---

## In This Repo's Architecture

code-review-graph sits in **Layer 2 — Documentation Context** because its job is context selection — deciding *which* content loads, not storing or persisting it. It implements the structural graph variant of [Progressive Loading](../../patterns/progressive-loading.md): instead of triggering context loads by file-path heuristics, it computes the exact dependency set from static analysis.

**Related patterns:** [Progressive Loading](../../patterns/progressive-loading.md), [Sub-Agent Isolation](../../patterns/sub-agent-isolation.md)
