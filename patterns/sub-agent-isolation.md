# Pattern: Sub-Agent Isolation

> One agent, one job, one context scope. Break complex work into focused sub-agents, each with only the context it needs. Agents that see less make fewer mistakes.

**WISC dimension:** I (Isolate)  
**Validated by:** [ECC](../tools/workflow/ecc.md) (25 named sub-agents), [Cursor Memory Bank](../tools/workflow/memory-bank.md) (phase isolation)

---

## The Problem

A single omniscient agent with all context is an anti-pattern. Give an agent everything and it:

- Gets distracted by information irrelevant to the current task
- Makes cascading errors (wrong assumption in task 1 propagates to tasks 2–5)
- Produces inconsistent outputs because its context is contradictory
- Burns tokens on context that serves no purpose for this specific task
- Is difficult to debug when something goes wrong

Sub-agent isolation is the practice of decomposing work into bounded tasks, each handled by an agent with a scoped context load.

---

## The Core Principle

```
Correct scope = minimum context needed to do the job well

Not: "give the agent everything and hope it ignores what's irrelevant"
But: "give the agent only what's relevant and nothing else"
```

---

## Sub-Agent Design

A sub-agent is defined by three things:

1. **Name** — what it is (Planner, Implementer, Reviewer, Debugger)
2. **Task** — the specific job it does
3. **Context scope** — exactly what it loads, no more

### Example: Backend Implementation Sub-Agent

```
Name: BackendImplementer
Task: Write or modify backend code in a specific service
Context loads:
  - projectBrief.md (tech stack, architecture)
  - .claude/rules/backend.md (domain rules)
  - The specific service/controller files being modified
  - No frontend files
  - No migration history  
  - No unrelated test files
  - No payment rules (unless this task is payment-specific)
Output: Modified files with test updates
```

### Example: Planning Sub-Agent

```
Name: Planner  
Task: Analyze requirements and produce an implementation plan
Context loads:
  - projectBrief.md
  - architecture docs
  - The requirement or bug report
  - No implementation files (not writing code yet)
  - No domain rules (evaluation mode, not implementation mode)
Output: A written implementation plan listing: files to change, approach, risks
```

### Example: Security Review Sub-Agent

```
Name: SecurityAuditor
Task: Review a diff or specific files for security vulnerabilities
Context loads:
  - Security checklist (OWASP Top 10 relevant items)
  - The diff or files under review
  - Auth middleware for context on permissions model
  - No unrelated business logic
Output: List of findings with severity + suggested fixes
```

---

## Isolation in Practice (GitHub Copilot)

GitHub Copilot doesn't have named sub-agents out of the box, but the pattern is achievable:

**Step 1: Start a fresh chat for each agent**  
Each new chat = a clean context. Don't carry a planning conversation into an implementation conversation.

**Step 2: Load only the relevant files**  
Use `#file:` references to be explicit about what you're including:
```
#file:memory-bank/projectBrief.md
#file:.claude/rules/backend.md
#file:backend/src/services/payment.service.ts

Implement the collectionMode field support in the payment service.
```

**Step 3: Pass outputs, not contexts**  
When moving from Planner to Implementer, pass the written plan — not the entire planning conversation. The implementer needs the *output* of planning, not the *discussion*.

```
# Planning chat output (the plan document):
"Implementation plan:
1. Add collectionMode to Payment model in schema.prisma
2. Run prisma migrate dev
3. Update payment.service.ts: initiatePayment() to accept and store collectionMode
4. Update payment routes: pass collectionMode from request body"

# New implementation chat:
#file:.claude/rules/backend.md
#file:backend/prisma/schema.prisma
#file:backend/src/services/payment.service.ts

Implement this plan: [paste the plan]
```

---

## Common Sub-Agent Roles

| Agent | Task | Context Scope |
|-------|------|---------------|
| **Planner** | Analyze + design | Requirements + architecture docs |
| **Researcher** | Investigate codebase | Read-only: specific files + search |
| **BackendImplementer** | Write backend code | Backend rules + relevant src files |
| **FrontendImplementer** | Write frontend code | Frontend rules + relevant src files |
| **MigrationAgent** | Database change | Schema + migration rules only |
| **TestWriter** | Write test cases | Test patterns + files being tested |
| **Reviewer** | Review diff | Diff + quality checklist |
| **Debugger** | Fix a specific bug | Error + reproducing code + relevant files |
| **SecurityAuditor** | Security review | Security checklist + files under review |

---

## When Not to Use Sub-Agents

Sub-agent isolation adds coordination overhead. Skip it for:
- Single-file changes with no cross-cutting concerns
- Quick bug fixes with obvious solutions
- Documentation updates

Use it for:
- Features touching 3+ files or services
- Any change to auth, payment, or security code
- Database migrations
- Anything you'd want reviewed before merging

---

## Related Patterns

- [Phase-Gated Context](phase-gated-context.md) — phases as a lightweight form of sub-agent isolation
- [Progressive Loading](progressive-loading.md) — how each sub-agent loads its tier
- [File Memory Structure](file-memory-structure.md) — how to structure files so sub-agents can load precisely what they need
