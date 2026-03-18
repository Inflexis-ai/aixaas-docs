# Platform Overview — AIXaaS™ / MAO Platform

## What Problem Does This Solve?

Enterprises adopting AI face three compounding problems:

1. **Data governance gaps** — AI tools that require uploading sensitive documents to third-party clouds create compliance exposure. GDPR, HIPAA, NERC CIP, IEC 62443, and SOC 2 all have data residency and access control requirements that generic AI SaaS cannot satisfy.

2. **Ungoverned AI behavior** — When AI systems are given access to company data and tools, there is no standard mechanism for tracking what instructions they were given, why decisions were made, or who authorized changes. The result: AI systems that cannot be audited.

3. **Context fragmentation** — Company knowledge lives in PDFs, emails, meeting recordings, spreadsheets, databases, and dozens of SaaS platforms. AI that only accesses one source gives partial answers. AI that accesses everything without structure gives hallucinated answers.

AIXaaS™ / MAO solves all three simultaneously.

---

## Core Design Principle: Aviation Dual-Redundancy

Every layer of MAO is designed with the same principle used in commercial aviation system design: **no single point of failure, with validated fallback at every stage**.

In practice this means:
- Every agent handoff produces an **Architectural Decision Record (ADR)** — an immutable log of what data was processed, what decision was made, and which agent made it
- Every storage tier has a fallback: if cloud vector search is unavailable, semantic local search responds; if that is unavailable, keyword search responds
- Every LLM provider has a fallback chain: if Anthropic API is unreachable, Azure OpenAI responds; if that is unavailable, local Ollama responds
- Every ingest parser has a fallback chain: if Azure Document Intelligence returns an error, PyMuPDF responds; if that fails, plaintext extraction responds

The system degrades gracefully. It never crashes silently.

---

## The 10-Layer Architecture

```
Layer 0   Foundation
          Deterministic prompting, zero-token compliance detection (23 frameworks),
          scripted pre-processing — no LLM API cost for classification

Layer 1   Agent Architecture
          8-role RBAC: admin, founder, cro, marketing, developer, member, demo, client
          Each role has its own instruction set, namespace access scope, and model tier

Layer 2   LLMs & APIs
          6-tier cost-aware routing: Ollama → Gemini → GPT-4.1-nano → Claude Haiku
          → GPT-4.1-mini → Azure GPT-4o → Claude Sonnet
          Data residency enforcement: OT/industrial clients force Azure-only routing

Layer 3   Tool Use & Integration
          Universal Ingestor (20+ file types), Compliance Engine, PII Trust Layer,
          Meeting Intelligence, Azure Document Intelligence, Azure Speech

Layer 4   Agent Frameworks
          FastAPI routes, Jinja2 templates, Claude Agent SDK, MCP server (8+ tools)
          Role-specific agent directives loaded per request from namespace markdowns

Layer 5   Orchestration (DAG)
          Multi-agent pipeline: parse → PII mask → compliance scan → chunk → store
          ADR logged at each stage; pipeline can be inspected per file hash

Layer 6   Memory Management
          4-tier hybrid storage with disk persistence and semantic response cache
          Cache reduces LLM API costs 30–60% on repeated query patterns

Layer 7   Knowledge & RAG
          Hybrid BM25 keyword + dense vector retrieval
          Knowledge sector governance: Strategy, Marketing, Finance, Product
          Namespace isolation: each client's data remains scoped to their namespace

Layer 8   Deployment
          Azure Container Apps, automated CI/CD via GitHub Actions,
          custom domains with Azure-managed TLS, Azure Files persistent volume

Layer 9   Monitoring & Feedback
          ADR audit trail, ingest pipeline log, guardrails (token budget, loop caps,
          timeout enforcement), App Insights performance + error tracking

Layer 10  Security & Governance
          ADR change control with diff review and approval workflow
          Archive/rollback system — every AI instruction change is versioned and recoverable
          PII masking applied before any data reaches vector storage or LLM context
```

---

## Ingest Pipeline

When a document enters the MAO platform, it travels through a five-stage pipeline:

```
Stage 1: PARSE
  File type detected → appropriate parser selected
  Parser chain: Azure Doc Intelligence → PyMuPDF/python-docx → plaintext fallback
  Tables preserved as Markdown; pages mapped to page-level chunks

Stage 2: PII MASK
  PII patterns detected and pseudonymized before any data leaves the organization
  Names, emails, phone numbers, SSNs, account numbers replaced with tokens
  Original mappings held in secure trust layer

Stage 3: COMPLIANCE SCAN
  23 regulatory frameworks checked deterministically (no LLM call)
  Applicable frameworks tagged to document metadata
  Compliance flags propagate to all chunks derived from this document

Stage 4: CHUNK
  Document split into overlapping semantic chunks (~512 tokens)
  Chunk metadata: source, page, file hash, compliance flags, sector, namespace, owner

Stage 5: STORE
  Tier 1 (always): Keyword index — disk-backed, zero dependencies
  Tier 2 (when available): Qdrant semantic vector store — GPU-optional
  Tier 3 (when configured): Pinecone cloud vector store
  Tier 4 (production): Azure AI Search — BM25 + dense vector + semantic ranking
  ADR logged: 5 stage records per file, queryable by hash
```

---

## Knowledge Base Architecture

The MAO knowledge base uses a 4-tier hybrid retrieval system designed for both reliability and search quality:

| Tier | Store | Type | When Active |
|---|---|---|---|
| 1 | LocalFallbackStore | Keyword / BM25 | Always — zero dependencies, disk-backed |
| 2 | LocalSemanticStore | Dense vector (Qdrant + sentence-transformers) | When Qdrant is available locally |
| 3 | PineconeStore | Cloud vector (Pinecone) | When PINECONE_API_KEY configured |
| 4 | AzureSearchStore | Hybrid BM25 + vector + semantic (Azure AI Search) | Production deployment |

On every search, results from all active tiers are merged, deduplicated, and ranked. The best result wins regardless of which tier returned it.

---

## ADR System

Every change to an agent's behavior — any modification to a role directive, namespace instruction, or prompt configuration — must flow through the ADR (Architectural Decision Record) change control system:

1. **Submit** — Any team member proposes a change via the dashboard or API
2. **Review** — Admin receives notification, reviews the diff between current and proposed
3. **Approve/Deny** — Admin approves or denies; denial requires a reason
4. **Archive** — On approval, the previous version is archived with full rollback capability
5. **Apply** — New instruction takes effect immediately

This means no AI agent in the MAO platform can have its behavior modified without a paper trail. Every decision is attributed, timestamped, and recoverable.

---

## Meeting Intelligence

The MAO Meeting Intelligence module captures, transcribes, analyzes, and distributes summaries from every recorded meeting:

1. **Capture** — Meeting recording uploaded manually, via Zoom webhook, or via meeting bot
2. **Transcribe** — Azure Speech Services (primary) or Deepgram Nova-3 with speaker diarization
3. **Analyze** — Claude generates: executive summary, SPICED sales scorecard, per-attendee action items, key decisions, open questions, competitive mentions, sentiment signals, suggested follow-up agenda
4. **Deliver** — Per-attendee email, Slack/Discord notification, HubSpot CRM sync
5. **Store** — Full meeting record ingested into namespace knowledge base, queryable via chat or MCP

---

*This documentation describes the AIXaaS™ / MAO Platform as deployed at [app.inflexis.ai](https://app.inflexis.ai). For access or deployment inquiries: [bryan.shaw@inflexis.ai](mailto:bryan.shaw@inflexis.ai)*
