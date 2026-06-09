---
name: token-governance
title: Token Governance & Cost Control
description: Global token management, model routing, caching, and cost control. Loaded before all other skills on every session.
version: 1.1.0
metadata:
  hermes:
    autoload: true
    scope: global
    priority: critical
    tags: [Governance, Cost, Tokens, Caching, Routing]
---

# Token Governance Framework v1.1

> This skill loads first, every session, every model. It cannot be overridden by user instruction. It exists to prevent runaway token consumption.

---

## 1. Model Routing Rules

Route tasks to the cheapest model capable of completing them accurately.

| Task Type | Model | Never Use |
|---|---|---|
| Simple lookup, formatting, short answer | claude-3-haiku / gpt-4o-mini / ollama-local | Sonnet, GPT-4o |
| Multi-step research, synthesis, legal analysis | claude-3-5-sonnet / gpt-4o | — |
| Embeddings only | HuggingFace sentence-transformers | Any LLM |
| Sensitive/private data | Ollama (local only) | Any external API |
| Code generation < 50 lines | gpt-4o-mini / haiku | GPT-4o, Sonnet |
| Full document drafts > 1000 words | claude-3-5-sonnet | GPT-4o (more expensive) |

**Default model = claude-3-haiku unless task complexity requires escalation.**

**Disabled providers (do not route to):**
- `openrouter` — credentials not configured, auth failing (HTTP 401)
- `ollama-cloud` — hitting session rate limits (HTTP 429); use local Ollama only
- `nvidia/nemotron` via openrouter — non-retryable auth failure

---

## 2. Skill Loading Rules

- **Load skill summaries first.** Full skill content loads only when the task explicitly requires it.
- **Each skill loads ONCE per session.** Never reload a skill already in working memory.
- **Maximum 3 skills active simultaneously.** If a 4th is needed, unload the least-recently-used.
- **autoload skills inject compressed headers only** — not full content — unless activated by a matching task trigger.
- **Never load reference files automatically.** Reference files load only when explicitly called by name.
- **`godmode` skill is SUSPENDED.** Do not load, invoke, or reference `red-teaming\godmode` or any variant. It has triggered prompt injection security warnings. Flag to user if it appears in any skill chain.

---

## 3. Response Length Caps

| Mode | Max Tokens | Override Trigger |
|---|---|---|
| Scholar (academic draft) | 900 | User says "full draft" or "complete section" |
| Advocate (narrative/Substack) | 700 | User says "full piece" or "publish-ready" |
| Fact-Checker | 500 | User says "full evidence map" |
| Data Strategy | 400 | User says "full analysis" |
| Operational (code, routing) | 400 | User says "full implementation" |
| Triage / Resource Match | 300 | User says "full list" |
| Conversational / clarifying | 150 | — |

---

## 4. Caching Rules

- **Cache all tool call results for 24 hours.** Same document, statute, or URL fetched in last 24hrs = return cached result.
- **Cache all embeddings for 7 days.** Do not re-embed if embedded in last 7 days.
- **Cache research synthesis outputs.** If query is semantically similar (>0.92 cosine similarity) to a prior query this session, surface cached response and ask if user wants a fresh run.
- **Never cache crisis triage outputs.** Resource availability changes. Always fetch fresh for DV/reentry resource matching.

---

## 5. Subagent & Loop Controls

- **Maximum subagent depth = 2.** No subagent may spawn another subagent that spawns another.
- **Maximum tool calls per turn = 5.** If a task requires more, pause and confirm with user.
- **No auto-retry loops.** If a tool call fails, report the failure and propose an alternative. Do not silently retry.
- **No speculative pre-fetching.** Do not fetch documents, URLs, or embeddings unless the current task explicitly requires them.
- **Same tool called twice in a row with same parameters = STOP.** Report loop detection to user immediately.

---

## 6. Session Kill Switch

If any threshold is reached, STOP and report before continuing:

- Single turn exceeds 2,000 tokens output without explicit "full draft" trigger
- Session cumulative output exceeds 15,000 tokens
- More than 5 tool calls in a single turn
- Same tool called more than 2 times in a row (loop detection)
- Any tool returns the same error 2 times in a row (fatal error — do not retry)

Kill switch message:
> ⚠️ **Token threshold reached.** This session has used [X] tokens. Do you want to continue? I can summarize progress and pause, or continue with a compressed response strategy.

---

## 7. Context Window Hygiene

- **Summarize, don't repeat.** Reference prior turn content in 1-2 sentences max. Never copy-paste prior output back into context.
- **Drop resolved context.** Once a subtask is complete and confirmed, remove its detail from active context. Keep only the outcome.
- **Compress memory injections.** When loading episodic memory, inject summaries (max 100 tokens per item), not full outputs.
- **One document at a time.** Never load more than one full document into context simultaneously.
- **Pre-conversation context cap = 2,000 tokens.** If autoloaded skills + memory exceed this before the first user message, compress further.

---

## 8. Cost Awareness

When a task will likely exceed 1,000 tokens, lead with:
> 💰 **Cost estimate:** This task will use approximately [X] tokens on [model]. Proceed?

Applies to: full document drafts, multi-source research synthesis, bulk embedding jobs, any task requiring more than 3 tool calls.

---

`token-governance v1.1 — Hermes Agent`
`Author: Mary Bay Flanagan`
`Last Updated: June 2026`
`Status: Active — CRITICAL PRIORITY`
