<div align="center">

# AIXaaS™ Platform Documentation

### Enterprise AI Orchestration — Governed, Auditable, Multi-Agent Intelligence

**[inflexis.ai](https://inflexis.ai)** · **[Live Demo](https://app.inflexis.ai)** · [Get in Touch](mailto:bryan.shaw@inflexis.ai)

</div>

---

## What Is AIXaaS™?

**AI-as-a-Service (AIXaaS™)** is Inflexis's enterprise AI orchestration platform. It deploys in your Azure tenant, connects to your documents and data, and gives every role in your organization governed access to AI — with a full audit trail, role-based permissions, and zero IP exposure to third parties.

Unlike a chatbot or a simple API wrapper, AIXaaS™ is an orchestration system:
- Multiple specialized agents with distinct roles and access scopes
- A structured knowledge pipeline that processes 20+ file formats into a searchable enterprise KB
- 23 regulatory compliance frameworks detected at zero token cost
- Every AI instruction change governed by a formal change-control process
- Every agent decision logged as an immutable Architectural Decision Record (ADR)

---

## Documentation Index

### Getting Started
- [Platform Overview](docs/platform-overview.md)
- [Architecture Reference](docs/architecture/overview.md)
- [Role-Based Access Control](docs/architecture/rbac.md)
- [Deployment Guide](docs/guides/deployment.md)

### Core Capabilities
- [Knowledge Base & RAG Pipeline](docs/architecture/rag-pipeline.md)
- [Compliance Engine](docs/architecture/compliance-engine.md)
- [Meeting Intelligence](docs/architecture/meeting-intelligence.md)
- [ADR Change Control System](docs/architecture/adr-system.md)

### Integrations
- [Azure Services Reference](docs/integrations/azure.md)
- [LLM Provider Configuration](docs/integrations/llm-providers.md)
- [API Reference](docs/integrations/api-reference.md)
- [MCP Server (Claude Desktop)](docs/integrations/mcp-server.md)

### Guides
- [Ingesting Documents](docs/guides/ingesting-documents.md)
- [Configuring Roles & Namespaces](docs/guides/roles-namespaces.md)
- [Multi-Tenant Setup](docs/guides/multi-tenant.md)

---

## Quick Architecture Reference

```
┌─────────────────────────────────────────────────────────────────────┐
│                         AIXaaS™ / MAO Platform                       │
├──────────────────┬──────────────────┬──────────────────────────────┤
│  INGEST PIPELINE │  KNOWLEDGE BASE  │      AGENT LAYER             │
│                  │                  │                              │
│  PDF  DOCX  PPTX │  Keyword Index   │  8 Role Agents               │
│  XLSX MP3  WAV   │  Semantic Vector │  Compliance Engine           │
│  YouTube  Web    │  Azure AI Search │  Meeting Intelligence        │
│  Markdown  RTF   │  Pinecone        │  ADR Change Control          │
│                  │                  │                              │
│  ↓ Parser Chain  │  4-Tier Hybrid   │  ADR Audit at Every Step     │
│  ↓ PII Masking   │  BM25 + Vector   │  Guardrails + Budget Caps    │
│  ↓ Compliance    │  Disk Persisted  │  Semantic Response Cache     │
│  ↓ Chunking      │  Always Warm     │  Role + Namespace Scoping    │
│  ↓ Store         │                  │                              │
└──────────────────┴──────────────────┴──────────────────────────────┘
          ↕ Azure Container Apps · Key Vault · Entra SSO ↕
```

---

## Supported File Types

| Category | Formats |
|---|---|
| Documents | PDF, DOCX, PPTX, XLSX, Markdown, RTF, TXT |
| Apple formats | Pages, Numbers, Keynote |
| Structured data | CSV, TSV, JSON |
| Web content | HTML, XML, direct URL scraping |
| Audio & Video | MP3, MP4, WAV, FLAC, WEBM |
| Online media | YouTube URLs (transcript extraction), web pages |

---

## Compliance Framework Coverage

AIXaaS™ detects content triggering 23 regulatory frameworks — at **zero LLM token cost** through deterministic pattern matching:

`GDPR` · `HIPAA` · `PCI-DSS` · `SOC 2` · `NIST CSF` · `ISO 27001` · `CCPA` · `FedRAMP` · `FISMA` · `FERPA` · `GLBA` · `NERC CIP` · `IEC 62443` · `NIST SP 800-82` · `CMMC` · `ITAR` · `EAR` · `SEC Rule 17a-4` · `FINRA` · `MiFID II` · `Basel III` · `Solvency II` · `DORA`

---

## Role-Based Access Control

8 pre-configured roles with distinct permissions, namespace access, and agent directives:

| Role | Use Case |
|---|---|
| `admin` | Full platform access, ADR approvals, user management |
| `founder` | Executive access to all data, strategic namespaces |
| `cro` | Sales pipeline, meeting intelligence, deal data |
| `marketing` | Brand content, campaigns, marketing sector |
| `developer` | Technical documentation, API access |
| `member` | General knowledge base access |
| `client` | Isolated client namespace — only their own data |
| `demo` | Sandboxed demo environment — no production data |

---

## LLM Provider Support

AIXaaS™ routes queries to the **cheapest provider meeting quality requirements**, with full fallback chain:

| Tier | Provider | Cost (per 1M tokens) | Use Case |
|---|---|---|---|
| 0 | Ollama (local) | Free | Dev/air-gapped environments |
| 1 | Google Gemini 2.0 Flash | $0.075 / $0.30 | Demo users, high-volume queries |
| 2 | Azure OpenAI GPT-4.1-nano | $0.10 / $0.40 | Standard member queries |
| 3 | Anthropic Claude Haiku | $0.80 / $4.00 | Internal team, compliance-sensitive |
| 4 | Azure OpenAI GPT-4.1-mini | $0.40 / $1.60 | Balanced quality/cost |
| 5 | Azure OpenAI GPT-4o | $2.50 / $10.00 | Enterprise data residency required |
| 6 | Anthropic Claude Sonnet | $3.00 / $15.00 | Highest quality, last resort |

Routing policy is configurable per role and namespace. Data residency requirements (e.g. OT/ICS clients) force Azure-only routing at Tier 5.

---

## Azure Infrastructure

AIXaaS™ is designed for production deployment on Microsoft Azure:

| Service | Purpose |
|---|---|
| Azure Container Apps | Auto-scaling application hosting, managed ingress |
| Azure Container Registry | Private Docker image store, SHA-tagged deployments |
| Azure Key Vault | All secrets centralized — zero plaintext credentials in config |
| Azure AI Search | Production vector + BM25 hybrid search, semantic ranking |
| Azure Blob Storage | Document archive, recording storage, backup |
| Azure Files | Persistent volume — data survives container restarts |
| Azure Entra ID | SSO authentication, group-to-role mapping |
| Azure Application Insights | Performance tracing, error tracking, live metrics |
| Azure AI Document Intelligence | Table-preserving PDF/DOCX parsing (enterprise tier) |
| Azure Speech Services | On-Azure audio transcription — data never leaves tenant |

---

## Deployment

AIXaaS™ deploys via GitHub Actions to Azure Container Apps in approximately 5 minutes per push:

```
git push origin main
  → GitHub Actions triggered
  → Docker image built and pushed to Azure Container Registry
  → Container App updated with new image SHA
  → Health check confirmed
  → Live at app.inflexis.ai
```

For detailed deployment instructions, see [Deployment Guide](docs/guides/deployment.md).

---

## Getting Access

AIXaaS™ is currently in enterprise pilot. To request a demonstration or discuss deployment for your organization:

📧 **[bryan.shaw@inflexis.ai](mailto:bryan.shaw@inflexis.ai)**
🌐 **[inflexis.ai](https://inflexis.ai)**

---

<div align="center">
<sub>© 2026 Inflexis · AIXaaS™ is a trademark of Inflexis · Documentation is provided for informational purposes. Platform source code is proprietary.</sub>
</div>
