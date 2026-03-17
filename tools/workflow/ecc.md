# ECC (Everything Claude Code)

> A full agent harness system: 108 skills, 25 sub-agents, 57 commands, and a CLAUDE.md hub pattern. The most comprehensive open-source implementation of skills-as-files and structured agent orchestration.

**Source:** [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) — 82.6k stars  
**Layer:** 4 — Workflow Orchestration  
**WISC:** I + W (sub-agent isolation; authored context files)

---

## The Problem It Solves

Ad-hoc agent usage produces ad-hoc results. When developers just "chat with the AI," they get inconsistent outputs, recreated context every session, and no systematic way to build on past work. ECC is the opposite: a structured harness where every capability is a named, versioned, loadable file.

---

## Core Concepts

### Skills (108 total)
Reusable, domain-specific instruction sets stored as individual markdown files. A skill is loaded when needed, not by default.

```
.claude/skills/
├── backend-patterns/SKILL.md     # Express, Prisma, TypeScript patterns
├── api-design/SKILL.md           # REST conventions, response envelopes
├── database-migrations/SKILL.md  # Safe schema evolution steps
├── session-handoff/SKILL.md      # How to write end-of-session handoff
└── verification-loop/SKILL.md    # Pre-commit verification checklist
```

The developer tells the agent: "Load the backend-patterns skill." The agent reads that file and applies its contents. Skills are **loaded by task relevance, not loaded by default**.

### Sub-Agents (25 named)
Named agents with defined context scopes and responsibilities:

```
Explore          — read-only codebase search and Q&A
Architect        — design decisions, tradeoffs, system design
Implementer      — code writing, only loads relevant domain files
Reviewer         — diff analysis, quality check
Debugger         — error analysis, root cause, fix generation
SecurityAuditor  — OWASP checks, permission gaps
```

Each sub-agent is invoked with a specific task and a specific context load. They don't share context with each other unless explicitly passed.

### CLAUDE.md Hub
The central project context file that acts as a routing table:

```markdown
# CLAUDE.md

## Tech Stack
- Backend: Node.js/Express/TypeScript/Prisma/PostgreSQL
- Frontend: Next.js 14, MUI, Redux

## Key File Locations
- Schema: backend/prisma/schema.prisma
- Auth middleware: backend/src/middlewares/auth.ts
- Error handling: backend/src/utils/errors.ts

## Skills (load when needed)
- Backend patterns: `.claude/skills/backend-patterns/SKILL.md`
- API design: `.claude/skills/api-design/SKILL.md`

## Rules
- Always use asyncHandler wrapper on controllers
- Never spread Prisma results — explicit field allowlists only
```

CLAUDE.md is not loaded in full every time — it's the index. The agent reads the index and loads only the section it needs.

---

## The Skills-as-Files Pattern

The key insight from ECC: **capture domain knowledge in files, not in prompts**.

Instead of re-typing "always use asyncHandler, always sanitize inputs, never spread Prisma results" at the start of every backend conversation, you write it once in `backend-patterns/SKILL.md`. Then:

```
# In your prompt:
"Load the backend-patterns skill. Now implement the payment endpoint."

# The agent reads the skill file, internalizes the patterns, implements accordingly.
```

Benefits:
- Knowledge is versioned (git history)
- Knowledge is shareable (commit the `.claude/` directory)
- Knowledge compounds — each skill improves over time as you discover new patterns
- Context is earned — skills load on demand, not as overhead

---

## Session Handoff Pattern

ECC formalizes the problem of session continuity with a dedicated `session-handoff` skill:

```markdown
# HANDOFF.md (written at session end)
## What Was Accomplished
- Implemented Razorpay webhook verification
- Fixed the auth middleware bug (missing authorize() call)

## Current State
- Payment endpoint is working end-to-end
- Tests passing: 212/212

## Next Steps
1. Wire the WhatsApp notification service into payment dispatch
2. Address GAP-23: chargeback endpoints

## Key Decisions Made This Session
- Using raw body middleware only on /webhooks routes (not global)
```

The next session starts by reading HANDOFF.md. Zero re-orientation overhead.

---

## Adapting ECC for GitHub Copilot

ECC was designed for Claude Code CLI but the patterns transfer directly:

| ECC Pattern | GitHub Copilot Equivalent |
|-------------|--------------------------|
| CLAUDE.md hub | `.github/copilot-instructions.md` or `CLAUDE.md` |
| `.claude/skills/` | `.github/instructions/` or `.claude/skills/` |
| Sub-agent invocation | Chat with specific files mentioned (`#file:`) |
| Session handoff | `HANDOFF.md` file in project root |

---

## In This Repo's Architecture

ECC provides the richest implementation of Layer 4 orchestration patterns. The skills-as-files pattern from ECC is directly applicable in any editor. The CLAUDE.md hub pattern works in Copilot, Cursor, and Cline alike. Sub-agent isolation is the W+I dimensions of WISC made concrete.

**Relevant patterns:** [Sub-Agent Isolation](../../patterns/sub-agent-isolation.md), [File Memory Structure](../../patterns/file-memory-structure.md), [Progressive Loading](../../patterns/progressive-loading.md)
