# ARCHITECTURE.md — Hermes Agent
> *System Design, Agent Flow, and Memory Layer*

---

## Overview

He is designed for asynchronous, multi-domain task execution — routing inputs through specialized functional modules and returning structured, cited, audience-calibrated outputs.

> **Stack Note:** Hermes runs on an open, privacy-respecting stack. 
---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        USER / INTERFACE                      │
│         (Chat UI · Notion · API · CLI · Lovable frontend)    │
└────────────────────────────┬────────────────────────────────┘
                             │ input
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                     HERMES AGENT CORE                        │
│                   (LangGraph Agent Loop)                     │
│                                                              │
│  ┌─────────────┐   ┌──────────────┐   ┌─────────────────┐  │
│  │  Intent     │──▶│  Router /    │──▶│  Domain Module  │  │
│  │  Classifier │   │  Planner     │   │  (see below)    │  │
│  └─────────────┘   └──────────────┘   └────────┬────────┘  │
│                                                 │           │
│  ┌──────────────────────────────────────────────▼────────┐  │
│  │                   MEMORY LAYER                        │  │
│  │   (Supabase pgvector · Conversation History · Cache)  │  │
│  └───────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────┘
                             │ output
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                     OUTPUT LAYER                             │
│   (Structured text · Notion page · Markdown · JSON · PDF)   │
└─────────────────────────────────────────────────────────────┘
```

---

## LLM Provider Strategy

Hermes uses a tiered model routing strategy:

| Tier | Use Case |
|---|---|---|---|
| **Primary** | | Complex reasoning, legal analysis, advocacy drafting |
| **Secondary** | Fallback drafting, formatting, structured output |
| **Embeddings** | sentence-transformers, BAAI/bge | Semantic search, pgvector ingestion, document clustering |
| **Local / Private** | Fully offline processing for sensitive case data |

---

## Agent Flow

### Step 1 — Intent Classification
Every input is first passed through an intent classifier that determines:
- **Domain:** Legal / Research / Advocacy / Data / Triage
- **Register:** Academic / Advocacy / Operational
- **Output format:** Text / Document / Database entry / Code / Referral list
- **Ambiguity flag:** If intent is unclear, Hermes pauses and asks one clarifying question before proceeding.

### Step 2 — Planning
The planner decomposes the task into ordered subtasks. For complex tasks (e.g., "write a grant narrative for a reentry program"), the planner generates a step list, shows it to the user for confirmation if needed, then executes.

### Step 3 — Domain Module Execution
Each domain has a dedicated module with its own tool bindings (see `TOOLS.md`). Modules can be chained — e.g., Legal Intelligence feeds into Research Synthesis, which feeds into Advocacy Communication.

### Step 4 — Memory Read/Write
Before execution, Hermes reads relevant context from the memory layer. After execution, outputs and metadata are written back for future retrieval.

### Step 5 — Output Formatting
The output is formatted according to the detected register and requested format. Citations are appended. Confidence levels are noted where applicable.

### Step 6 — Escalation Check
Before delivery, Hermes checks whether the output crosses a human threshold (crisis content, legal advice, clinical guidance). If so, a boundary note and warm-handoff resource are appended.

---

## Domain Modules

```
hermes-agent/
└── modules/
    ├── legal/          # OCGA lookup, case law, OCR, document parsing
    ├── research/       # Literature search, citation, synthesis, drafting
    ├── advocacy/       # Brief writing, testimony, grant narratives
    ├── data/           # Supabase, Notion, GitHub, AWS routing
    └── triage/         # Resource matching, intake, referral mapping
```

Each module exposes a standard interface:
```python
async def run(task: HermesTask) -> HermesOutput:
    ...
```

Where `HermesTask` carries: `intent`, `domain`, `register`, `inputs`, `memory_context`  
And `HermesOutput` carries: `content`, `sources`, `confidence`, `escalation_flag`, `format`

---

## Memory Layer

Hermes uses a three-tier memory model:

### Tier 1 — Working Memory (In-Context)
Conversation history within the active session. 

### Tier 2 — Episodic Memory  
Persistent storage of past tasks, outputs, and user preferences. Queried via semantic similarity 

```sql
-- Core memory table
CREATE TABLE hermes_memory (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id TEXT,
  task_type TEXT,
  domain TEXT,
  input_summary TEXT,
  output_summary TEXT,
  sources JSONB,
  embedding VECTOR(768),
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Tier 3 — Knowledge Base (Supabase + AWS S3 + Cloudflare R2)
Static and semi-static reference data: legal resource directories, statute snapshots, curated DV/reentry resource lists, research corpora, and NotebookLM exports. Indexed for RAG retrieval.
---

## Dataset & Corpus Ingestion

Hermes treats the following as structured, citable datasets:

- **NotebookLM exports** — Markdown/JSON exports from curated research notebooks, ingested into the knowledge base and indexed for RAG retrieval.
- **Hugging Face Datasets** — Open datasets relevant to criminal justice, legal NLP, and social services, pulled via the `datasets` Python library.
- **Curated static datasets** — OCGA statute snapshots, DV resource directories, reentry program lists, stored in AWS S3 / Cloudflare R2.

All ingested datasets are versioned, timestamped, and stored with source metadata in Supabase.

---

## Prompt Architecture

Hermes uses a layered system prompt structure:

```
[SOUL] — Identity, values, boundaries (from SOUL.md — loaded at agent init)
[REGISTER] — Active voice/tone mode (set by intent classifier)
[DOMAIN CONTEXT] — Domain-specific instructions (loaded per module)
[MEMORY CONTEXT] — Relevant episodic memory (injected per task)
[TASK] — The current user input
```

This layered approach keeps the base prompt lean while allowing rich context injection without exceeding token limits.

---


---

## Security & Privacy

- **No PII transmitted to external APIs** without explicit user authorization.
- **Secrets managed via** environment variables + Supabase Vault + AWS Secrets Manager (never hardcoded).
- **Audit logging** on all memory writes and tool calls.

---


---

`ARCHITECTURE.md v1.2 — Hermes Agent`  
`Author: Mary Bay Flanagan`  
`Last Updated: June 2026`  
`Status: Active`
