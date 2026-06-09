# Supabase Memory Layer Schema

This document defines the database schema for Hermes's persistent memory layer. This schema enables Hermes to accumulate intelligence over time, recall user preferences, track research history, and maintain case context.

---

## Overview

The memory layer uses **PostgreSQL via Supabase** to store:

1. **User Data** — Account information and metadata
2. **User Preferences** — Communication style, expertise level, citation preferences
3. **Past Tasks** — Research history, enabling pattern recognition and context recall
4. **Case Context** — Active and archived case information with cross-references
5. **Resource Lookups** — Cached research results and frequently-accessed resources
6. **Memory Interactions** — Detailed conversation history for learning and improvement

**Design Principles**:
- **Row-Level Security (RLS)** — Strict data isolation per user
- **Indexing** — Optimized for rapid retrieval and search
- **Versioning** — Track updates to important records (e.g., case context changes)
- **Archival** — Soft deletes to preserve historical data
- **JSONB for Flexibility** — Complex nested data without rigid schema constraints

---

## 1. Core Tables

### 1.1 `users` Table

**Purpose**: Central user identity and metadata

```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  expertise_level TEXT CHECK (expertise_level IN ('beginner', 'intermediate', 'expert')),
  preferred_communication_style TEXT CHECK (preferred_communication_style IN 
    ('technical', 'conversational', 'balanced')),
  timezone TEXT DEFAULT 'America/New_York',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,
  
  CONSTRAINT email_valid CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_expertise ON users(expertise_level);
```

**Row-Level Security**:
```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read their own data"
  ON users FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can update their own data"
  ON users FOR UPDATE
  USING (auth.uid() = user_id);
```

**Fields**:
- `user_id`: Unique identifier (UUID)
- `email`: User's email address (unique)
- `name`: Full name or display name
- `expertise_level`: Self-assessed or inferred expertise in legal matters
- `preferred_communication_style`: Used to set default tone/register
- `timezone`: For scheduling and timestamps
- `created_at`, `updated_at`: Audit timestamps
- `deleted_at`: Soft delete marker (NULL if active)

---

### 1.2 `user_preferences` Table

**Purpose**: Granular user preferences for communication, citations, and behavior

```sql
CREATE TABLE user_preferences (
  preference_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  key TEXT NOT NULL,
  value TEXT,
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  UNIQUE(user_id, key)
);

CREATE INDEX idx_user_preferences_user_id ON user_preferences(user_id);
```

**Common Preferences**:
```json
{
  "cite_format": "full|brief|bluebook",
  "detail_level": "high|medium|low",
  "domain_focus": "family_law|employment_law|torts|...",
  "response_length": "short|medium|long",
  "include_analogies": "true|false",
  "include_case_summaries": "true|false",
  "preferred_agencies": ["georgia_bar", "dol", "eeoc"],
  "exclude_jurisdictions": ["federal", "other_states"],
  "abbreviate_statutes": "true|false",
  "consent_research_tracking": "true|false"
}
```

**Insertion Example**:
```sql
INSERT INTO user_preferences (user_id, key, value, metadata)
VALUES (
  '123e4567-e89b-12d3-a456-426614174000',
  'cite_format',
  'brief',
  '{"description": "User prefers brief citations without page numbers"}'
);
```

---

### 1.3 `past_tasks` Table

**Purpose**: Research history for recall, pattern recognition, and contextual memory

```sql
CREATE TABLE past_tasks (
  task_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  query TEXT NOT NULL,
  query_embedding VECTOR(1536),  -- If using embeddings for similarity search
  domain TEXT,  -- e.g., 'family_law', 'employment_law', 'torts'
  result_summary TEXT,
  result_full JSONB,
  resources_used JSONB,  -- Array of {source, items, urls}
  confidence_score NUMERIC CHECK (confidence_score >= 0 AND confidence_score <= 1),
  tags TEXT[],
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  accessed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  accessed_count INTEGER DEFAULT 1
);

CREATE INDEX idx_past_tasks_user_id ON past_tasks(user_id);
CREATE INDEX idx_past_tasks_domain ON past_tasks(domain);
CREATE INDEX idx_past_tasks_created_at ON past_tasks(created_at DESC);
CREATE INDEX idx_past_tasks_tags ON past_tasks USING GIN(tags);
CREATE INDEX idx_past_tasks_embedding ON past_tasks USING ivfflat (query_embedding vector_cosine_ops);
```

