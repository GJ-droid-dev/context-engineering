# Pattern: Bidirectional Memory

> The agent doesn't just read from your knowledge base — it writes back to it. Every conversation is an opportunity to grow the project's structured knowledge, not just consume it.

**WISC dimension:** W (Write — agent as author, not just reader)  
**Validated by:** [Basic Memory](../tools/memory/basic-memory.md) (the only tool built specifically for this)

---

## The Problem

Most context architectures are one-directional:
```
Developer writes files → Agent reads files → Agent produces output
```

The knowledge flows one way. The agent learns things in a session — discovers a tricky edge case, identifies a pattern, resolves an ambiguity — and all of that is lost when the session ends. The next session's agent starts as ignorant as the last.

Bidirectional memory makes the agent a contributor, not just a consumer:
```
Developer writes files → Agent reads + writes files → Knowledge grows
```

---

## The Pattern

At meaningful moments during a session, the agent writes structured notes back to the project's knowledge base. These notes are:

1. **Persistent** — they survive session end
2. **Human-readable** — plain markdown, editable by the developer
3. **Queryable** — indexed for retrieval in future sessions
4. **Versioned** — commit the knowledge base to git like any other source

---

## What the Agent Should Write

### Observations (facts about entities)

```markdown
## PaymentWebhookHandler

#payment #webhook #security

### Observations
- Razorpay sends raw body; must NOT use body-parser before signature verification
- Signature verification uses HMAC-SHA256 with RAZORPAY_WEBHOOK_SECRET
- Verified in: backend/src/controllers/payment.controller.ts:handleWebhook
- This tripped us up on 2026-03-15 — 2 hours debugging
```

### Relations (connections between entities)

```markdown
## Relations
- depends_on: [[RazorpaySDK]]
- validates_with: [[WebhookSecretEnvVar]]
- tested_by: [[PaymentRoutes.test.ts]]
```

### Decisions (why, not just what)

```markdown
## Decision: Raw body middleware scoped to /webhooks only

Date: 2026-03-15
Context: Needed raw body for Razorpay signature verification but body-parser 
was already global.
Decision: Added express.raw() only on the /webhooks route prefix, before 
the global body-parser.
Consequence: Webhook verification works. All other routes unaffected.
Alternative considered: global raw body + manual JSON.parse — rejected because 
it breaks multer and form parsing.
```

---

## When to Trigger a Write

The agent should write to the knowledge base when:

| Trigger | What to Write |
|---------|---------------|
| Bug resolved after >30 min debugging | Observation: root cause + what tripped us up |
| Architectural decision made | Decision: what + why + alternatives |
| New pattern established | System pattern: how it works + where it's used |
| Surprising behavior discovered | Observation: the gotcha + how to avoid it |
| Task completed | Progress: what was done + what's next |

---

## Implementation with Basic Memory

The [Basic Memory](../tools/memory/basic-memory.md) MCP server is the primary tool for this pattern. Configure it, then instruct the agent:

```markdown
# In your CLAUDE.md or system prompt

## Knowledge Base
After completing any significant task or discovering a non-obvious fact:
1. Write an Observation to the knowledge base via `write_note`
2. Link related entities via `create_relations`
3. Include: what was discovered + why it matters + where in the code it applies
```

The agent then writes naturally during conversation, building up the graph over time.

---

## Implementation Without Basic Memory

If you're not using Basic Memory, a simpler variant works:

```markdown
# lessons.md (manually maintained)

## 2026-03-15: Razorpay Webhook Raw Body
Raw body middleware must come BEFORE global body-parser, and only on the 
/webhooks route prefix. Global application breaks multer.
→ See: backend/src/app.ts lines 34-38

## 2026-03-10: Prisma Spread Leak
Spreading Prisma query results (`...query`) leaks internal fields including 
password hashes. Always destructure: `const { id, email, role } = query`.
→ Pattern established in: backend rules, auth controller
```

This is simpler but requires the developer (not the agent) to do the writing. The full bidirectional pattern offloads this to the agent.

---

## The Compounding Effect

A knowledge base written incrementally across sessions compounds:

- Session 1: 0 notes. Agent has no memory.
- Session 10: 20 notes. Agent recalls one relevant gotcha per task.
- Session 50: 80 notes. Agent has internalized the project's idiosyncrasies.
- Session 100: 200 notes. Agent avoids known pitfalls automatically.

This is why the pattern is worth the overhead. The value is not in session 1; it's in session 50.

---

## Related Patterns

- [File Memory Structure](file-memory-structure.md) — how to organize the files the agent writes to
- [Project Brief Bootstrap](project-brief-bootstrap.md) — the stable context that bidirectional memory grows alongside
- [Progressive Loading](progressive-loading.md) — how to load the knowledge base efficiently at session start
