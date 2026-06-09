# Tool Manifest for Hermes Agent

This document maps each functional domain to its corresponding APIs, integrations, and data sources. Hermes uses these tools to access information, persist state, and drive reasoning.

---

## 1. Legal Research & Information Access

### 1.1 Georgia Code (OCGA) - Legal Database

**Purpose**: Access official Georgia Code and case law

**Integration Type**: Web Scraper / API

**Key Endpoints**:
- **OCGA Official**: https://law.justia.com/georgia/codes/ (Justia mirror)
- **Alternative**: https://advance.lexis.com/ (if institutional access available)

**Capabilities**:
- Full-text search across Georgia Code sections
- Statute retrieval with amendment history
- Related case law linking
- Cross-references between sections

**Authentication**: Public access (no auth required for basic queries)

**Rate Limits**: Respect robots.txt; implement delays between requests

**Response Format**: HTML parsing or API JSON (if available)

**Usage Example**:
```
Query: "O.C.G.A. § 34-7-2 wage and hour"
Returns: Full text, related cases, legislative history, effective date
```

---

### 1.2 Case Law Search

**Purpose**: Research appellate and trial court decisions

**Integration Type**: API / Web Scraper

**Key Endpoints**:
- **Georgia Court of Appeals**: https://www.gaappeals.us/
- **Google Scholar**: https://scholar.google.com/ (public access)
- **Fastcase/Westlaw** (if institutional access)

**Capabilities**:
- Search by case name, party, citation, topic
- Full opinion text retrieval
- Docket information
- Headnotes and syllabus

**Authentication**: Public for Google Scholar; institutional credentials for premium services

**Rate Limits**: Vary by service

**Response Format**: HTML or structured data

**Usage Example**:
```
Query: "Georgia Supreme Court personal injury" years:2020-2025
Returns: Relevant cases with citations, summaries, full opinions
```

---

### 1.3 Georgia Statutes & Administrative Rules

**Purpose**: Access state administrative rules and regulations

**Integration Type**: API / Web Scraper

**Key Endpoints**:
- **Georgia Secretary of State**: https://sos.ga.gov/
- **Georgia Administrative Procedure**: https://law.ga.gov/

**Capabilities**:
- Department-specific regulations
- Rule adoption and amendment tracking
- Regulatory impact assessments

**Authentication**: Public access

**Rate Limits**: Standard web scraping etiquette

---

## 2. Knowledge Management & Notes

### 2.1 Notion MCP (Model Context Protocol)

**Purpose**: Store, retrieve, and organize legal research, case notes, and reference materials

**Integration Type**: MCP Server / API

**Protocol**: Model Context Protocol (secure, structured communication)

**Key Operations**:
- **Create/Update Database Records**: Store cases, statutes, research summaries
- **Query Databases**: Full-text and structured search across knowledge base
- **Retrieve Relationships**: Pull linked references between cases, statutes, and notes

**Authentication**: Notion API Token (OAuth 2.0)

**Typical Schema**:
```
Database: Cases
  - Name (Title)
  - Citation (unique ID)
  - Court (select)
  - Date Decided (date)
  - Summary (rich text)
  - Related Statutes (relation)
  - Tags (multi-select)

Database: Statutes
  - Code Section (Title, e.g., "O.C.G.A. § 34-7-2")
  - Full Text (rich text)
  - Effective Date (date)
  - Amendment History (rich text)
  - Related Cases (relation)

Database: Research Topics
  - Topic (Title)
  - Notes (rich text)
  - Sources (relation to Cases/Statutes)
  - Status (select: In Progress, Complete)
```

**Capabilities**:
- Store legal research and case summaries
- Cross-link cases and statutes
- Maintain research notes and methodologies
- Search across full knowledge base