**Resources Used Schema** (JSONB):
```json
{
  "resources": [
    {
      "source": "ocga",
      "items": ["§ 34-7-2", "§ 34-7-4"],
      "urls": ["https://law.justia.com/..."]
    },
    {
      "source": "case_law",
      "items": ["Smith v. Jones, 250 Ga. App. 123"],
      "urls": ["https://scholar.google.com/..."]
    },
    {
      "source": "notion",
      "items": ["family_law_notes", "case_template_custody"]
    }
  ],
  "search_terms": ["wage and hour", "overtime"],
  "ai_model": "gemini-1.5-pro"
}
```

**Result Full Schema** (JSONB):
```json
{
  "answer": "Full text of the response provided to the user",
  "citations": ["Smith v. Jones...", "O.C.G.A. § 34-7-2"],
  "confidence": 0.95,
  "limitations": ["...", "..."],
  "follow_up_suggestions": ["...", "..."]
}
```

**Insertion Example**:
```sql
INSERT INTO past_tasks (user_id, query, domain, result_summary, resources_used, tags)
VALUES (
  '123e4567-e89b-12d3-a456-426614174000',
  'Can an employer refuse to pay overtime?',
  'employment_law',
  'Under O.C.G.A. § 34-7-2, employers must pay overtime...',
  '{"resources": [...]}',
  ARRAY['overtime', 'wage_and_hour', 'employment']
);
```

**Retrieval for Memory Context**:
```sql
-- Find similar past tasks
SELECT query, result_summary, tags FROM past_tasks
WHERE user_id = $1 AND domain = $2
ORDER BY accessed_count DESC, created_at DESC
LIMIT 3;

-- Or with embedding similarity (if using pgvector)
SELECT query, result_summary FROM past_tasks
WHERE user_id = $1
ORDER BY query_embedding <-> $2  -- $2 is the current query embedding
LIMIT 3;
```

---

### 1.4 `case_context` Table

**Purpose**: Track active and archived cases, enabling session continuity and case-specific reasoning

```sql
CREATE TABLE case_context (
  context_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  case_name TEXT NOT NULL,
  case_status TEXT CHECK (case_status IN ('active', 'archived', 'closed')),
  case_details JSONB NOT NULL,
  related_tasks UUID[],  -- Array of task IDs related to this case
  notes TEXT,
  last_accessed TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT case_details_required CHECK (case_details IS NOT NULL)
);

CREATE INDEX idx_case_context_user_id ON case_context(user_id);
CREATE INDEX idx_case_context_status ON case_context(case_status);
CREATE INDEX idx_case_context_created_at ON case_context(created_at DESC);
CREATE INDEX idx_case_context_related_tasks ON case_context USING GIN(related_tasks);
```

**Case Details Schema** (JSONB):
```json
{
  "parties": ["Smith", "Jones"],
  "case_type": "family_law",
  "subcategory": "custody",
  "jurisdiction": "Georgia Superior Court",
  "county": "Fulton",
  "status_description": "Awaiting judge decision",
  "key_dates": {
    "filed": "2024-01-15",
    "hearing_scheduled": "2024-08-20",
    "deadline_response": "2024-07-15"
  },
  "legal_issues": ["custody_determination", "visitation_rights"],
  "related_statutes": ["O.C.G.A. § 19-9-1", "O.C.G.A. § 19-9-3"],
  "relevant_case_law": [
    {"citation": "Smith v. Jones, 250 Ga. 123", "relevance": "similar custody facts"}
  ],
  "urgency": "high",
  "metadata": {
    "attorney_name": "Jane Doe",
    "contact": "jane@example.com"
  }
}
```

**Insertion Example**:
```sql
INSERT INTO case_context (user_id, case_name, case_status, case_details)
VALUES (
  '123e4567-e89b-12d3-a456-426614174000',
  'Smith v. Jones - Custody',
  'active',
  '{
    "parties": ["Smith", "Jones"],
    "case_type": "family_law",
    "jurisdiction": "Fulton Superior Court",
    ...
  }'::jsonb
);
```

**Retrieval for Context**:
```sql
-- Get active cases for a user
SELECT case_name, case_details FROM case_context
WHERE user_id = $1 AND case_status = 'active'
ORDER BY last_accessed DESC;

-- Get a specific case with related research
SELECT * FROM case_context
WHERE user_id = $1 AND case_name ILIKE $2;
```

