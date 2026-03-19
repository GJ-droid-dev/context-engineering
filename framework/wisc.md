# WISC Framework

> The simplest mental model for context engineering. Four dimensions that cover every decision about what goes into an AI agent's context window.

---

## What Is WISC?

WISC is an organizing framework for thinking about context management in AI coding agents. It names four distinct strategies for controlling what your agent knows:

| Letter | Dimension | Core Question |
|--------|-----------|---------------|
| **W** | **Write** | What do we pre-author and persist for the agent to read? |
| **I** | **Isolate** | Which agents or phases get only their relevant slice? |
| **S** | **Select** | What do we pull into context on-demand for this specific task? |
| **C** | **Compress** | How do we shrink content that must be included? |

These four dimensions are **not steps** — they're independent levers you can apply simultaneously. A well-engineered context stack will use all four.

---

## W — Write

**The strategy:** Author context in advance and persist it so agents read from it consistently.

**The problem it solves:** Agents hallucinate or forget things that were established earlier. Writing them down in a structured, discoverable place eliminates the need to re-establish the same facts every session.

**What "write" looks like in practice:**

```
projectBrief.md          ← tech stack, architecture decisions, team conventions
CLAUDE.md / .cursorrules ← project-specific rules and constraints for the agent
decisions.log            ← why we chose Postgres over MongoDB, why tabs not spaces
activeContext.md         ← what we're currently working on and why
```

**W-layer tools from this repo:**
- [mem0](../tools/memory/mem0.md) — writes structured memory to a persistent store
- [ConPort](../tools/memory/conport.md) — `log_decision`, `log_system_pattern`, `log_progress`
- [Basic Memory](../tools/memory/basic-memory.md) — agent writes Observations and Relations to Markdown
- [ECC](../tools/workflow/ecc.md) — the CLAUDE.md hub pattern; skills-as-files
- [Context Hub](../tools/docs/context-hub.md) — curated, versioned doc bundles authored by humans
- [Hindsight](../tools/memory/hindsight.md) — retain/recall/reflect; agents that learn patterns from accumulated experience across sessions, not just retrieve recent facts

**Key principle:** The W layer is your single source of truth. If an agent could drift or forget it — write it down.

