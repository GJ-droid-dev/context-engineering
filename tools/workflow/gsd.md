# GSD 2

> A standalone coding agent runtime that implements context engineering as a TypeScript application — not as prompts. Every task runs in a fresh context window with exactly the right files pre-inlined. Fresh session per task, programmatic context control, state on disk.

**Source:** [gsd-build/gsd-2](https://github.com/gsd-build/gsd-2) — 2.2k stars  
**Layer:** 4 — Workflow Orchestration  
**WISC:** I + S + W (isolate by task; select per-dispatch; write structured state files)

---

## The Problem It Solves

Prompt-based workflow frameworks ask the LLM to manage its own context — read these files, follow these phases, don't forget this rule. That works until it doesn't. Over a long session, quality degrades: the agent accumulates garbage context, forgets earlier decisions, and burns tokens on orchestration overhead.

GSD-2's core insight: **a TypeScript application can do what a prompt can only ask**. It controls the agent harness directly — clearing context between tasks, injecting exactly the right content at dispatch time, managing git branches, tracking cost, recovering from crashes.

---

## Architecture: The Dispatch Loop

GSD structures work into a strict hierarchy:

```
Milestone  →  a shippable version (4–10 slices)
  Slice    →  one demoable vertical capability (1–7 tasks)
    Task   →  one context-window-sized unit of work
```

The iron rule: **a task must fit in one context window**. If it can't, it's two tasks.

### Fresh context per unit

Every task, every research phase, every planning step gets a **clean 200k-token context window**. No accumulated garbage. No context rot from earlier in the session.

### Pre-inlined dispatch prompt

When GSD dispatches a task, it constructs the context prompt by reading from disk and inlining:

| Content | Source file |
|---------|-------------|
| Current task plan | `T01-PLAN.md` |
| Parent slice plan | `S01-PLAN.md` |
| Prior task summaries | `T*-SUMMARY.md` |
| Dependency summaries | Adjacent task summaries |
| Roadmap excerpt | `M001-ROADMAP.md` |
| Architectural decisions | `DECISIONS.md` |
| Project state | `STATE.md` |

The LLM receives all of this pre-loaded — it doesn't spend tool calls on orientation.

---

## The .gsd/ File Hierarchy

State lives on disk. Auto mode reads it, writes it, and advances based on what it finds. No in-memory state survives across sessions.

```
.gsd/
├── STATE.md              ← quick-glance dashboard; always read first
├── DECISIONS.md          ← append-only architectural decisions register
├── PROJECT.md            ← living doc: what the project is right now
│
├── M001-ROADMAP.md       ← milestone slice checklist + risk levels
├── M001-CONTEXT.md       ← decisions from discuss phase
├── M001-RESEARCH.md      ← codebase + ecosystem research
│
├── milestones/
│   └── M001/
│       ├── S01-PLAN.md   ← slice task decomposition with must-haves
│       ├── S01-UAT.md    ← human test script derived from slice outcomes
│       ├── T01-PLAN.md   ← individual task plan with verification criteria
│       └── T01-SUMMARY.md← what happened: YAML frontmatter + narrative
│
└── preferences.md        ← per-project: models, timeouts, budget ceiling
```

This is a concrete production implementation of the [File Memory Structure](../../patterns/file-memory-structure.md) pattern.

---

## Context Engineering Features

### Per-phase model routing

```yaml
# .gsd/preferences.md
models:
  research: claude-haiku-3-5      # fast, cheap
  planning: claude-opus-4-6       # highest quality
  execution: claude-sonnet-4-6    # balanced
  completion: claude-sonnet-4-6
```

Different phases use different models. Research doesn't need opus-quality reasoning. Planning does.

### Crash recovery

A lock file tracks the current unit. If the session dies, the next run reads the surviving session file, synthesizes a recovery briefing from every tool call that made it to disk, and resumes with full context. No starting over.

### Verification enforcement

```yaml
verification_commands:
  - npm run lint
  - npm run test
verification_auto_fix: true
verification_max_retries: 2
```

Commands run automatically after task execution. Failures trigger auto-fix retries before advancing.

---

## GSD-2 vs. Other Layer 4 Workflow Tools

| Dimension | GSD-2 | Cursor Memory Bank | ECC |
|-----------|-------|--------------------|-----|
| Implementation | TypeScript application | Markdown prompts | Markdown prompts + scripts |
| Context isolation | Hard (fresh process per task) | Soft (agent instructed to unload) | Soft (agent instructed to scope) |
| Phase control | State machine reading .gsd/ files | Agent reads phase rules | Agent reads CLAUDE.md hub |
| Crash recovery | ✅ Lock files + session forensics | ❌ | ❌ |
| Cost tracking | ✅ Per-unit token/cost ledger | ❌ | ❌ |
| Git integration | ✅ Worktree isolation per milestone | ❌ | ❌ |
| Best for | Autonomous long-running agent sessions | Developer-driven phased workflows | Skills-based agent harness systems |

---

## In This Repo's Architecture

GSD-2 sits in **Layer 4 — Workflow Orchestration** as a production reference implementation of multiple patterns in this repo:

- **[Sub-Agent Isolation](../../patterns/sub-agent-isolation.md)** — fresh context window per task is the strongest enforcement of this pattern
- **[Phase-Gated Context](../../patterns/phase-gated-context.md)** — the dispatch-based variant: disk state determines phase, not agent instruction
- **[File Memory Structure](../../patterns/file-memory-structure.md)** — the `.gsd/` hierarchy is a concrete, battle-tested instantiation

**Related patterns:** [Phase-Gated Context](../../patterns/phase-gated-context.md), [Sub-Agent Isolation](../../patterns/sub-agent-isolation.md), [File Memory Structure](../../patterns/file-memory-structure.md)
