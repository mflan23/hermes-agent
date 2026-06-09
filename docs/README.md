# Hermes Documentation

This folder contains comprehensive documentation for the Hermes agent system, organized by domain and system component.

## Folder Structure

### `/architecture/`
System design and prompt engineering documentation.

- **`system-prompt.md`** — The core prompt architecture for Hermes, including:
  - Base identity and role definition
  - Voice & tone registers (Legal-Professional, Empathetic-Conversational, Research-Analytical, Strategic-Advisory)
  - Context detection logic
  - Domain expertise framework
  - Guardrails and limitations
  - Output structure guidelines
  - Memory integration patterns

### `/domains/`
Domain-specific expertise frameworks (to be developed).

Each domain will include:
- Key statutes and regulations
- Major case law and precedents
- Procedural requirements
- Common disputes and issues
- Domain-specific resources and agencies
- Recommended next steps by issue type

**Planned Domains**:
- `family-law.md` — Custody, divorce, adoption, child support
- `employment-law.md` — Wage disputes, discrimination, termination
- `torts.md` — Personal injury, negligence, liability
- `contract-law.md` — Formation, interpretation, breach, remedies
- `property-law.md` — Real estate, ownership, disputes
- `administrative-law.md` — Regulatory compliance, appeals
- `criminal-law.md` — Procedures, sentencing, appeals

### `/memory/`
Database and memory management documentation.

- **`database-schema.md`** — Supabase PostgreSQL schema including:
  - User data and preferences tables
  - Past tasks and research history
  - Case context management
  - Resource lookup caching
  - Memory interactions and conversation history
  - Views and common query patterns
  - Performance and indexing strategies
  - Data retention and privacy policies

---

## Quick Start

1. **Understanding Hermes**: Start with [SOUL.md](../SOUL.md) to grasp the mission and principles.

2. **System Architecture**: Review [system-prompt.md](architecture/system-prompt.md) to understand how Hermes reasons and communicates.

3. **Tools & Integrations**: Reference [TOOLS.md](../TOOLS.md) for API endpoints and integration details.

4. **Memory Layer**: Study [database-schema.md](memory/database-schema.md) to understand persistent state management.

5. **Domain Expertise**: Consult individual domain files as needed for specific legal areas.

---

## Key Concepts

### Voice & Tone Registers
Hermes adapts communication style based on context:
- **Legal-Professional** — For attorneys and legal professionals
- **Empathetic-Conversational** — For lay persons and emotional context
- **Research-Analytical** — For systematic research and methodology
- **Strategic-Advisory** — For decision support and planning

See [system-prompt.md](architecture/system-prompt.md#2-voice--tone-register-system) for details.

### Memory Layer
Hermes accumulates knowledge through:
- **User Preferences** — Communication style, expertise level, citation formats
- **Past Tasks** — Research history and successful methodologies
- **Case Context** — Active cases with persistent state across sessions
- **Resource Lookups** — Cached results for fast re-retrieval
- **Memory Interactions** — Conversation history for learning

See [database-schema.md](memory/database-schema.md) for implementation details.

### Domain Expertise
Each legal domain has:
- Core statutes and regulations
- Landmark cases
- Procedural pathways
- Common issues and solutions
- Relevant agencies and resources

See individual domain files for specialization.

---

## Contributing to Documentation

When updating documentation:

1. **System Prompt Changes**: Update [system-prompt.md](architecture/system-prompt.md) and reference SOUL.md principles.

2. **New Domains**: Create domain files in `/domains/` following the template in `domain-template.md` (if available).

3. **Schema Changes**: Update [database-schema.md](memory/database-schema.md) with migration notes and versioning.

4. **Tool Integrations**: Update [TOOLS.md](../TOOLS.md) when adding new APIs or endpoints.

---

## Related Files

- [SOUL.md](../SOUL.md) — Mission, vision, and core principles
- [README.md](../README.md) — Project overview and getting started
- [TOOLS.md](../TOOLS.md) — Tool manifest and integrations catalog
