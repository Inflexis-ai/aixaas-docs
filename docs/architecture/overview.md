# Architecture Overview — MAO Platform

## Design Philosophy

MAO is designed around **aviation dual-redundancy**: every critical path has a validated fallback, every agent handoff is logged as an immutable record, and no single component failure can corrupt the system or lose data.

This is not a pattern borrowed from AI research — it is borrowed from the Federal Aviation Administration's system safety standards, applied to enterprise AI orchestration.

---

## System Topology

```
┌────────────────────────────────────────────────────────────────────────┐
│                           Azure Container Apps                         │
│                                                                        │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        FastAPI Application                      │   │
│  │                                                                 │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐     │   │
│  │  │  /api/ingest  │ │  /api/chat   │  │  /api/admin        │     │   │
│  │  │  /api/ingest  │ │  /api/chat   │  │  /api/meetings     │     │   │
│  │  │  /api/ingest  │ │  /api/status │  │  /auth/*           │     │   │
│  │  └──────┬────────┘ └──────┬───────┘  └────────┬───────────┘     │   │
│  │         │                 │                   │                 │   │
│  │  ┌──────▼──────────────────▼────────────────────▼───────────┐   │   │
│  │  │                    Core Modules                          │   │   │
│  │  │  Universal Ingestor · Compliance Engine · Model Router   │   │   │
│  │  │  ADR Logger · Archive Manager · Semantic Cache           │   │   │
│  │  │  User Context · Prompt Composer · Guardrails             │   │   │
│  │  │  Meeting Intelligence · PII Trust Layer                  │   │   │
│  │  └──────────────────────────┬───────────────────────────────┘   │   │
│  │                             │                                   │   │
│  │  ┌──────────────────────────▼────────────────────────────────┐  │   │
│  │  │                   Storage Layer (4-Tier)                  │  │   │
│  │  │  T1: Keyword/disk   T2: Qdrant   T3: Pinecone   T4: Azure │  │   │
│  │  └───────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────┐   │
│  │  Key Vault   │  │  AI Search   │  │ Blob Storage │  │  Entra ID │   │
│  │  (secrets)   │  │  (production │  │  (archive +  │  │  (SSO +   │   │
│  │              │  │   vector KB) │  │   recordings │  │   RBAC)   │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └───────────┘   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Storage Architecture

### 4-Tier Hybrid Knowledge Store

The MAO knowledge base uses a tiered storage architecture where all active tiers participate in every search, and results are merged by relevance score:

**Tier 1 — LocalFallbackStore (always active)**
- BM25 keyword index stored to disk as JSON shards
- Survives container restarts — loaded on startup from persistent Azure Files volume
- No external dependencies — guaranteed to respond even when all cloud services are down
- Appropriate for: guaranteed availability, air-gapped environments

**Tier 2 — LocalSemanticStore (when configured)**
- Qdrant vector database with sentence-transformer embeddings
- 768-dimensional dense vectors, cosine similarity search
- Disk-backed — index persists across container restarts
- Appropriate for: semantic (meaning-based) search without cloud dependency

**Tier 3 — PineconeStore (when configured)**
- Pinecone cloud vector database
- Managed, serverless — no infrastructure to maintain
- Appropriate for: cloud deployment with external vector store preference

**Tier 4 — AzureSearchStore (production)**
- Azure AI Search with hybrid BM25 + dense vector retrieval
- Semantic ranking (L2 reranking) for highest relevance quality
- Native Azure integration — Managed Identity, Key Vault, RBAC
- Appropriate for: production enterprise deployment

### Namespace Isolation

All knowledge is organized into namespaces. A namespace is a logical partition of the knowledge base:

```
documents/     — General company documents
compliance/    — Regulatory and compliance content
marketing/     — Brand, campaigns, go-to-market
strategy/      — Internal strategy, roadmaps
client/{slug}/ — Per-client isolated namespace
sandbox/{user}/ — Per-user demo sandbox
meetings/      — Meeting transcripts and summaries
```

Role-based access control enforces namespace boundaries: a `client` role can only search their own namespace; a `marketing` role cannot access `strategy`; an `admin` can access everything.

---

## LLM Routing Architecture

Every chat query is routed through a cost-aware model selection pipeline:

```
Query arrives at /api/chat
  ↓
Zero-token pre-filter (deterministic classification)
  ├── factual query? → deterministic answer, no LLM call
  └── navigation query? → deterministic answer, no LLM call
  ↓
Semantic cache check
  └── similar query answered recently? → cached response, no LLM call
  ↓
Knowledge base search (all active tiers)
  ↓
Budget check (daily per-user/per-role limit)
  ├── budget OK → model router
  └── budget exhausted → local LLM fallback (Ollama / chunk display)
  ↓
Model router (role + namespace + data residency → tier selection)
  Tier 1: Gemini 2.0 Flash   ($0.075/1M tokens — demo, member roles)
  Tier 2: GPT-4.1-nano       ($0.10/1M — client, developer roles)
  Tier 3: Claude Haiku       ($0.80/1M — cro, founder, admin + compliance NS)
  Tier 4: GPT-4.1-mini       ($0.40/1M — balanced fallback)
  Tier 5: Azure GPT-4o       ($2.50/1M — data residency required)
  Tier 6: Claude Sonnet      ($3.00/1M — highest quality, final fallback)
  ↓
Response + sources returned
  ↓
Semantic cache populated (for future similar queries)
```

---

## Security Architecture

### Authentication

MAO supports two authentication methods simultaneously:

1. **HTTP Basic Auth** — Username/password stored as bcrypt hashes in environment config. Used by API clients and CLI scripts. Credentials delivered via `Authorization: Basic` header.

2. **Azure Entra SSO (OIDC)** — Microsoft Entra ID via MSAL OIDC flow. Employees authenticate with their Microsoft identity (`@inflexis.ai` accounts). On successful authentication, user's UPN is mapped to a MAO role, and a signed session cookie is issued. Session expiry: 8 hours.

### Secrets Management

Zero plaintext credentials in application code or configuration files:
- All secrets stored in Azure Key Vault
- Application accesses Key Vault via Managed Identity (no credential required)
- Container App environment variables reference Key Vault secrets via `secretref:`
- Local development: `.env` file (never committed to version control)

### PII Masking

Before any document content reaches the vector store or any LLM prompt:
1. PII patterns detected: names, email addresses, phone numbers, SSNs, account numbers, dates of birth
2. Detected PII replaced with reversible tokens: `[NAME_1]`, `[EMAIL_1]`, `[PHONE_1]`
3. Token-to-value mapping held in secure in-memory trust layer (not stored to disk)
4. LLM responses are de-tokenized before delivery to users (where context permits)

---

*For integration details, see [Azure Services Reference](../integrations/azure.md) and [API Reference](../integrations/api-reference.md)*