---

### 1.5 `resource_lookups` Table

**Purpose**: Cache frequently-accessed research results for faster retrieval and pattern recognition

```sql
CREATE TABLE resource_lookups (
  lookup_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  query TEXT NOT NULL,
  source TEXT NOT NULL,  -- 'ocga', 'case_law', 'notion', 'web', etc.
  results JSONB NOT NULL,
  accessed_count INTEGER DEFAULT 1,
  last_accessed TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  ttl_days INTEGER DEFAULT 90,  -- Time-to-live for cache
  
  UNIQUE(user_id, source, query)
);

CREATE INDEX idx_resource_lookups_user_id ON resource_lookups(user_id);
CREATE INDEX idx_resource_lookups_source ON resource_lookups(source);
CREATE INDEX idx_resource_lookups_accessed_at ON resource_lookups(last_accessed DESC);
```

**Results Schema** (varies by source):

**OCGA Results**:
```json
{
  "source": "ocga",
  "query": "§ 34-7-2",
  "results": [
    {
      "section": "34-7-2",
      "title": "Payment of wages",
      "text": "Full text of statute...",
      "effective_date": "2020-06-15",
      "amendments": ["2020-HB 550"],
      "related_sections": ["34-7-1", "34-7-3", "34-7-4"]
    }
  ]
}
```

**Case Law Results**:
```json
{
  "source": "case_law",
  "query": "Georgia wage and hour",
  "results": [
    {
      "citation": "Smith v. Jones, 250 Ga. App. 123, 549 S.E.2d 456 (2001)",
      "court": "Georgia Court of Appeals",
      "year": 2001,
      "summary": "...",
      "headnotes": ["...", "..."],
      "url": "https://scholar.google.com/..."
    }
  ]
}
```

**Insertion & Update**:
```sql
INSERT INTO resource_lookups (user_id, query, source, results)
VALUES ($1, $2, $3, $4)
ON CONFLICT (user_id, source, query) DO UPDATE
SET accessed_count = accessed_count + 1,
    last_accessed = NOW(),
    results = EXCLUDED.results;
```

---

### 1.6 `memory_interactions` Table

**Purpose**: Store detailed conversation history for continuous improvement and learning patterns

```sql
CREATE TABLE memory_interactions (
  interaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  case_context_id UUID REFERENCES case_context(context_id),
  task_id UUID REFERENCES past_tasks(task_id),
  conversation_turn INTEGER,  -- Sequential within a session
  user_message TEXT NOT NULL,
  assistant_response TEXT NOT NULL,
  tone_register TEXT,  -- 'legal_professional', 'empathetic_conversational', etc.
  domains_detected TEXT[],
  response_quality_rating SMALLINT CHECK (response_quality_rating >= 1 AND response_quality_rating <= 5),
  user_feedback TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  archived BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_memory_interactions_user_id ON memory_interactions(user_id);
CREATE INDEX idx_memory_interactions_case_id ON memory_interactions(case_context_id);
CREATE INDEX idx_memory_interactions_created_at ON memory_interactions(created_at DESC);
```

**Usage**:
```sql
-- Store a conversation turn
INSERT INTO memory_interactions (
  user_id, case_context_id, conversation_turn, 
  user_message, assistant_response, tone_register, domains_detected
)
VALUES (
  $1, $2, 1,
  'What is Georgia wage and hour law?',
  'O.C.G.A. § 34-7-2 requires...',
  'empathetic_conversational',
  ARRAY['employment_law', 'wage_and_hour']
);

-- Retrieve conversation history for context
SELECT user_message, assistant_response FROM memory_interactions
WHERE user_id = $1 AND case_context_id = $2
ORDER BY conversation_turn DESC
LIMIT 10;
```

---

## 2. Views for Common Queries

### 2.1 User Preferences View
```sql
CREATE VIEW user_preferences_view AS
SELECT 
  user_id,
  MAX(CASE WHEN key = 'cite_format' THEN value END) as cite_format,
  MAX(CASE WHEN key = 'detail_level' THEN value END) as detail_level,
  MAX(CASE WHEN key = 'domain_focus' THEN value END) as domain_focus,
  MAX(CASE WHEN key = 'response_length' THEN value END) as response_length
FROM user_preferences
WHERE deleted_at IS NULL
GROUP BY user_id;
```

