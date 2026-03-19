# Pattern: Phase-Gated Context

> Divide work into named phases. Each phase has a defined context set that loads when the phase starts and unloads (or shrinks) when the phase ends. Agents never have context from phases they haven't entered.

**WISC dimension:** I + S (Isolate by phase; Select per phase)  
**Validated by:** [Cursor Memory Bank](../tools/workflow/memory-bank.md) (VAN→PLAN→CREATIVE→BUILD→REFLECT→ARCHIVE)

---

## The Problem

Without phases, agents drift. A discovery conversation bleeds into implementation. An implementation task detours into refactoring. Context from "we were evaluating options" contaminates "we are now writing code." The agent loses track of what mode it's in.

Phase gates are explicit checkpoints. The phase transition is a context swap — out with the old, in with the new. The agent knows exactly what mode it's in because the context it has proves it.

---

## Core Concept

```
Phase = a named mode of work + a defined context set + a clear exit condition
```

A task moves through phases sequentially. Each phase:
- Has a named identity (PLAN, BUILD, REVIEW, etc.)
- Defines which files and rules load when it activates
- Has a clear exit condition (e.g., "a written implementation plan is approved")
- Swaps context when it exits (unloads current phase files, loads next phase files)

---

## A Minimal 3-Phase System

Start here before going to 6 phases:

### Phase 1: PLAN
```
Context loaded:
  - projectBrief.md
  - activeContext.md
  - architecture docs relevant to the task
  - [NO implementation rules yet]

Exit condition:
  - Written plan with specific files to change, approach chosen
  - Approved by developer before moving to BUILD
```

### Phase 2: BUILD
```
Context loaded:
  - projectBrief.md (keep)
  - domain rules for files being modified
  - ONLY the specific files being changed
  - Context7 docs for relevant libraries
  - [NO discovery docs from PLAN — they're done]

Exit condition:
  - Implementation complete, compiles, passes lint
```

### Phase 3: REVIEW
```
Context loaded:
  - The diff of changes made
  - Test results
  - Quality checklist
  - [NO implementation files — the review is about the diff, not the code]

Exit condition:
  - All checklist items passed or explicitly deferred
```

---

## Phase Transition Protocol

At every phase boundary, the agent announces the transition and gets confirmation before proceeding:

```
Agent:  "PLAN phase complete. Implementation plan:
         - Add collectionMode field to schema
         - Run prisma migrate dev
         - Update payment.service.ts lines 45–78
         - Update payment routes to pass collectionMode
         
         Ready to enter BUILD phase? [y/n]"

Developer: "y"

Agent: [loads BUILD context, begins implementation]
```

This explicit handoff prevents drift and gives the developer a checkpoint to redirect if the plan is wrong.

---

## The 6-Phase System (Memory Bank)

The [Cursor Memory Bank](../tools/workflow/memory-bank.md) implements the full 6-phase version:

| Phase | Context Loaded | Exit Condition |
|-------|---------------|----------------|
| VAN | Complexity rules + task definition | Complexity level assigned (1–4) |
| PLAN | Architecture + relevant patterns | Written implementation plan approved |
| CREATIVE | Design constraints + alternatives | Approach chosen + rationale documented |
| BUILD | Domain rules + active files only | Implementation complete, tests pass |
| REFLECT | Test results + quality checklist | All items resolved or deferred |
| ARCHIVE | Memory bank templates | Summary written, files updated |

For Complexity Level 1 tasks, only BUILD is needed. For Level 4, all 6 phases run.

---

## Implementing Phase Gates in Any Editor

You don't need special tooling. A simple convention works:

**In your system prompt / CLAUDE.md:**
```markdown
## Phase Protocol

Work happens in phases. Before starting any task:
1. Declare the phase: "Entering PLAN phase for [task]"
2. Load only the phase-appropriate context
3. At phase exit, explicitly announce transition and get approval

Never enter BUILD phase without an approved PLAN. 
Never enter REVIEW phase without completion confirmed in BUILD.
```

**Phase-specific rule files** (one per phase):
```
.context/phases/
├── plan.md    ← rules for planning (architecture, tradeoffs)
├── build.md   ← rules for implementation (conventions, patterns)
└── review.md  ← rules for review (checklist, quality gates)
```

---

## When to Use Phases

**Always use phases for:**
- Features touching more than 3 files
- Any database schema change
- Changes to authentication or authorization logic
- Anything that could break existing tests

**Optional for:**
- Single-file bug fixes (just BUILD)
- Documentation updates (just BUILD)
- Simple copy/config changes (just BUILD)

---

## Dispatch-Based Phase Gating (GSD-2 variant)

The phase systems above use agent instructions to enforce phase boundaries — the agent is told to load only phase-appropriate context. A harder enforcement mechanism: **fresh context window per task**, where the phase boundary is a process boundary.

[GSD-2](../tools/workflow/gsd.md) implements this: every task, research step, and planning phase runs in a completely fresh 200k-token context window. The previous session is gone. Phase transition is not an instruction — it's a restart.

```
Instructions-based (soft):           Dispatch-based (hard):

Phase 1: PLAN                        Phase 1: PLAN
  Load: plan rules                     New process → fresh context
  Agent: "I'll unload these..."        Inject: plan rules + task only
  [context accumulates]                Execute → write S01-PLAN.md to disk
                                       Process ends
Phase 2: BUILD                       Phase 2: BUILD
  Unload: plan rules (maybe)           New process → fresh context
  Load: build rules                    Read S01-PLAN.md from disk
  [leftover plan context leaks in]     Inject: build rules + plan file
                                       Execute → write T01-SUMMARY.md
```

The state that carries between phases is the **written artifact** — `S01-PLAN.md`, `T01-SUMMARY.md`, `DECISIONS.md` — not the conversation. This makes phase isolation guaranteed rather than instructed.

**When this matters:**
- Long-running autonomous sessions (hours, not minutes)
- Multi-agent workflows where different models handle different phases
- Any scenario where you've seen context rot degrade quality across tasks

**Trade-off:** Harder to implement (requires a runtime, not just prompts). Start with instruction-based phase gating and move to dispatch-based only when soft isolation proves insufficient.

---

## Related Patterns

- [Progressive Loading](progressive-loading.md) — the token budget effect of phase gating
- [Sub-Agent Isolation](sub-agent-isolation.md) — using separate agents per phase
- [Project Brief Bootstrap](project-brief-bootstrap.md) — what's in the universal (every-phase) context