**Good W content:**
- Architectural decisions and their rationale
- Tech stack choices and version pins
- Patterns that must be followed (e.g., error handling conventions)
- Patterns that must be avoided (e.g., don't use this deprecated API)
- Ongoing work state and next steps

**Bad W content:**
- Ephemeral information (today's weather, current time)
- Information that changes faster than you'll update the file
- Raw code dumps without explanation

---

## I — Isolate

**The strategy:** Partition work across sub-agents or phases so each agent receives only the context relevant to its task.

**The problem it solves:** A single agent with all context is expensive, unfocused, and makes cascading mistakes. An agent writing a UI component doesn't need the payment service schema. Isolation keeps agents small, fast, and correct.

**What "isolate" looks like in practice:**

```
# Instead of one agent with everything:
Agent: "Refactor the entire codebase"

# Use isolated agents with scoped context:
Orchestrator → PlanningAgent (reads: requirements, architecture)
            → ResearchAgent (reads: specific file or API)
            → ImplementationAgent (reads: relevant files + rules only)
            → ReviewAgent (reads: diff + test results)
```

**I-layer tools from this repo:**
- [ECC](../tools/workflow/ecc.md) — 25 named sub-agents with defined context scopes
- [Cursor Memory Bank](../tools/workflow/memory-bank.md) — 6 phases, each with a defined context set
- [GSD-2](../tools/workflow/gsd.md) — fresh context window per task; hard process-boundary isolation between Milestones, Slices, and Tasks (not just instruction-level)

**Key principle:** The correct scope for an agent is the minimum context needed to do its job well. Not more.

**Isolation patterns:**
- **Phase isolation** — different context loads for discovery vs. implementation vs. review
- **Domain isolation** — backend agent never loads frontend rules and vice versa
- **Responsibility isolation** — one agent reads, one writes, one verifies

**Warning signs you need more isolation:**
- The agent keeps referencing things from 40+ messages ago without being prompted
- You're pasting the same large context block into every task
- The agent gives correct-sounding but wrong answers (ungrounded generation)

---

## S — Select

**The strategy:** Pull context into the window on-demand, based on what the current task actually needs.

**The problem it solves:** Loading context eagerly (everything at session start) wastes token budget on information the agent may never use. Selection loads only what's relevant, when it's relevant.

**What "select" looks like in practice:**

```
# Eager loading (bad):
System prompt = all rules + all docs + full schema + full history

# Selective loading (good):
Task: "Fix the auth middleware bug"
→ SELECT: auth.middleware.ts + auth rules + error handling pattern
→ SKIP: payment schema, UI guidelines, migration history
```

**S-layer tools from this repo:**
- [Context7](../tools/docs/context7.md) — selects the right library docs for the current import
- [Cursor Memory Bank](../tools/workflow/memory-bank.md) — loads the rule files for the current phase only
- [ConPort](../tools/memory/conport.md) — semantic search over logged decisions; retrieves relevant ones
- [Basic Memory](../tools/memory/basic-memory.md) — hybrid BM25 + vector search over the knowledge graph
- [ICM](../tools/memory/icm.md) — retrieves the relevant slice of session history
- [code-review-graph](../tools/docs/code-review-graph.md) — blast-radius structural selection; loads only the dependency graph of the changed file (avg 6.8x token reduction)

**Key principle:** Context selection is a query, not a dump. The better you define what "relevant" means for a task, the better the selection.

**Selection triggers:**
| Signal | What to Select |
|--------|----------------|
| File being edited | Rules for that file's domain |
| Library import detected | Docs for that library via Context7 |
| Task involves a past decision | Retrieve that decision + its rationale |
| Starting a new phase | Load the phase-specific rule set |

---

## C — Compress

**The strategy:** Reduce the token cost of content that must be in context, without losing its semantic content.

**The problem it solves:** Some content can't be excluded — it's mandatory. Compression doesn't remove it; it makes it smaller. This frees budget for more relevant content or reduces cost per call.

**What "compress" looks like in practice:**

```
# Verbose (uncompressed):
"In this project, we follow a convention where all React 
components must be functional components and must use the 
'use client' directive at the top of the file if they contain
any hooks such as useState, useEffect, or useSelector."

# Compressed:
"All React components: functional only. Add 'use client' if using any hooks."
```

**C-layer tools from this repo:**
- [RTK](../tools/compression/rtk.md) — CLI proxy that rewrites verbose LLM I/O; 60–90% reduction

**Manual compression techniques:**
- Replace prose rules with bullet lists or tables
- Use code examples instead of describing code in natural language
- Remove "helpful" explanations the agent doesn't need
- Strip all politeness framing from system prompts
- Use abbreviations for frequently repeated terms with a legend at the top

**Key principle:** Compress *after* you know what must be included. Compressing the wrong content first is premature optimization.

**Compression ordering:** Write → Isolate → Select → **Compress**. You can only compress what you've decided must be there.

---

## Putting WISC Together

The four dimensions work together, not in isolation. Here's how a real project uses all four:

```
W — Write a projectBrief.md, a decisions.log, and domain-specific rules files
I — Separate planning from implementation; scope agents to their domain
S — Use Context7 for live docs; load only the rules file for the current file type
C — Run docs through RTK before injecting; keep rules files under 500 tokens each
```

**A session that uses all four W/I/S/C dimensions well will:**
1. Start with stable facts already written (not re-established from scratch)
2. Give each agent only what it needs
3. Pull current docs rather than relying on training data
4. Maximize the information density of every token in the window

---

## WISC vs. The 5 Layers

WISC describes *how* you manage context. The [5-layer architecture](../ARCHITECTURE.md) describes *what kind* of context you're managing. They're complementary lenses:

| Layer | Primarily Which WISC Dimension |
|-------|-------------------------------|
| Layer 1 — Foundational Memory | W (write and persist) |
| Layer 2 — Documentation Context | S (just-in-time selection) |
| Layer 3 — Session & Project Memory | W + S (write and selectively retrieve) |
| Layer 4 — Workflow Orchestration | I + S (isolate and select by phase) |
| Layer 5 — Compression | C (compress what must be included) |

---

## Quick Reference Card

```
W — WRITE:    Author it. Persist it. CLAUDE.md, decisions.log, projectBrief.md
I — ISOLATE:  One agent, one job, one context scope. Sub-agents beat one big agent.
S — SELECT:   Pull on demand. Load what this task needs, not everything you have.
C — COMPRESS: Shrink mandatory content. Tables > prose. Examples > descriptions.
```
