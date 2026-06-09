# TOOLS.md — Hermes Agent
> *Tool Manifest: Domain-to-Integration Mapping*

---

## Overview

This document maps each of Hermes's functional domains to the specific tools, APIs, SDKs, and services he uses to execute work. Every tool listed here is either currently integrated, planned for integration, or available as a plugin depending on deployment context.

Hermes does not use tools blindly. Each call is purposeful, logged where possible, and traceable to a specific task or output.

---

## Legal Intelligence

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **OCGA Online** (law.justia.com / lexisnexis) | Georgia statute lookup and cross-referencing | Web fetch / scrape |
| **CourtListener API** (Free Law Project) | Federal and state case law retrieval | REST API |
| **Google Scholar** | Case law search and citation verification | Web fetch |
| **Tesseract OCR / Google Document AI** | PDF court document parsing and structuring | SDK / GCP API |
| **Vertex AI (Gemini)** | Legal document summarization and plain-language translation | GCP SDK |
| **LangChain / LlamaIndex** | RAG pipeline over legal document collections | Python SDK |

---

## Research Synthesis

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **Semantic Scholar API** | Academic paper search and metadata retrieval | REST API |
| **CrossRef API** | Citation verification and DOI resolution | REST API |
| **Zotero API** | Citation library management and export | REST API |
| **Vertex AI (Gemini)** | Literature synthesis, thematic coding, draft generation | GCP SDK |
| **Google AI Studio** | Prompt prototyping and model testing | Web UI / API |
| **Notion API** | Research output storage, literature review databases | REST API |
| **Google Docs API** | Academic draft creation and formatting | GCP SDK |

---

## Advocacy Communication

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **Vertex AI (Gemini)** | Brief drafting, testimony generation, grant narrative writing | GCP SDK |
| **Notion API** | Document storage, template management, publishing | REST API |
| **Google Docs API** | Final document formatting and export | GCP SDK |
| **Readable.io / Hemingway logic** | Reading level calibration for plain-language outputs | Algorithmic |
| **SendGrid / Gmail API** | Warm-handoff communication and outreach drafting | REST API |

---

## Data & Systems Routing

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **Supabase** | Primary database: storage, ingestion pipelines, RLS policies | Supabase JS/Python SDK |
| **Supabase pgvector** | Vector embeddings for semantic search over legal/research docs | PostgreSQL extension |
| **Notion API** | Knowledge base management, task tracking, page creation | REST API |
| **GitHub API** | Repository management, issue tracking, commit coordination | REST API / Octokit |
| **BigQuery** | Large-scale data querying and analytics pipelines | GCP SDK |
| **Google Cloud Storage** | Document and file storage (PDFs, OCR outputs, exports) | GCP SDK |
| **Vertex AI Pipelines** | Orchestrated ML/LLM workflow execution | GCP SDK |
| **Firebase** | Real-time data sync for frontend-facing features | Firebase SDK |
| **LangChain** | Agent orchestration, tool chaining, memory management | Python SDK |

---

## Human-Centered Triage

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **211.org API / Aunt Bertha (findhelp.org)** | Social services and DV resource matching by geography | REST API |
| **NCADV / Ahimsa House resource data** | DV shelter, legal aid, and safety planning references | Curated dataset |
| **Georgia Legal Aid / GLSP directories** | Civil legal aid referral matching | Web fetch / curated |
| **Reentry.net / CSG Justice Center data** | Reentry housing, employment, expungement resources | Web fetch / curated |
| **Supabase** | Resource database storage and eligibility filtering | Supabase SDK |
| **Vertex AI (Gemini)** | Intake interpretation, needs assessment structuring | GCP SDK |

---

## Cross-Cutting Infrastructure

| Tool | Purpose | Integration Type |
|---|---|---|
| **LangChain / LangGraph** | Agent loop orchestration, tool routing, memory | Python SDK |
| **Vertex AI (Gemini 1.5 Pro / Flash)** | Primary LLM backbone | GCP SDK |
| **Supabase pgvector** | Long-term semantic memory and document retrieval | PostgreSQL extension |
| **Google Cloud Run** | Serverless agent deployment | GCP |
| **GitHub Actions** | CI/CD pipeline for agent updates and testing | YAML workflows |
| **Cursor IDE / VS Code** | Development environment | Local |
| **Lovable** | Rapid frontend scaffolding for user-facing interfaces | No-code platform |

---

## Tool Governance

- **No tool is called without purpose.** Every tool call maps to a defined task.
- **Failures are surfaced, not silently swallowed.** If a tool returns an error or empty result, Hermes reports it and offers an alternative path.
- **Costs are tracked.** GCP API calls, especially Vertex AI, are monitored against budget thresholds.
- **Data privacy is enforced.** No personally identifiable information from intake or case data is sent to external APIs without explicit authorization.

---

`TOOLS.md v1.0 — Hermes Agent`  
`Author: Mary Bay Flanagan`  
`Last Updated: June 2026`  
`Status: Active`
