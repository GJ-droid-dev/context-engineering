# context-engineering

> Coding agents lose their mind around message 40. They hallucinate APIs they've never seen, forget decisions made an hour ago, and repeat mistakes across every session. This repo gives them a memory architecture — curated patterns, versioned rules, and a layered context system that makes your agent measurably smarter with every task. Everything here is open, synthesized from 16 real tools, and maintained as plain markdown — you can read exactly what your agent loads, remix it for your stack, and contribute what works back to the field.

---

## The Problem: Context Rot

LLM quality degrades measurably as a session grows. Research from Chroma shows attention at message 60 is ~20% of what it was at message 1. The typical symptoms:

- **Hallucinated APIs** — the agent invents method signatures it saw referenced 30 messages ago
- **Forgotten decisions** — architectural choices from earlier in the session get silently reversed
- **Repeated mistakes** — the same error fixed in message 10 reappears in message 50
- **Context stuffing** — devs paste the same boilerplate into every chat to keep the agent oriented

This is not an LLM limitation. It's a **context architecture problem**. And it has engineering solutions.

---

## The Solution: Context Engineering

Context engineering is the discipline of designing *what information your AI agent has access to, when, and in what form*. It sits between prompt engineering (one-off wording) and fine-tuning (model changes) — it's the infrastructure layer that keeps agents oriented across sessions.

The key insight: **the context window is not a dump; it's a curated feed.** Every token that loads into context is a choice. The goal is maximum signal per token — loading only what the agent needs, exactly when it needs it.

---

## Architecture: The 5 Layers

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 5: COMPRESSION                                   │
│  Reduce token cost of existing context (RTK, minifiers) │
├─────────────────────────────────────────────────────────┤
│  LAYER 4: WORKFLOW ORCHESTRATION                        │
│  Phase-gate context loading; sub-agent isolation        │
│  (Memory Bank, ECC, WISC framework)                     │
├─────────────────────────────────────────────────────────┤
│  LAYER 3: SESSION & PROJECT MEMORY                      │
│  Persist decisions, patterns, progress across sessions  │
│  (ConPort, Basic Memory, ICM)                           │
├─────────────────────────────────────────────────────────┤
│  LAYER 2: DOCUMENTATION CONTEXT                         │
│  Real-time, versioned, curated library docs in prompts  │
│  (Context7, Context Hub)                                │
├─────────────────────────────────────────────────────────┤
│  LAYER 1: FOUNDATIONAL MEMORY                           │
│  Application-level user/session memory for agents       │
│  (mem0, MCP Memory Server, LlamaIndex)                  │
└─────────────────────────────────────────────────────────┘
```

For the full architecture breakdown, see [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Repo Map

```
context-engineering/
├── README.md                          ← you are here
├── ARCHITECTURE.md                    ← full 5-layer architecture + tool mapping
│
├── framework/
│   └── wisc.md                        ← WISC: the organizing mental model
│
├── tools/                             ← one file per tool, sourced from real repos
│   ├── docs/
│   │   ├── context7.md               ← real-time library docs via MCP
│   │   ├── context-hub.md            ← curated, versioned, annotatable docs
│   │   └── code-review-graph.md      ← structural codebase graph; blast-radius context selection
│   ├── memory/
│   │   ├── basic-memory.md           ← bi-directional Markdown knowledge graph
│   │   ├── conport.md                ← SQLite project knowledge graph
│   │   ├── hindsight.md              ← biomimetic memory + reflective synthesis
│   │   ├── icm.md                    ← MCP-native session memory
│   │   └── mem0.md                   ← application-level memory SDK
│   ├── workflow/
│   │   ├── ecc.md                    ← skills/agents/rules system
│   │   ├── gsd.md                    ← reference implementation: fresh context per task
│   │   └── memory-bank.md            ← phase-based workflow + token savings
│   └── compression/
│       └── rtk.md                    ← 60–90% token reduction proxy
│
├── patterns/                          ← standalone techniques, validated by tools
│   ├── project-brief-bootstrap.md
│   ├── progressive-loading.md
│   ├── phase-gated-context.md
│   ├── bidirectional-memory.md
│   ├── file-memory-structure.md
│   ├── reflective-memory.md
│   └── sub-agent-isolation.md
│
├── editors/                           ← editor-specific implementation guides
│   ├── github-copilot.md
│   ├── cursor.md
│   └── cline.md
│
└── templates/                         ← copy-paste ready, battle-tested files
    ├── projectBrief.md.template
    ├── activeContext.md.template
    ├── tasks.md.template
    ├── CLAUDE.md.template
    └── prime-domain.instructions.md.template
