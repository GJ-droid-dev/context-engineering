# Pattern: File Memory Structure

> Design your memory files as a structured hierarchy — each file has a clear purpose, a defined scope, and a known update frequency. Every file the agent reads is intentional. Nothing loads by accident.

**WISC dimension:** W (Write — structural design of the W layer)  
**Validated by:** [ECC](../tools/workflow/ecc.md) (CLAUDE.md hub), [Cursor Memory Bank](../tools/workflow/memory-bank.md) (memory-bank/ directory), [ConPort](../tools/memory/conport.md), [Basic Memory](../tools/memory/basic-memory.md)

---

## The Problem

Unstructured memory is as bad as no memory. If you have 15 markdown files that all contain mixed concerns — rules alongside current task status alongside architecture docs alongside one-off experiments — the agent loads them all and gets confused. Worse, it can't tell which information is current and which is stale.

File memory structure is the discipline of organizing your memory files deliberately: one concern per file, one file per concern, one purpose per file.

---

## The Canonical File Structure

This structure is synthesized from ECC, Memory Bank, ConPort, and Basic Memory conventions:

```
project-root/
│
├── CLAUDE.md                      ← AI hub: routing table + key rules summary
│                                     (always loaded; links to everything else)
│
├── memory-bank/                   ← stable project knowledge
│   ├── projectBrief.md            ← tech stack, architecture, stable facts
│   ├── systemPatterns.md          ← recurring patterns in this codebase
│   ├── activeContext.md           ← current work state (updated frequently)
│   └── progress.md                ← task history + what's next
│
├── .claude/                       ← agent configuration
│   ├── skills/                    ← skills-as-files (load on demand)
│   │   ├── backend-patterns/SKILL.md
│   │   ├── api-design/SKILL.md
│   │   └── [domain]/SKILL.md
│   └── rules/                     ← phase/domain-specific rules
│       ├── backend.md
│       ├── frontend.md
│       └── payments.md
│
├── tasks/                         ← session-scoped work tracking
│   ├── todo.md                    ← current tasks + status
│   ├── lessons.md                 ← accumulated discoveries (bidirectional memory output)
│   └── HANDOFF.md                 ← end-of-session summary for next session
│
└── .conport/ or ./memory/         ← tool-specific stores (optional)
    └── [ConPort or Basic Memory managed files]
```

---

## File Responsibilities

### CLAUDE.md — The Hub

The entry point for the agent. Never loads the full project context — it's a routing table that tells the agent *where* to find context for a specific need.

```markdown
# CLAUDE.md

## What This Is
[1-paragraph product summary]

## Tech Stack
[bullet list, ~10 lines]

## When You Need More Context
- Backend patterns → .claude/skills/backend-patterns/SKILL.md
- Payment logic → .claude/rules/payments.md  
- Architecture decisions → memory-bank/projectBrief.md

## Key Rules (always apply)
- [3-5 non-negotiable rules, max 5 lines each]
```

**Load frequency:** Every session, always. Keep it under 300 tokens.

---

### projectBrief.md — Stable Facts

The immutable foundation. Everything in here should be true today, in a month, and at project end.

**Load frequency:** Every session, always. Target 200–400 tokens.  
**Update frequency:** Only on major architectural change.

---

### systemPatterns.md — Codebase Conventions

How things are done in *this* codebase. Not general best practices — project-specific established patterns.

```markdown
# systemPatterns.md

## Error Handling
All controllers use asyncHandler wrapper. Never res.status().json() directly.
Throw: new AppError('message', statusCode)

## API Response Format
import { sendSuccess, sendCreated, sendPaginated } from '@/utils/response'

## Database Access
Never spread Prisma results. Always destructure explicit fields.
✅ const { id, email, role } = user
❌ res.json({ ...user })
```

**Load frequency:** Most sessions. Trigger: working on backend code.  
**Update frequency:** When a new pattern becomes established.

---

### activeContext.md — Current Work State

The living document of what's happening right now. The most frequently updated file.

```markdown
# activeContext.md

## Current Focus
[One sentence: what are we building?]

## In Progress
[Bullet list of active tasks]

## Recent Decisions
[Decisions made in the last 48 hours — with rationale]

## Blockers
[Anything preventing progress]

## Immediate Next Steps
[Ordered list, maximum 5 items]
```

**Load frequency:** Every session, always.  
**Update frequency:** After every significant task.

---

### lessons.md — Accumulated Knowledge

The output of [bidirectional memory](bidirectional-memory.md). Discoveries, gotchas, resolved bugs — the project's institutional knowledge.

```markdown
# lessons.md

## [date]: [brief title]
[What was discovered. Why it matters. Where in the code it applies.]
```

**Load frequency:** Selective — when working in a domain related to past lessons.  
**Update frequency:** After every non-trivial bug fix or discovery.

---

### HANDOFF.md — Session Continuity

Written at session end. Read at session start. Eliminates re-orientation overhead.

**Load frequency:** First thing every session.  
**Update frequency:** Every session end.

---

## File Size Guidelines

| File | Target Size | Max |
|------|-------------|-----|
| CLAUDE.md | 300 tokens | 500 |
| projectBrief.md | 400 tokens | 600 |
| systemPatterns.md | 400 tokens | 700 |
| activeContext.md | 200 tokens | 400 |
| HANDOFF.md | 300 tokens | 500 |
| lessons.md | grows over time | no hard limit, load selectively |
| SKILL.md files | 500 tokens | 800 |
| domain rule files | 300 tokens | 500 |

Files that exceed these limits should be split, not padded.

---

## Related Patterns

- [Project Brief Bootstrap](project-brief-bootstrap.md) — what goes in projectBrief.md
- [Progressive Loading](progressive-loading.md) — how to load these files efficiently
- [Bidirectional Memory](bidirectional-memory.md) — how lessons.md gets populated
- [Context Pruning](context-pruning.md) — enforcing the size caps above; rewrite-on-update discipline; git as archive
