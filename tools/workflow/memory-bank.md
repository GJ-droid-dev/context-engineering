# Cursor Memory Bank

> A 6-phase workflow system that delivers ~70% token reduction through hierarchical context loading. Context is loaded by phase, not by default — agents only get what the current phase requires.

**Source:** [vanzan01/cursor-memory-bank](https://github.com/vanzan01/cursor-memory-bank) — 3k stars  
**Layer:** 4 — Workflow Orchestration  
**WISC:** I + S (isolate by phase; select files per phase)

---

## The Problem It Solves

Loading context eagerly — dumping all rules, all docs, all history at session start — is a trap. It's expensive, noisy, and fills the window before any work begins. The Memory Bank system flips this: context is earned by phase. An agent in the PLAN phase doesn't load implementation rules. An agent in the BUILD phase doesn't load discovery rules.

**Measured result:** Switching from eager loading to hierarchical phase-based loading produced ~70% token reduction in practice.

---

## The 6 Phases

```
VAN → PLAN → CREATIVE → BUILD → REFLECT → ARCHIVE
```

| Phase | Purpose | Context Loaded |
|-------|---------|----------------|
| **VAN** | Validate task, assess complexity | Complexity rules + task definition only |
| **PLAN** | Detailed implementation planning | Architecture docs + relevant patterns |
| **CREATIVE** | Design decisions, approach selection | Design constraints + alternatives framework |
| **BUILD** | Implementation | Domain rules + active file context only |
| **REFLECT** | Review and test | Test patterns + quality checklist |
| **ARCHIVE** | Document and close | Memory bank templates + summary format |

---

## Complexity Levels

Not every task runs all 6 phases. The system defines 4 complexity levels:

| Level | Description | Phases Required |
|-------|-------------|-----------------|
| 1 | Quick fix, obvious solution | BUILD only |
| 2 | Small feature, clear approach | PLAN → BUILD → REFLECT |
| 3 | Medium feature, some unknowns | VAN → PLAN → BUILD → REFLECT → ARCHIVE |
| 4 | Complex/architectural change | All 6 phases |

**VAN phase responsibility:** Assess the task and assign a complexity level before any other work begins.

---

## File Structure

```
memory-bank/
├── projectBrief.md          ← stable project context (W layer)
├── activeContext.md         ← what we're doing right now
├── progress.md              ← task tracking and history
├── systemPatterns.md        ← recurring patterns in this codebase
│
└── rules/                   ← phase-specific rule files (loaded lazily)
    ├── van-rules.md
    ├── plan-rules.md
    ├── creative-rules.md
    ├── build-rules.md
    ├── reflect-rules.md
    └── archive-rules.md
```

The key is that **rules files are not loaded at session start**. Each phase loads only its own rule file.

---

## The Hierarchical Loading Pattern

```
Session Start:
  └─ Load: projectBrief.md + activeContext.md (always)
     └─ ~400 tokens

VAN Phase:
  └─ Load: van-rules.md + task description
     └─ +200 tokens (now ~600 total)

PLAN Phase:
  └─ Unload: van-rules.md
  └─ Load: plan-rules.md + relevant architecture docs
     └─ +300 tokens (still ~700 total)

BUILD Phase:
  └─ Unload: plan-rules.md
  └─ Load: build-rules.md + ONLY the files being modified
     └─ +variable tokens for actual code
```

Compare to eager loading where all rules + all docs load upfront (~2500 tokens before any work).

---

## Implementation (adapted for any editor)

The Memory Bank pattern is editor-agnostic. The core implementation is:

1. Write stable context to `projectBrief.md` and `systemPatterns.md` (W layer)
2. Maintain `activeContext.md` as a living document of current work (W layer)
3. Break your system prompt into phase files in `rules/` (S layer)
4. Instruct the agent: "Start in VAN phase. Load only van-rules.md."
5. Agent announces phase transitions; you confirm and the agent loads the next rule file

---

## Cursor-Specific Setup

The source repo provides ready-made rule files for Cursor (`.cursor/rules/`). Each rule file ends with an instruction to load the next phase's file on phase transition.

---

## In This Repo's Architecture

Memory Bank is the most concrete implementation of the **phase-gated context** and **progressive loading** patterns. The ~70% token reduction figure comes from this system, and it's the primary evidence that Layer 4 orchestration is worth the setup cost.

**Relevant patterns:** [Phase-Gated Context](../../patterns/phase-gated-context.md), [Progressive Loading](../../patterns/progressive-loading.md)
