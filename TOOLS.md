# TOOLS.md — Hermes Agent
> *Tool Manifest: Domain-to-Integration Mapping*

---

## Overview

This document maps each of Hermes's functional domains to the specific tools, APIs, SDKs, and services he uses to execute work. Every tool listed here is either currently integrated, planned for integration, or available as a plugin depending on deployment context.

Hermes does not use tools blindly. Each call is purposeful, logged where possible, and traceable to a specific task or output.

> **Stack Note:** Hermes runs on an open, privacy-respecting stack. No Google Cloud Platform services are used (no Vertex AI, no BigQuery, no Cloud Run, no GCS).

---

## Legal Intelligence

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **OCGA Online** (law.justia.com) | Georgia statute lookup and cross-referencing | Web fetch / scrape |
| **CourtListener API** (Free Law Project) | Federal and state case law retrieval | REST API |
| **Tesseract OCR** | PDF court document parsing and structuring | Python SDK (pytesseract) |
| **Anthropic Claude API** | Legal document summarization and plain-language translation | REST API |
| **LlamaIndex** | RAG pipeline over legal document collections | Python SDK |
| **Ollama** (local models) | Offline/private legal document processing | Local SDK |
| **NotebookLM exports** | Pre-synthesized research corpus ingestion as structured datasets | JSON / Markdown datasets |

---

## Research Synthesis

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **Semantic Scholar API** | Academic paper search and metadata retrieval | REST API |
| **CrossRef API** | Citation verification and DOI resolution | REST API |
| **Zotero API** | Citation library management and export | REST API |
| **Anthropic Claude API** | Literature synthesis, thematic coding, draft generation | REST API |
| **OpenAI API** | Draft generation and formatting fallback | REST API |
| **Hugging Face Inference API** | Embeddings generation for semantic search and clustering | REST API |
| **NotebookLM exports** | Structured dataset ingestion from curated research notebooks | JSON / Markdown datasets |
| **Notion API** | Research output storage, literature review databases | REST API |

---

## Advocacy Communication

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **Anthropic Claude API** | Brief drafting, testimony generation, grant narrative writing | REST API |
| **OpenAI API** | Secondary drafting and editing support | REST API |
| **Notion API** | Document storage, template management, publishing | REST API |
| **Readable.io / Hemingway logic** | Reading level calibration for plain-language outputs | Algorithmic |
| **SendGrid / Resend** | Warm-handoff communication and outreach drafting | REST API |

---

## Data & Systems Routing

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **Supabase** | Primary database: storage, ingestion pipelines, RLS policies | Supabase JS/Python SDK |
| **Supabase pgvector** | Vector embeddings for semantic search over legal/research docs | PostgreSQL extension |
| **Hugging Face Inference API** | Embedding generation for pgvector ingestion | REST API |
| **Notion API** | Knowledge base management, task tracking, page creation | REST API |
| **GitHub API** | Repository management, issue tracking, commit coordination | REST API / Octokit |
| **AWS S3** | Document and file storage (PDFs, OCR outputs, exports) | AWS SDK (boto3) |
| **AWS Lambda** | Serverless function execution for pipelines and triggers | AWS SDK |
| **Cloudflare R2** | Secondary/edge file storage, S3-compatible | S3-compatible SDK |
| **Railway / Render** | Backend API and agent hosting | Platform SDK |
| **LangChain / LangGraph** | Agent orchestration, tool chaining, memory management | Python SDK |
| **Firebase** | Real-time data sync for frontend-facing features | Firebase SDK |

---

## Human-Centered Triage

| Tool / Source | Purpose | Integration Type |
|---|---|---|
| **211.org API / findhelp.org (Aunt Bertha)** | Social services and DV resource matching by geography | REST API |
| **NCADV / Ahimsa House resource data** | DV shelter, legal aid, and safety planning references | Curated dataset |
| **Georgia Legal Aid / GLSP directories** | Civil legal aid referral matching | Web fetch / curated |
| **Reentry.net / CSG Justice Center data** | Reentry housing, employment, expungement resources | Web fetch / curated |
| **Supabase** | Resource database storage and eligibility filtering | Supabase SDK |
| **Anthropic Claude API** | Intake interpretation, needs assessment structuring | REST API |
| **NotebookLM exports** | Pre-processed resource guides and policy summaries | JSON / Markdown datasets |

---

## Cross-Cutting Infrastructure

| Tool | Purpose | Integration Type |
|---|---|---|
| **LangChain / LangGraph** | Agent loop orchestration, tool routing, memory | Python SDK |
| **Anthropic Claude API** (claude-3-5-sonnet / haiku) | Primary LLM backbone | REST API |
| **OpenAI API** (gpt-4o / gpt-4o-mini) | Secondary LLM and fallback | REST API |
| **Hugging Face Inference API** | Embeddings, open model access, fine-tuned model hosting | REST API |
| **Ollama** | Local/offline model fallback for sensitive or private data | Local SDK |
| **Supabase pgvector** | Long-term semantic memory and document retrieval | PostgreSQL extension |
| **AWS Lambda + S3** | Serverless pipeline execution and file storage | AWS SDK (boto3) |
| **Cloudflare R2** | Edge file storage, PDFs, exports | S3-compatible SDK |
| **Railway / Render** | Primary backend and agent API hosting | Platform |
| **GitHub Actions** | CI/CD pipeline for agent updates and testing | YAML workflows |
| **Cursor IDE / VS Code** | Development environment | Local |
| **Lovable** | Rapid frontend scaffolding for user-facing interfaces | No-code platform |

---

## Tool Governance

- **No tool is called without purpose.** Every tool call maps to a defined task.
- **Failures are surfaced, not silently swallowed.** If a tool returns an error or empty result, Hermes reports it and offers an alternative path.
- **No Google Cloud Platform.** Hermes does not use Vertex AI, BigQuery, Cloud Run, or Google Cloud Storage.
- **Data privacy is enforced.** No personally identifiable information from intake or case data is sent to external APIs without explicit authorization.
- **Local fallback available.** Sensitive tasks can route to Ollama for fully offline processing.
- **NotebookLM exports are treated as structured datasets** — ingested, indexed, and cited like any other research corpus.

---

`TOOLS.md v1.2 — Hermes Agent`  
`Author: Mary Bay Flanagan`  
`Last Updated: June 2026`  
`Status: Active`
