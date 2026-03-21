# Pattern: Context Pruning

> Living context files are state snapshots, not logs. Before writing, rewrite. Git is the archive.

**WISC dimension:** W (constrains the W layer — shapes how often and how much is written back)  
**Validated by:** [File Memory Structure](file-memory-structure.md) (defines file purposes and size caps), [Reflective Memory](reflective-memory.md) (synthesis before retention)

---

## The Problem

Context files designed as state snapshots accumulate history when agents append rather than rewrite. The failure mode compounds across sessions:

- `HANDOFF.md` grows 4–5 stale sessions deep — agent must read and discard noise before reaching current state
- `activeContext.md` carries a "Recently Completed" list that grows unbounded — items are added but never removed
- `todo.md` accumulates a 26+ row Completed table and a closed Gaps section — completed work blocks view of active work
- Each extra token costs attention. A 300-line task file is not a richer signal — it is a noisier one

The root cause is not the agent writing — it is the absence of a pruning reflex. Append is the default write path. Rewrite requires intent.

---

## Core Principle

**A context file represents current state. Only current state.**

```
Before writing:  read file → extract only what is still true → rewrite
After writing:   file holds the smallest accurate representation of now
```

History is not lost. Git commits preserve every prior state. The expectation that context files are the history store is the architectural error — correct it by stating clearly: **git is the archive.**

This principle has one deliberate exception: `lessons.md` is append-only by design (see [Pruning Rules by File](#pruning-rules-by-file) below).

---

## The Pruning Algorithm

Every time an agent updates a context file, it follows this three-step sequence:

```
1. READ    the current file
2. EXTRACT what is still true, still active, still relevant
3. REWRITE the file from scratch with only that content
```

Rewrite, don't touch. Do not diff-and-patch. Do not append a new section. Replace the file entirely with the current state.

This is equivalent to a running `git stash drop` on stale context — except git already has it if you need it.

---

## Size Thresholds (Pruning Triggers)

When a file crosses the **Soft Cap**, prune before the next append. When it crosses the **Hard Cap**, prune immediately — do not write new content into an oversize file.

| File | Soft Cap | Hard Cap | Pruning Action |
|------|----------|----------|----------------|
| `HANDOFF.md` | 500 tokens | 700 tokens | Keep current session only; summarize prior session in 2–3 lines if still relevant, then drop |
| `activeContext.md` | 400 tokens | 600 tokens | Drop Recently Completed; keep only in-progress, next steps, open blockers, decisions <48h old |
| `todo.md` | 600 tokens | 900 tokens | Move Completed rows to git (commit the file as-is first); rewrite with empty Completed table; drop resolved Gaps |
| `HANDOFF.md` (multi-session) | — | 2 sessions | Never let more than 2 sessions coexist in this file |
| `systemPatterns.md` | 700 tokens | 900 tokens | Merge redundant patterns; remove patterns superseded by code changes |
| `CLAUDE.md` / hub | 500 tokens | 700 tokens | Trim stale routing entries; shorten rules that grew too verbose |
| `lessons.md` | no limit | no limit | **Append-only by design** — do not prune, load selectively instead |

*Token estimates assume GPT-4 tokenization. For rough line-count equivalents: 400 tokens ≈ 50 lines of prose.*

---

## Pruning Rules by File

### `HANDOFF.md` — Maximum 2 Sessions

`HANDOFF.md` is read at session start to restore context. It is not a session archive.

**Keep:**
- What was being worked on when the last session ended
- Any blocking decisions that remain unresolved
- The exact next step for the incoming session

**Drop:**
- Sessions older than the current + 1 prior
- Completed items that are now in code or in `todo.md`
- Design rationale that belongs in `systemPatterns.md` or `projectBrief.md`

**Rewrite trigger:** Every session end. Writing a new handoff means the old one becomes prior — prune it to a 2–3 line summary, then add the new session.

---

### `activeContext.md` — No History, Only Now

`activeContext.md` answers: "What are we doing right now?" If a piece of information answers a different question, it does not belong here.

**Keep:**
- Tasks that are currently in progress
- Blockers that have not been resolved
- Decisions made within the last 48 hours
- Next steps (maximum 5, ordered, actionable)
- Open questions awaiting answers

**Drop:**
- Anything in a "Recently Completed" section more than 2 items deep
- Decisions older than 72 hours that are now reflected in code
- Design context that is stable enough for `systemPatterns.md`

**Rewrite trigger:** When "Recently Completed" list reaches 3+ items, or when the file exceeds 400 tokens.

---

### `todo.md` — Active Work Only

`todo.md` is a task tracker, not a task history.

**Keep:**
- In Progress tasks (maximum 3)
- Backlog (prioritized, only unstarted items)
- Blockers with unresolved owners
- Discovered items not yet triaged
- Gaps that are still open

**Drop (after committing the file to git):**
- Completed table — empty it; git has the history
- Resolved Gaps rows
- Backlog sections that have been fully completed

**Rewrite trigger:** When Completed table exceeds 10 rows, or when the file exceeds 600 tokens. Commit first, then rewrite.

---

### `lessons.md` — The Deliberate Exception

`lessons.md` is the one file that should grow. It is the project's institutional knowledge. Pruning it removes real value.

Instead of pruning, load it selectively:
- Do not include `lessons.md` in every-session reads
- Load it only when working in a domain where past lessons are relevant
- If it grows very large (1500+ tokens), split by domain: `lessons-api.md`, `lessons-database.md`

See [Progressive Loading](progressive-loading.md) for selective load patterns.

---

## Pruning Contract (for templates)

Each context file template carries a **Pruning Contract** — a comment block that specifies the file's pruning rules. The contract makes the rules self-documenting and agent-readable without consulting this pattern file.

```markdown
<!-- PRUNING CONTRACT
     Type: [state-snapshot | append-only]
     Soft Cap: [N tokens]
     Hard Cap: [N tokens]
     Keep: [what to retain]
     Drop: [what to remove]
     Trigger: [when pruning is required]
     Archive: git history
-->
```

---

## Agent Instruction: The Trim-Before-Write Reflex

Embed this instruction in the agent's system prompt or CLAUDE.md for projects using context files:

```
Before updating any context file (activeContext.md, todo.md, HANDOFF.md):
1. Read the current file
2. Check if it exceeds its soft cap (see context-pruning pattern)
3. If yes: rewrite the file with current state only before adding new content
4. If no: proceed with update, but still drop stale completed/resolved items
Git is the archive — do not preserve history in context files.
```

---

## What This Is Not

**Not compression.** Compression reduces the token cost of content you must keep. Pruning removes content that is no longer relevant. Both are valuable; they are different operations. See [RTK](../tools/compression/rtk.md) for compression.

**Not archiving.** There is no `archive/` directory. Pruned content is not moved — it is dropped. Git retains it at the commit before the prune.

**Not summarizing.** Do not replace dropped content with a one-line summary unless the summary is genuinely needed for current work. The instinct to summarize before dropping usually signals attachment to history, not a real context need.

---

## Related Patterns

- [File Memory Structure](file-memory-structure.md) — defines the canonical file hierarchy and size guidelines this pattern enforces
- [Progressive Loading](progressive-loading.md) — the correct strategy for `lessons.md` (load selectively, don't prune)
- [Reflective Memory](reflective-memory.md) — if you need to synthesize before pruning, reflect first, then prune
- [Phase-Gated Context](phase-gated-context.md) — using phase transitions as natural pruning trigger points
