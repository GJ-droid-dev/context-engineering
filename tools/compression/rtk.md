# RTK (Reduce Token Kit)

> A CLI proxy that compresses your AI agent's input and output, achieving 60–90% token reduction on common coding tasks. Add it between your editor and the LLM API.

**Source:** [rtk-ai/rtk](https://github.com/rtk-ai) — 9.5k stars  
**Layer:** 5 — Compression  
**WISC:** C (compress mandatory content)

---

## The Problem It Solves

Some context is mandatory. You can't exclude the codebase when you're asking the agent to refactor it. You can't exclude the error log when you're asking the agent to debug it. But verbose code, over-explained errors, and padding in LLM outputs all waste tokens without adding information.

RTK sits in the pipeline and rewrites verbose content into compact equivalents — same information, fewer tokens.

---

## How It Works

RTK acts as a transparent CLI proxy:

```
Your Editor → RTK → LLM API → RTK → Your Editor
```

On the inbound side, RTK compresses the prompt (code files, error logs, context strings) before sending to the LLM.

On the outbound side, RTK compresses the LLM's response (strips padding, redundant explanations, apology preambles) before returning it to the editor.

The net effect: **the LLM sees a denser prompt and returns a denser response.** You see the same quality of answer, but the API call costs less and fits more useful context into the same token window.

---

## Setup

```bash
npm install -g @rtk-ai/rtk
rtk init
```

**Configure your editor to route through RTK:**

The `rtk init` command generates the necessary proxy config. For VS Code, it sets up a local proxy endpoint that intercepts Copilot API calls.

**Manual config (if needed):**
```bash
rtk serve --port 4117
# Then set OPENAI_API_BASE=http://localhost:4117 in your environment
```

---

## What Gets Compressed

| Input Type | Compression Strategy | Typical Reduction |
|------------|---------------------|-------------------|
| Code files | Remove blank lines, inline short comments, abbreviate variable names with a legend | 30–50% |
| Error logs | Extract stack trace only; strip duplicate frames | 50–70% |
| LLM responses | Strip preamble ("Great question!"), strip restating the question, strip sign-off | 20–40% |
| Documentation | Extract only the relevant API signatures from large doc pages | 60–80% |
| Conversation history | Summarize older turns into compact fact statements | 70–90% |

---

## When RTK Matters Most

- **Large codebase analysis:** When you need to send multiple files to the agent
- **Long sessions:** When conversation history is accumulating and eating the window
- **Tight token budgets:** When using smaller/cheaper models with limited context windows
- **High-frequency agent calls:** When running automated pipelines with many LLM calls

---

## Manual Compression Techniques (Without RTK)

If you don't want to install a proxy, these manual techniques give partial results:

```markdown
# Instead of prose rules:
"In this project, we follow the convention where all Express controllers 
must be wrapped in the asyncHandler utility function imported from 
@/utils/asyncHandler, and all expected errors should be thrown as 
new AppError instances with a message and HTTP status code."

# Compressed:
Controllers: asyncHandler wrapper required. Errors: throw AppError(msg, code).
```

```markdown
# Instead of full error logs:
[paste entire 200-line stack trace]

# Compressed: send only
Error: Cannot read property 'userId' of undefined
  at authenticate (middlewares/auth.ts:23:24)
  at Layer.handle [as handle_request] (express/lib/router/layer.js:95:5)
```

```markdown
# Instead of full file contents:
[paste entire 300-line service file]

# Compressed: paste only the relevant function + its imports
```

---

## Compression Ordering

Compression is the last layer in the context architecture, not the first:

```
1. Write the right content (W)
2. Isolate to the right agents (I)
3. Select only relevant content (S)
4. Compress what remains (C)
```

Compressing everything upfront without selection is counterproductive — you may compress away relevance. Apply compression *after* the selection problem is solved.

---

## In This Repo's Architecture

RTK implements the C dimension of WISC — the layer that maximizes information density of content that must be present. It's Layer 5 because it operates on whatever the other four layers have assembled, squeezing maximum value from the remaining token budget.

**Relevant patterns:** [Progressive Loading](../../patterns/progressive-loading.md)
