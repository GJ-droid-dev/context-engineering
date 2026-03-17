# Pattern: Project Brief Bootstrap

> Author the stable project context once, in a structured file, so every new session starts with a fully oriented agent — not a blank slate.

**WISC dimension:** W (Write)  
**Validated by:** [ECC](../tools/workflow/ecc.md), [Cursor Memory Bank](../tools/workflow/memory-bank.md), [Context Hub](../tools/docs/context-hub.md)

---

## The Problem

The first 15 minutes of every new AI session are wasted re-explaining the same things:
- "We're using TypeScript, not JavaScript"
- "The backend is Express with Prisma"
- "We never spread Prisma results"
- "The payment service uses Razorpay"

Multiply this by 200 working sessions and you've paid for hundreds of hours of re-orientation — in your time, in tokens, and in re-established context that's only as accurate as your memory in that moment.

---

## The Pattern

Write a structured `projectBrief.md` (or equivalent) at project start. Update it as the project evolves. Start every session by loading it.

```markdown
# projectBrief.md

## What This Is
[One paragraph. What the product does. Who uses it.]

## Tech Stack
- Backend: Node.js 20, Express, TypeScript, Prisma ORM
- Database: PostgreSQL 16
- Frontend: Next.js 14, MUI v6, Redux Toolkit
- Queues: BullMQ + Redis (Upstash)
- Hosting: Railway (backend + frontend), Cloudflare R2 (storage)

## Architecture Decisions
- API-first: backend exposes REST API; frontend is a separate Next.js app
- Auth: JWT access tokens (15min) + refresh tokens (7 days)
- All controllers wrapped in asyncHandler — never res.json() directly
- Payments: Razorpay for collection, Cashfree for domestic payouts

## What We Never Do
- Never spread Prisma results (`...query`) — always explicit field picks
- Never hardcode hex colors in frontend — always theme.palette.*
- Never add direct npm test without --no-coverage (slow)

## Key File Locations
backend/prisma/schema.prisma         ← single source of truth for data models
backend/src/middlewares/auth.ts      ← authenticate() + authorize() middleware
frontend/src/theme/                  ← theme tokens
```

---

## Where to Put It

| Editor | Recommended location |
|--------|---------------------|
| GitHub Copilot | `projectBrief.md` in project root, referenced in `.github/copilot-instructions.md` |
| Cursor | `.cursor/projectBrief.md` or project root |
| Cline | `.clinerules/projectBrief.md` |
| All | `memory-bank/projectBrief.md` (Memory Bank convention) |

---

## What Good Project Brief Content Looks Like

**Good — stable facts that rarely change:**
- Tech stack and versions
- Architecture decisions and rationale
- Patterns that must be followed
- Patterns that must be avoided
- Key file locations
- Current product status (MVP / production / v2)

**Bad — volatile content:**
- Current task description (use `activeContext.md` for this)
- Error messages from today
- Experimental approaches being evaluated
- Team member names and responsibilities

---

## The Companion: activeContext.md

Project brief = what never changes.  
Active context = what's true right now.

```markdown
# activeContext.md

## Current Sprint Focus
Implementing the payment settlement flow (Section 6 of the workflow spec).

## Current Task
- Building the daily settlement runner (scheduled via BullMQ)
- Schema gap: need to add Payment.collectionMode field before this works

## Most Recent Decision
Decided to hardcode markupPct = 0 until schema migration is done.
Do not refactor until migration is merged.

## Next Steps
1. Add collectionMode field to schema + run migration
2. Wire settlement runner into the queues config
3. Update tests to cover settlement edge cases
```

---

## Session Start Checklist

```
1. Load projectBrief.md      → stable orientation
2. Load activeContext.md     → what we're doing today
3. Load tasks.md or todo.md  → task state
4. Query ConPort/ICM         → recent decisions + session memory
```

This four-file load typically costs 300–600 tokens and gives the agent everything it needs to start immediately.

---

## Updating the Brief

Add an update to `projectBrief.md` when:
- A major architectural decision is made
- The tech stack changes
- A new pattern is established as a project standard
- A previously-used approach is now banned

Never let the brief go more than 2 weeks without a review.