### 2.2 Active Cases View
```sql
CREATE VIEW active_user_cases AS
SELECT 
  user_id,
  case_name,
  case_details->>'case_type' as case_type,
  case_details->>'status_description' as status,
  last_accessed
FROM case_context
WHERE case_status = 'active'
ORDER BY last_accessed DESC;
```

### 2.3 Recent Research View
```sql
CREATE VIEW recent_research AS
SELECT 
  user_id,
  query,
  domain,
  result_summary,
  tags,
  created_at,
  accessed_count
FROM past_tasks
ORDER BY created_at DESC
LIMIT 100;
```

---

## 3. Access Patterns & Queries

### 3.1 Load User Memory for a Session
```sql
-- Get user context
SELECT * FROM users WHERE user_id = $1;

-- Get user preferences
SELECT * FROM user_preferences_view WHERE user_id = $1;

-- Get active cases
SELECT case_name, case_details FROM active_user_cases WHERE user_id = $1;

-- Get recent research
SELECT query, domain, result_summary FROM past_tasks
WHERE user_id = $1 AND domain = $2
ORDER BY created_at DESC
LIMIT 5;
```

### 3.2 Record a New Task
```sql
INSERT INTO past_tasks (
  user_id, query, domain, result_summary, resources_used, tags, created_at
) VALUES ($1, $2, $3, $4, $5, $6, NOW());
```

### 3.3 Update Case Context
```sql
UPDATE case_context
SET case_details = case_details || $1,
    updated_at = NOW(),
    last_accessed = NOW()
WHERE user_id = $2 AND context_id = $3;
```

### 3.4 Cache Lookup Results
```sql
INSERT INTO resource_lookups (user_id, query, source, results)
VALUES ($1, $2, $3, $4)
ON CONFLICT (user_id, source, query) DO UPDATE
SET accessed_count = accessed_count + 1, last_accessed = NOW();
```

---

## 4. Performance & Indexing Strategy

### Indexes for Common Queries
- **By User**: `idx_*_user_id` on all main tables for rapid user-specific queries
- **By Time**: `idx_*_created_at` for chronological retrieval
- **By Status**: `idx_case_context_status` for filtering active/archived cases
- **Full-Text Search**: GIN index on `tags` for tag-based searches
- **Vector Search**: ivfflat index on embeddings (if using pgvector)

### Query Performance
- User-specific queries should complete <100ms due to RLS + indexed lookup
- Domain-specific research history: <200ms
- Case context retrieval: <50ms

---

## 5. Data Retention & Privacy

### Retention Policies
- `users`: Never deleted (soft-delete via `deleted_at`)
- `past_tasks`: Retained indefinitely for memory accumulation
- `memory_interactions`: Retained for 2 years by default, then archived
- `resource_lookups`: TTL-based (default 90 days, refreshable)

### Privacy & Security
- All user data is encrypted at rest
- RLS ensures users can only access their own data
- Sensitive fields (case details, user notes) are not logged
- Compliance: GDPR-ready with user deletion support

---

## 6. Migration & Deployment

### Initial Migration Script
```sql
-- Run these commands in Supabase SQL editor to initialize schema

-- 1. Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgvector";

-- 2. Create tables
[All CREATE TABLE statements above]

-- 3. Enable RLS
[All ALTER TABLE ENABLE ROW LEVEL SECURITY statements]

-- 4. Create RLS policies
[All CREATE POLICY statements]

-- 5. Create indexes
[All CREATE INDEX statements]

-- 6. Create views
[All CREATE VIEW statements]
```

### Deployment Checklist
- [ ] Verify RLS is enabled on all user-data tables
- [ ] Test row-level security policies
- [ ] Verify indexes are created and performant
- [ ] Test key access patterns with sample data
- [ ] Document backup and recovery procedures
- [ ] Set up monitoring for query performance
- [ ] Configure TTL/archival jobs for old records

---

## 7. Future Enhancements

- **Vector Embeddings**: Use pgvector for semantic similarity on research queries
- **Full-Text Search**: PostgreSQL full-text search for more sophisticated search
- **Change Tracking**: Audit tables to track all modifications (versioning)
- **Real-Time Subscriptions**: Supabase Realtime for live case updates
- **ML Pipeline**: Automated confidence scoring based on historical accuracy