```

---

## Quick Start

**If you use GitHub Copilot:** → [editors/github-copilot.md](editors/github-copilot.md)  
**If you use Cursor:** → [editors/cursor.md](editors/cursor.md)  
**If you use Cline:** → [editors/cline.md](editors/cline.md)

**If you want to understand the theory:** → [framework/wisc.md](framework/wisc.md) then [ARCHITECTURE.md](ARCHITECTURE.md)  
**If you want a specific pattern:** → browse [patterns/](patterns/)  
**If you want copy-paste files:** → browse [templates/](templates/)

---

## Sources

This repo synthesizes 16 open-source tools and frameworks:

| Source | Why It Matters | Contribution to This Repo |
|--------|----------------|---------------------------|
| WISC Framework | First clean mental model for the whole problem | Organizing architecture: W/I/S/C |
| [upstash/context7](https://github.com/upstash/context7) | Real-time docs pulled into prompts via MCP, native VS Code support | The live-docs pattern |
| [mem0ai/mem0](https://github.com/mem0ai/mem0) | Academic benchmark paper, +26% accuracy on LOCOMO | Application-level memory tiers |
| [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) | 108 skills, 25 agents, best-in-class CLAUDE.md hub | Skills-as-files, sub-agent isolation |
| [cline/cline](https://github.com/cline/cline) | `.clinerules` directory pattern | The project anchor pattern |
| [andrewyng/context-hub](https://github.com/andrewyng/context-hub) | Versioned, annotatable docs CLI + MCP | Stable curated docs layer |
| [rtk-ai/rtk](https://github.com/rtk-ai) | 60–90% token reduction proxy | Token compression architecture |
| [vanzan01/cursor-memory-bank](https://github.com/vanzan01/cursor-memory-bank) | ~70% token reduction via hierarchical loading | Progressive loading + phase-gating |
| [basicmachines-co/basic-memory](https://github.com/basicmachines-co/basic-memory) | LLMs write back to your files — truly bi-directional | Bi-directional memory pattern |
| [GreatScottyMac/context-portal](https://github.com/GreatScottyMac/context-portal) | SQLite project brain with explicit decision links | Structured project knowledge graph |
| [rtk-ai/icm](https://github.com/rtk-ai) | MCP-native session memory + VS Code auto-config | Session-level memory |
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | Anthropic's reference Memory MCP server | MCP as universal integration layer |
| [run-llama/llama_index](https://github.com/run-llama/llama_index) | Why hybrid BM25 + vector search beats either alone | RAG theoretical foundation |
| [vectorize-io/hindsight](https://github.com/vectorize-io/hindsight) | SOTA LongMemEval; biomimetic retain/recall/reflect architecture | Reflective memory pattern; Layer 1 extension |
| [gsd-build/gsd-2](https://github.com/gsd-build/gsd-2) | Full agent runtime implementing phase-gating + sub-agent isolation as code | Reference implementation of Layer 4 patterns |
| [tirth8205/code-review-graph](https://github.com/tirth8205/code-review-graph) | 6.8x token reduction via blast-radius structural analysis | Structural progressive loading pattern |

---

## Contributing

Patterns and tool notes that you've validated in production are the most valuable additions. To contribute:

1. Tool notes go in `tools/` — include setup steps, key gotchas, and a link to the source repo
2. Patterns go in `patterns/` — each must reference at least 2 tools that validate it
3. Templates go in `templates/` — must be copy-paste ready with zero placeholders left undefined

All content is plain markdown. No build step. No dependencies.

---

*Last updated: March 2026*
