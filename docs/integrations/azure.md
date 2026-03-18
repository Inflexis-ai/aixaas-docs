# Azure Services Integration Reference

AIXaaS™ is designed as an Azure-native platform. All production services are provisioned in Azure, and the platform uses Managed Identity to access Azure resources without any stored credentials.

---

## Required Azure Services

### Azure Container Apps
The primary compute platform. MAO runs as a containerized FastAPI application.

- **Resource:** `mao-platform` (Container App)
- **Environment:** `inflexis-env`
- **Scaling:** 1–3 replicas (configurable)
- **Ingress:** External, HTTPS only
- **Custom domains:** `app.inflexis.ai`, `app.aixaas.org`

```bash
# Deploy or update the Container App
az containerapp update \
  --name mao-platform \
  --resource-group inflexis-rg \
  --image inflexisacr.azurecr.io/mao:latest
```

### Azure Container Registry (ACR)
Private Docker image registry. All container images are built and pushed here via GitHub Actions CI/CD.

- **Registry:** `inflexisacr.azurecr.io`
- **Image:** `inflexisacr.azurecr.io/mao:{sha}`
- **Auth:** Service principal via GitHub Actions; application uses Managed Identity

### Azure Key Vault
All application secrets are stored here. The Container App accesses Key Vault via system-assigned Managed Identity — no API keys stored in environment variables.

- **Vault:** `inflexis-keyvault`
- **Access:** Key Vault Secrets Officer role on Managed Identity

Secrets stored:
```
anthropic-api-key         — Anthropic Claude API key
azure-openai-key          — Azure OpenAI API key
azure-openai-endpoint     — Azure OpenAI endpoint URL
azure-speech-key          — Azure Speech Services key
gemini-api-key            — Google Gemini API key
appinsights-connection-string  — Application Insights
auth-users                — HTTP Basic Auth credentials (bcrypt)
auth-roles                — Role assignments per user
entra-client-id           — Entra SSO app registration client ID
entra-client-secret       — Entra SSO client secret
entra-tenant-id           — Azure tenant ID
```

### Azure AI Search
Production knowledge base — hybrid BM25 + dense vector + semantic ranking.

- **Service:** `inflexis-search`
- **Index:** `mao-knowledge` (or per-namespace indexes)
- **Recommended tier:** Standard S1 for semantic ranking + SLA

The platform uses Azure AI Search as the Tier 4 knowledge store, providing the highest quality hybrid retrieval available. The Tier 1–3 stores (keyword, Qdrant, Pinecone) remain active as fallbacks.

### Azure Blob Storage
Used for document archive, meeting recording storage, and backup.

- **Account:** `inflexismaostorage`
- **Containers:** `documents`, `recordings`, `backup`
- **Access:** Managed Identity

### Azure Files (Persistent Volume)
Provides persistent storage for the Container App's `/app/data` directory. Without this volume, all ingested data is lost when the container restarts.

- **Storage Account:** `inflexismaostorage`
- **Share:** `mao-data`
- **Mount point:** `/app/data`
- **Access mode:** ReadWrite

This volume contains:
- `/app/data/knowledge_base/` — LocalFallbackStore keyword chunks (disk-backed)
- `/app/data/adrs/` — ADR records (pipeline + change control)
- `/app/data/registrations/` — User registration records
- `/app/data/meetings/` — Meeting records

### Azure Entra ID (SSO)
Microsoft identity integration for employee Single Sign-On.

- **Tenant:** `bryanshawinflexis.onmicrosoft.com`
- **App registration:** `mao-platform` (client ID stored in Key Vault)
- **Authentication flow:** OIDC via MSAL → signed session cookie (8-hour expiry)
- **Group-to-role mapping:** Entra security groups map to MAO roles

| Entra Group | MAO Role |
|---|---|
| `MAO-Admins` | `admin` |
| `MAO-Founders` | `founder` |
| `MAO-CRO` | `cro` |
| `MAO-Marketing` | `marketing` |

### Azure Application Insights
Performance monitoring, error tracking, and usage telemetry.

- **Resource:** `mao-insights`
- **Activation:** Set `APPLICATIONINSIGHTS_CONNECTION_STRING` environment variable
- **Free tier:** 5GB/month data ingestion

Telemetry captured:
- HTTP request latency and response codes
- LLM API call duration and token usage
- Knowledge base search latency per tier
- ADR submission and approval events
- Authentication events (SSO + Basic Auth)

### Azure AI Document Intelligence
Enterprise-grade PDF and DOCX parsing — extracts tables as structured Markdown, preserves document layout and reading order.

- **Model:** `prebuilt-layout`
- **Configuration:** `AZURE_DOC_INTEL_KEY` + `AZURE_DOC_INTEL_ENDPOINT`
- **Fallback:** PyMuPDF → python-docx → plaintext extraction
- **Cost:** $1.50 per 1,000 pages; 500 pages/month free

### Azure Speech Services
On-Azure audio transcription for meeting recordings. Data never leaves the Azure tenant.

- **Configuration:** `AZURE_SPEECH_KEY` + `AZURE_SPEECH_REGION`
- **Fallback:** Deepgram Nova-3 → OpenAI Whisper
- **Cost:** $1.00/audio hour; 5 hours/month free (F0 tier)

---

## Deployment Topology

```
Internet
  ↓ HTTPS
Azure Front Door (optional — CDN + WAF)
  ↓
Azure Container Apps (mao-platform)
  ↓ Managed Identity ↓
  ├── Azure Key Vault (secrets)
  ├── Azure AI Search (knowledge base)
  ├── Azure Blob Storage (documents, recordings)
  ├── Azure Files (persistent /app/data volume)
  ├── Azure Application Insights (telemetry)
  ├── Azure Entra ID (SSO authentication)
  ├── Azure AI Document Intelligence (PDF parsing)
  └── Azure Speech Services (audio transcription)
```

---

## Environment Variables Reference

Variables injected at runtime by Container Apps from Key Vault secrets:

| Variable | Source | Purpose |
|---|---|---|
| `AZURE_KEYVAULT_URL` | Direct config | Key Vault URL — all other secrets fetched from here |
| `ANTHROPIC_API_KEY` | Key Vault | Claude API |
| `AZURE_OPENAI_KEY` | Key Vault | Azure OpenAI API |
| `AZURE_OPENAI_ENDPOINT` | Key Vault | Azure OpenAI endpoint |
| `AZURE_OPENAI_DEPLOYMENT` | Direct config | Deployment name (e.g. `gpt-4o`) |
| `AZURE_SPEECH_KEY` | Key Vault | Speech Services |
| `AZURE_SPEECH_REGION` | Direct config | e.g. `eastus` |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` | Key Vault | App Insights |
| `AUTH_USERS` | Key Vault | HTTP Basic Auth credentials |
| `AUTH_ROLES` | Key Vault | Role assignments |
| `ENTRA_CLIENT_ID` | Key Vault | SSO app registration |
| `ENTRA_CLIENT_SECRET` | Key Vault | SSO client secret |
| `ENTRA_TENANT_ID` | Key Vault | Azure tenant |
| `ENTRA_AUTHORITY` | Direct config | `https://login.microsoftonline.com/{tenant_id}` |
| `ENTRA_REDIRECT_URI` | Direct config | `https://app.inflexis.ai/auth/callback` |

---

*For deployment instructions, see [Deployment Guide](../guides/deployment.md)*
