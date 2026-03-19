# Pattern: Reflective Memory

> After accumulating memories, periodically synthesize new understanding from them. The `reflect` operation generates insights the agent was never explicitly told — derived from patterns across retained facts and experiences.

**WISC dimension:** W (enriches the W layer with a synthesis loop — writing derived understanding, not just retrieved facts)  
**Validated by:** [Hindsight](../tools/memory/hindsight.md) (SOTA on LongMemEval benchmark as of Jan 2026)

---

## The Problem

Most agent memory systems are recall pipes: store information, retrieve information. An agent with recall-only memory can answer "What did the user say about deployment preferences?" It cannot answer "Given everything I know about how this team operates, what approach are they likely to prefer for this new decision?"

The gap is synthesis. A recall-only agent:

- Retrieves what it was told, but never derives understanding from it
- Has no mechanism to form beliefs — only to surface stored facts
- Treats a high volume of related memories as N separate facts, never as a pattern
- Cannot answer "what should I expect?" — only "what did I see?"

Reflective memory is the loop that turns accumulated facts into understanding.

---

## Core Concept

```
retain  →  facts accumulate (world / experiences pathway)
              ↓ (periodically)
reflect →  facts are synthesized into mental models
              ↓
recall  →  both raw facts AND derived understanding are retrievable
```

The `reflect` operation does not retrieve — it generates. It takes a query about the memory bank and produces new observations that were never explicitly stored. These observations are retained as mental models, available to future recall operations.

---

## When to Call reflect

Reflect is not a per-request operation. It's a periodic synthesis step — called at natural break points, not on every query.

**Good triggers for reflect:**
- After a batch of related facts have been retained (e.g., after a sprint's decisions are logged)
- Before starting a new phase of work (synthesize what was learned in the previous phase)
- When the agent needs to answer a "what should I do?" question rather than "what did I see?"
- At session end, to consolidate episodic memory into durable understanding

**Example reflect queries:**
```python
# Synthesize project patterns
client.reflect(bank_id="project", query="What architectural patterns does this team consistently apply?")

# Synthesize risk landscape
client.reflect(bank_id="project", query="What risks have materialized or been flagged in recent work?")

# Synthesize behavioral understanding
client.reflect(bank_id="users", query="What communication and decision-making preferences does this user show?")

# Synthesize learning from failures
client.reflect(bank_id="project", query="What approaches have failed and why?")
```

---

## Implementation

### Step 1: Build the memory base with retain

Regular retain calls during the session, logged to a named bank:

```python
client.retain(bank_id="project", content="Team rejected the microservices approach due to operational overhead concerns")
client.retain(bank_id="project", content="Auth service was separated from main API due to security isolation requirements")
client.retain(bank_id="project", content="Database migrations run synchronously during deployment — intentional design decision")
```

### Step 2: Trigger reflect at phase transitions

```python
# At the end of a discovery/research phase, before planning begins
insights = client.reflect(
    bank_id="project",
    query="What constraints and preferences should guide the implementation approach?"
)

# Inject the synthesized insights into the planning phase context
planning_context = f"## Synthesized Project Understanding\n{insights}\n\n## Task\n{task_description}"
```

### Step 3: Use recall for per-request retrieval, reflect for phase-level synthesis

```python
# Per-request: use recall (fast, retrieves existing facts)
relevant_facts = client.recall(bank_id="project", query="payment API conventions")

# Phase transition: use reflect (slower, generates new understanding)
strategic_understanding = client.reflect(
    bank_id="project",
    query="What do I need to understand about this codebase before implementing?"
)
```

---

## In the Memory File Structure

Reflective memory maps to the `systemPatterns.md` and `lessons.md` files in the [File Memory Structure](file-memory-structure.md) pattern. The difference: reflect generates these syntheses automatically from accumulated memories, rather than requiring explicit human or agent curation.

This creates a pipeline:

```
session events  →  retain  →  accumulated bank
                               ↓ (at phase boundaries)
                    reflect  →  synthesized understanding
                               ↓
                              usable as Tier 2 context
```

---

## Evidence

Hindsight's reflect operation contributed to state-of-the-art performance on the **LongMemEval benchmark** as of January 2026. The benchmark tests whether agents can answer questions that require understanding patterns across a long conversation history — exactly the use case for synthesis over simple retrieval. Results were independently reproduced by Virginia Tech's Sanghani Center for AI and Data Analytics.

---

## Related Patterns

- [File Memory Structure](file-memory-structure.md) — how to structure banks and where synthesized insights live
- [Progressive Loading](progressive-loading.md) — reflective synthesis outputs are Tier 2 context
- [Bidirectional Memory](bidirectional-memory.md) — the write-side pattern that feeds the retain pipeline
