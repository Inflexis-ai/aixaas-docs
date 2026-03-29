<div align="center">

<img src="https://img.shields.io/badge/AIXaaS™-Documentation-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="AIXaaS Documentation"/>

# AIXaaS™ Platform Documentation

**Enterprise AI Orchestration — Governed · Auditable · Azure-Native**

[![Platform Status](https://img.shields.io/badge/platform-live-brightgreen?style=flat-square)](https://app.inflexis.ai)
[![Azure](https://img.shields.io/badge/cloud-Azure%20Container%20Apps-0078D4?style=flat-square&logo=microsoftazure)](https://azure.microsoft.com/en-us/products/container-apps)
[![Python](https://img.shields.io/badge/python-3.12-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109-009688?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Anthropic](https://img.shields.io/badge/Claude-Haiku%20%7C%20Sonnet-8B5CF6?style=flat-square)](https://www.anthropic.com/)
[![License](https://img.shields.io/badge/license-Proprietary-red?style=flat-square)](https://inflexis.ai)
[![Docs](https://img.shields.io/badge/docs-this%20repo-blue?style=flat-square)](https://github.com/inflexis-ai/aixaas-docs)

<br/>

[**Live Demo**](https://app.inflexis.ai) · [**API Reference**](docs/integrations/api-reference.md) · [**Examples**](https://github.com/inflexis-ai/mao-examples) · [**Get Access**](mailto:bryan.shaw@inflexis.ai)

</div>

---

## What Is AIXaaS™?

**AI-as-a-Service (AIXaaS™)** is Inflexis's enterprise AI orchestration platform. It deploys in your tenant environment or can be hosted on ours as a iSaaS product, connects to your documents and operational data, and gives every role in your organization **governed, auditable AI access** — with a full change-control trail, role-based permissions, and zero IP exposure to third parties.

Unlike a chatbot or a simple API wrapper, AIXaaS™ is a complete **orchestration system**:

```
Your Documents + Data
        ↓
  Universal Ingestor (20+ file types)
        ↓
  PII Masking → Compliance Scan → Chunk → 4-Tier Store
        ↓
  8-Role RBAC + Namespace Governance
        ↓
  Cost-Aware Model Routing (6 LLM providers, cheapest first)
        ↓
  Grounded Response + Source Citations + ADR Audit Record
        ↓
  Your Team (role-appropriate answer, in their namespace)
```

Every step is logged. Every agent instruction change is version-controlled. Every provider has a fallback.

---

## Documentation

### 🚀 Getting Started
| | |
|---|---|
| [Platform Overview](docs/platform-overview.md) | What AIXaaS™ is, the problems it solves, and core design principles |
| [Architecture Reference](docs/architecture/overview.md) | Full system topology, storage tiers, security model |
| [Deployment Guide](docs/guides/deployment.md) | Azure Container Apps CI/CD, rollback, custom domains, health checks |

### 🔌 Integrations
| | |
|---|---|
| [Azure Services Reference](docs/integrations/azure.md) | All Azure resources, configuration, CLI commands |
| [LLM Provider Configuration](docs/integrations/llm-providers.md) | 6-tier routing, role policies, data residency, fallback behavior |
| [API Reference](docs/integrations/api-reference.md) | All endpoints with curl examples, request/response schemas |
| [MCP Server](docs/integrations/mcp-server.md) | Claude Desktop integration — 8 tools exposed |

### 📐 Architecture Deep Dives
| | |
|---|---|
| [RAG Pipeline](docs/architecture/rag-pipeline.md) | Ingest → chunk → store → retrieve — all 5 pipeline stages |
| [Compliance Engine](docs/architecture/compliance-engine.md) | 23 frameworks, zero-token detection, framework reference |
| [Meeting Intelligence](docs/architecture/meeting-intelligence.md) | Transcription, SPICED scoring, delivery pipeline |
| [ADR Change Control](docs/architecture/adr-system.md) | Agent governance, diff review, archive/rollback |

### 📖 Guides
| | |
|---|---|
| [Ingesting Documents](docs/guides/ingesting-documents.md) | Upload documents, batch ingest, supported file types |
| [Roles & Namespaces](docs/guides/roles-namespaces.md) | Configure RBAC, namespace isolation, sector governance |
| [Multi-Tenant Setup](docs/guides/multi-tenant.md) | Per-client namespace isolation, white-label deployment |

---

## Architecture at a Glance

<details>
<summary><strong>📊 System Topology</strong></summary>

```
┌─────────────────────────────────────────────────────────────────────┐
│                         AIXaaS™ / MAO Platform                       │
│                        Azure Container Apps                          │
├──────────────────┬──────────────────┬──────────────────────────────┤
│  INGEST PIPELINE │  KNOWLEDGE BASE  │       AGENT LAYER            │
│                  │                  │                              │
│  PDF  DOCX  PPTX │  T1: Keyword     │  8-Role RBAC                 │
│  XLSX MP3  MP4   │  T2: Qdrant/vec  │  Compliance Engine (23 fw)   │
│  YouTube  Web    │  T3: Pinecone    │  Meeting Intelligence        │
│  Markdown  RTF   │  T4: Azure Search│  ADR Change Control          │
│  CSV  JSON  HTML │                  │  Semantic Cache (30-60% ↓)   │
│                  │  4-Tier Hybrid   │  Token Budget Guardrails      │
│  → Parser Chain  │  BM25 + Vector   │  6-Tier Model Router         │
│  → PII Mask      │  Always Warm     │  MCP Server (8 tools)        │
│  → Comply Scan   │  Disk Persisted  │                              │
│  → Chunk + Store │                  │                              │
└──────────────────┴──────────────────┴──────────────────────────────┘
          ↕ Azure Key Vault · Entra SSO · App Insights ↕
```

</details>

<details>
<summary><strong>🏷️ Compliance Frameworks (23 — Zero Token Cost)</strong></summary>

<br/>

`GDPR` `HIPAA` `PCI-DSS` `SOC 2` `NIST CSF` `ISO 27001` `CCPA` `FedRAMP` `FISMA` `FERPA` `GLBA` `NERC CIP` `IEC 62443` `NIST SP 800-82` `CMMC` `ITAR` `EAR` `SEC 17a-4` `FINRA` `MiFID II` `Basel III` `Solvency II` `DORA`

All 23 detected via deterministic pattern matching — **no LLM call, no API cost, instant classification**.

</details>

<details>
<summary><strong>👤 8-Role RBAC</strong></summary>

<br/>

| Role | Min LLM Tier | Namespace Access | Use Case |
|---|---|---|---|
| `admin` | Tier 3 | All | Full platform control, ADR approvals |
| `founder` | Tier 3 | All | Executive access, strategic namespaces |
| `cro` | Tier 3 | Sales + meetings | Pipeline, SPICED scoring, deal data |
| `marketing` | Tier 2 | Brand + marketing | Campaigns, content, brand sector |
| `developer` | Tier 2 | Technical + docs | API, codebase, documentation |
| `member` | Tier 1 | General | Standard knowledge base access |
| `client` | Tier 2 | Own namespace only | Isolated client data |
| `demo` | Tier 1 | Sandbox only | Demo environment, no production data |

</details>

---

## Quick Start — Query the API

```python
import requests

BASE_URL = "https://app.inflexis.ai"
AUTH = ("your-username", "your-password")

# Search the knowledge base
response = requests.post(f"{BASE_URL}/api/chat", auth=AUTH, json={
    "query": "What compliance frameworks apply to our industrial documents?",
    "namespace": "compliance",
    "top_k": 8,
})

result = response.json()
print(result["response"])
# → Grounded answer with sources, model used, and budget status
```

See [mao-examples](https://github.com/inflexis-ai/mao-examples) for full scripts and Jupyter notebooks.

---

## Ecosystem & Related Projects

AIXaaS™ builds on, integrates with, and complements the best open-source AI infrastructure:

| Project | Stars | Relationship |
|---|---|---|
| [anthropics/anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) | [![Stars](https://img.shields.io/github/stars/anthropics/anthropic-sdk-python?style=flat-square)](https://github.com/anthropics/anthropic-sdk-python) | Primary LLM SDK — MAO uses Claude Haiku + Sonnet |
| [anthropics/anthropic-cookbook](https://github.com/anthropics/anthropic-cookbook) | [![Stars](https://img.shields.io/github/stars/anthropics/anthropic-cookbook?style=flat-square)](https://github.com/anthropics/anthropic-cookbook) | Reference patterns for agent design |
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | [![Stars](https://img.shields.io/github/stars/modelcontextprotocol/servers?style=flat-square)](https://github.com/modelcontextprotocol/servers) | MCP ecosystem — MAO ships its own 8-tool MCP server |
| [microsoft/autogen](https://github.com/microsoft/autogen) | [![Stars](https://img.shields.io/github/stars/microsoft/autogen?style=flat-square)](https://github.com/microsoft/autogen) | Multi-agent framework — MAO adds enterprise governance AutoGen lacks |
| [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | [![Stars](https://img.shields.io/github/stars/langchain-ai/langgraph?style=flat-square)](https://github.com/langchain-ai/langgraph) | DAG orchestration — MAO's pipeline uses same pattern, without the overhead |
| [joaomdmoura/crewAI](https://github.com/joaomdmoura/crewAI) | [![Stars](https://img.shields.io/github/stars/joaomdmoura/crewAI?style=flat-square)](https://github.com/joaomdmoura/crewAI) | Role-based agent crews — MAO's RBAC is the enterprise governance layer |
| [BerriAI/litellm](https://github.com/BerriAI/litellm) | [![Stars](https://img.shields.io/github/stars/BerriAI/litellm?style=flat-square)](https://github.com/BerriAI/litellm) | LLM routing — MAO's model router adds per-role cost governance |
| [qdrant/qdrant](https://github.com/qdrant/qdrant) | [![Stars](https://img.shields.io/github/stars/qdrant/qdrant?style=flat-square)](https://github.com/qdrant/qdrant) | Vector database — MAO's Tier 2 semantic store |
| [run-llama/llama_index](https://github.com/run-llama/llama_index) | [![Stars](https://img.shields.io/github/stars/run-llama/llama_index?style=flat-square)](https://github.com/run-llama/llama_index) | RAG framework reference — MAO's hybrid retrieval draws on these patterns |

---

## Contributing

This is a documentation repository for the AIXaaS™ platform. We welcome:

- **Bug reports** — broken links, outdated information, incorrect CLI commands → [Open an issue](https://github.com/inflexis-ai/aixaas-docs/issues/new?template=bug_report.md)
- **Documentation improvements** — clarity, typos, missing content → [Open a PR](https://github.com/inflexis-ai/aixaas-docs/pulls)
- **Feature documentation requests** — missing docs for a platform capability → [Open an issue](https://github.com/inflexis-ai/aixaas-docs/issues/new?template=feature_request.md)

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Platform Access

AIXaaS™ is currently in enterprise pilot. Contact us to request access or discuss deployment:

📧 **[bryan.shaw@inflexis.ai](mailto:bryan.shaw@inflexis.ai)**
🌐 **[inflexis.ai](https://inflexis.ai)**
💼 **[LinkedIn — Bryan Shaw](https://www.linkedin.com/in/bryanjshaw/)**

---

<div align="center">
<sub>© 2026 Inflexis · AIXaaS™ is a trademark of Inflexis · Documentation is provided for informational purposes · Platform source is proprietary</sub>
</div>
