# Pattern: Progressive Loading

> Load context in layers, not dumps. Start with stable universal context, then progressively add task-specific context as the work narrows. Unload what's no longer needed.

**WISC dimension:** S + C (Select progressively; Compress by not loading)  
**Validated by:** [Cursor Memory Bank](../tools/workflow/memory-bank.md) (~70% token reduction), [ECC](../tools/workflow/ecc.md) (skills-as-files), [RTK](../tools/compression/rtk.md)

---

## The Problem

Eager loading — loading everything at session start — is the default behavior of most agent setups. It wastes tokens, pollutes the context with irrelevant noise, and hits token limits before the actual work begins.

The two failure modes:
1. **Load too little:** Agent hallucinates because it lacks grounding
2. **Load too much:** Agent gets confused by irrelevant context; costs balloon

Progressive loading is the middle path: load the minimum needed, then expand as the task narrows.

---

## The Pattern

Context loads in three tiers, from always-loaded to task-specific:

```
Tier 1 — Universal (always loaded, small)
  └─ projectBrief.md    ~200 tokens
  └─ activeContext.md   ~150 tokens

Tier 2 — Domain (loaded when entering a domain)
  └─ backend-rules.md   ~400 tokens  ← loaded when working in backend/
  └─ frontend-rules.md  ~300 tokens  ← loaded when working in frontend/
  └─ payment-rules.md   ~250 tokens  ← loaded when working on payments

Tier 3 — Task (loaded for this specific task only)
  └─ The actual file(s) being modified
  └─ Relevant past decision from ConPort
  └─ Current library docs from Context7
```

Each tier loads only when its context becomes relevant. Tier 3 content is often unloaded between tasks.

---

## Implementation

### Step 1: Define your tier structure

Write one file per domain, each under 500 tokens:

```
.context/
├── universal/
│   ├── projectBrief.md      ← always load
│   └── activeContext.md     ← always load
├── domain/
│   ├── backend.md           ← load when editing backend/
│   ├── frontend.md          ← load when editing frontend/
│   ├── payments.md          ← load when editing payment files
│   └── database.md          ← load when editing schema or migrations
└── task/                    ← transient; cleared between tasks
    └── currentTask.md
```

### Step 2: Define load triggers

```markdown
# In your system prompt / CLAUDE.md

## Context Loading Rules
- ALWAYS load: .context/universal/projectBrief.md + activeContext.md
- Load .context/domain/backend.md when: editing files in backend/src/
- Load .context/domain/frontend.md when: editing files in frontend/src/
- Load .context/domain/payments.md when: editing files matching *payment*, *settlement*, *payout*
- Load .context/domain/database.md when: editing prisma/ or running migrations
```

### Step 3: Progressive narrowing

```
Task: "Add a new payment method to the checkout flow"

Load: universal + domain/payments.md + domain/frontend.md
Add: the specific component file + the payment service file
Add: Context7 docs for the payment library being used
Remove: domain/frontend.md after implementation is done
Add: domain/backend.md when writing the API endpoint
```

---

## The Unload Step

Most developers forget to unload. After a task is complete:

1. Clear or archive `task/currentTask.md`
2. Remove domain files that are no longer relevant
3. Update `activeContext.md` with the new state

Failing to unload means the next task inherits noise from the previous one.

---

## Token Budget Framing

Think of the context window as a budget, not a dump:

```
Total context window: 128,000 tokens

Tier 1 (universal):     ~500 tokens (0.4%)
Tier 2 (domain):       ~600 tokens (0.5%)
Tier 3 (task files):   ~8,000 tokens (6%)
Tier 3 (docs):         ~3,000 tokens (2%)
Available for output:  ~116,000 tokens (91%)
```

Compare to eager loading where you spend 20,000+ tokens on context before the first line of code.

---

## Evidence

**Cursor Memory Bank** demonstrated ~70% token reduction by moving from eager to phase-gated loading. The same principle applied to any project's domain files will produce proportional savings.

---

## Related Patterns

- [Phase-Gated Context](phase-gated-context.md) — explicit phase boundaries for load/unload
- [Project Brief Bootstrap](project-brief-bootstrap.md) — what goes in Tier 1
- [Sub-Agent Isolation](sub-agent-isolation.md) — each agent has its own tier budget