**Usage Example**:
```
Operation: Create new case record
Input: {
  "database": "Cases",
  "properties": {
    "Name": "Smith v. Jones",
    "Citation": "2024 WL 123456",
    "Court": "Georgia Supreme Court",
    "Date Decided": "2024-06-15",
    "Summary": "Plaintiff argued...",
    "Related Statutes": ["O.C.G.A. § 34-7-2"]
  }
}
Output: Record ID, confirmation
```

---

## 3. User Data & Preferences

### 3.1 Supabase Client (PostgreSQL)

**Purpose**: Centralized persistence layer for user data, memory, and preferences

**Integration Type**: Supabase SDK / REST API

**Key Features**:
- Real-time database with PostgreSQL
- Row-level security (RLS) policies
- Full-text search indexing
- Automatic backups and recovery

**Tables** (see `/docs/memory/database-schema.md` for full details):

#### 3.1.1 `users` Table
```
Columns:
- user_id (UUID, primary key)
- email (text, unique)
- name (text)
- expertise_level (enum: beginner, intermediate, expert)
- preferred_communication_style (enum: technical, conversational, balanced)
- timezone (text)
- created_at (timestamp)
- updated_at (timestamp)
```

#### 3.1.2 `user_preferences` Table
```
Columns:
- preference_id (UUID, primary key)
- user_id (UUID, foreign key)
- key (text) — e.g., "cite_format", "detail_level"
- value (text) — stored preference value
- created_at (timestamp)
- updated_at (timestamp)
```

#### 3.1.3 `past_tasks` Table
```
Columns:
- task_id (UUID, primary key)
- user_id (UUID, foreign key)
- query (text)
- result_summary (text)
- resources_used (jsonb)
- created_at (timestamp)
- domain (text) — e.g., "family_law", "torts"
```

#### 3.1.4 `case_context` Table
```
Columns:
- context_id (UUID, primary key)
- user_id (UUID, foreign key)
- case_name (text)
- case_details (jsonb) — structured case information
- related_tasks (UUID[]) — array of related task IDs
- active (boolean)
- created_at (timestamp)
- updated_at (timestamp)
```

#### 3.1.5 `resource_lookups` Table
```
Columns:
- lookup_id (UUID, primary key)
- user_id (UUID, foreign key)
- query (text) — original search query
- results (jsonb) — cached results
- source (text) — e.g., "ocga", "case_law"
- accessed_count (integer)
- last_accessed (timestamp)
- created_at (timestamp)
```

**Authentication**: Supabase API Key (public/anonymous for client-side, secret for backend)

**Connection String**:
```
******db.supabase.co:5432/postgres
```

**Row-Level Security**: Enable RLS on all user-specific tables to ensure data isolation

**Usage Example**:
```sql
-- Retrieve user preferences
SELECT key, value FROM user_preferences 
WHERE user_id = $1 AND key IN ('cite_format', 'detail_level');

-- Log a research task
INSERT INTO past_tasks (user_id, query, domain, resources_used, created_at)
VALUES ($1, $2, $3, $4, NOW());

-- Find similar past tasks
SELECT query, result_summary FROM past_tasks
WHERE user_id = $1 AND domain = $2
ORDER BY created_at DESC
LIMIT 5;
```

**Capabilities**:
- Fast CRUD operations with PostgreSQL
- Complex queries for memory retrieval
- Real-time subscriptions for live updates
- Full-text search across task history

---

## 4. AI/LLM & Reasoning

### 4.1 Google Vertex AI

**Purpose**: Primary LLM provider for reasoning, synthesis, and conversation

**Integration Type**: REST API / Python SDK

**Key Models**:
- **Gemini 1.5 Pro**: Advanced reasoning, long-context handling, multimodal
- **Gemini 1.5 Flash**: Fast responses, lower latency, cost-effective
- **Gemini 2.0**: Latest frontier model (if available)

**Project Setup**:
```
Project ID: [your-gcp-project]
Location: us-central1 (or preferred region)
Service Account: Enable Vertex AI API
Authentication: Service account key (JSON) or Application Default Credentials
```

