# ARCHITECTURE.md — Hermes Agent
> *System Design, Agent Flow, and Memory Layer*

---

## Overview

Hermes is a LangChain/LangGraph-orchestrated AI agent running on a Vertex AI (Gemini) backbone, with Supabase as the primary data and memory layer. He is designed for asynchronous, multi-domain task execution — routing inputs through specialized functional modules and returning structured, cited, audience-calibrated outputs.

This document describes the system architecture, agent flow, memory design, and deployment model.

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
│   (Structured text · Notion page · Google Doc · JSON · PDF) │
└─────────────────────────────────────────────────────────────┘
```

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
    ├── data/           # Supabase, Notion, GitHub, BigQuery routing
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
Conversation history within the active session. Managed by LangGraph's state graph. Cleared on session end unless explicitly saved.

### Tier 2 — Episodic Memory (Supabase)
Persistent storage of past tasks, outputs, and user preferences. Queried via semantic similarity using pgvector embeddings.

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

### Tier 3 — Knowledge Base (Supabase + GCS)
Static and semi-static reference data: legal resource directories, statute snapshots, curated DV/reentry resource lists, research corpora. Indexed for RAG retrieval.

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

## Deployment Model

| Component | Platform | Notes |
|---|---|---|
| Agent runtime | Google Cloud Run | Serverless, auto-scaling |
| LLM backbone | Vertex AI (Gemini 1.5 Pro / Flash) | Pro for complex tasks, Flash for speed |
| Database / memory | Supabase (PostgreSQL + pgvector) | Primary data layer |
| File storage | Google Cloud Storage | PDFs, OCR outputs, exports |
| CI/CD | GitHub Actions | Auto-deploy on push to main |
| Frontend (optional) | Lovable / Firebase | User-facing chat or intake interface |
| Dev environment | Cursor IDE / VS Code | Local development |

---

## Security & Privacy

- **Row-Level Security (RLS)** enforced on all Supabase tables.
- **No PII transmitted to external APIs** without explicit user authorization.
- **Service accounts scoped minimally** — each GCP service account has only the permissions its module requires.
- **Secrets managed via** Google Secret Manager (never hardcoded).
- **Audit logging** on all memory writes and tool calls.

---

## Roadmap

- [ ] `modules/legal/` — OCGA RAG pipeline (v1)
- [ ] `modules/triage/` — Georgia DV resource matcher (v1)
- [ ] Memory layer schema and Supabase setup
- [ ] LangGraph agent loop scaffold
- [ ] Google Cloud Run deployment config
- [ ] GitHub Actions CI/CD pipeline
- [ ] Lovable frontend for intake interface
- [ ] `modules/research/` — Academic synthesis pipeline (v1)
- [ ] `modules/advocacy/` — Grant narrative generator (v1)

---

`ARCHITECTURE.md v1.0 — Hermes Agent`  
`Author: Mary Bay Flanagan`  
`Last Updated: June 2026`  
`Status: Active`