**Key Endpoints**:
- **Generate Content**: `POST /v1/projects/{projectId}/locations/{location}/publishers/google/models/{modelId}:generateContent`
- **Streaming**: `POST /v1/projects/{projectId}/locations/{location}/publishers/google/models/{modelId}:streamGenerateContent`

**Authentication**: Service account credentials (stored securely)

**Request Parameters**:
```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {"text": "Your prompt here"}
      ]
    }
  ],
  "system_instruction": {
    "parts": [
      {"text": "You are Hermes, the legal research agent..."}
    ]
  },
  "generation_config": {
    "temperature": 0.7,
    "top_p": 0.95,
    "max_output_tokens": 2048
  },
  "safety_settings": [
    {
      "category": "HARM_CATEGORY_DANGEROUS_CONTENT",
      "threshold": "BLOCK_ONLY_HIGH"
    }
  ]
}
```

**Response Format**: Structured JSON with content, stop reason, finish details

**Usage Example**:
```python
from vertexai.generative_models import GenerativeModel

model = GenerativeModel("gemini-1.5-pro")
response = model.generate_content(
    "Summarize O.C.G.A. § 34-7-2 for a non-lawyer",
    system_instruction="You are Hermes, a legal research agent..."
)
print(response.text)
```

**Cost Model**: Pay-per-request; higher for longer contexts and more complex models

**Rate Limits**: Configurable; typically 60 requests per minute for standard accounts

---

## 5. Data Integration & Synchronization

### 5.1 ETL Pipelines (Future)

**Purpose**: Scheduled data synchronization and caching

**Planned Components**:
- **OCGA Sync**: Weekly scrape and cache of Georgia Code updates
- **Case Law Sync**: Periodic updates from legal databases
- **Notion Backup**: Regular sync of Notion data to Supabase for redundancy

**Tools**: Cloud Scheduler, Cloud Functions, or dedicated ETL service

---

## 6. Memory & Context Management

### 6.1 Session Memory

**Purpose**: In-conversation context and short-term state

**Implementation**:
- Conversation history maintained in request context
- Sliding window of recent messages (e.g., last 10 exchanges)
- Temporary reasoning artifacts (calculations, analyses, drafts)

**Persistence**: In-memory during session; archivable to Supabase upon completion

---

## Integration Architecture

```
User Input
    ↓
[Hermes Prompt Engine] — Vertex AI (reasoning, synthesis)
    ↓
    ├─→ [Tool Router] — Routes to appropriate integration
    │       ├─→ OCGA Scraper (if researching Georgia Code)
    │       ├─→ Case Law Search (if looking up precedent)
    │       ├─→ Notion MCP (if retrieving prior research)
    │       ├─→ Supabase (if accessing user preferences, memory)
    │       └─→ Vertex AI (for analysis and synthesis)
    ↓
[Response Composer]
    ↓
User Output + Memory Update
```

---

## Configuration & Secrets

All integrations require secure credential management:

**Environment Variables** (`.env` file, not committed):
```
SUPABASE_URL=https://[project].supabase.co
SUPABASE_KEY=[anon-key]
VERTEX_AI_PROJECT_ID=[gcp-project]
VERTEX_AI_LOCATION=us-central1
NOTION_API_KEY=[your-notion-token]
OCGA_CACHE_ENABLED=true
```

**Service Accounts**:
- Vertex AI: GCP service account with `aiplatform.user` role
- Notion: Integration token with read/write permissions
- Supabase: API key with RLS-compliant scope

---

## Monitoring & Observability

- **Supabase**: Built-in query performance monitoring
- **Vertex AI**: Cloud Logging integration for request/response tracking
- **Notion**: Rate limit tracking and error logging
- **Custom**: Application-level metrics (latency, accuracy, cache hit rates)

---

## Future Integrations

- **Twilio**: SMS/voice communication
- **GitHub**: Research notes and code snippets
- **Email**: Sending research summaries and alerts
- **Custom Domain Scrapers**: Industry-specific data sources (regulatory databases, legal publications)
