# Deployment Guide — AIXaaS™ / MAO Platform

This guide covers deploying the MAO platform to Azure Container Apps. The deployment pipeline is fully automated via GitHub Actions — a `git push` to `main` triggers a build and redeploy in approximately 5 minutes.

---

## Prerequisites

- Azure subscription with the following resources provisioned:
  - Resource Group: `inflexis-rg`
  - Container Apps Environment: `inflexis-env`
  - Azure Container Registry: `inflexisacr`
  - Azure Key Vault: `inflexis-keyvault`
  - Azure AI Search: `inflexis-search`
  - Azure Storage Account: `inflexismaostorage`
- GitHub repository with the following secrets configured:
  - `AZURE_CREDENTIALS` — Service principal JSON for GitHub Actions Azure login
  - `AUTH_USERS`, `AUTH_ROLES` — HTTP Basic Auth credentials and role assignments
  - `ENTRA_CLIENT_ID`, `ENTRA_CLIENT_SECRET`, `ENTRA_TENANT_ID` — Entra SSO
  - `ANTHROPIC_API_KEY` — Anthropic Claude API key
  - `AZURE_OPENAI_KEY`, `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_DEPLOYMENT` — Azure OpenAI
  - `AZURE_SPEECH_KEY`, `AZURE_SPEECH_REGION` — Azure Speech Services
  - `GEMINI_API_KEY` — Google Gemini
  - `APPINSIGHTS_CONNECTION_STRING` — Application Insights
  - `DEMO_WEBHOOK_URL` — Webhook URL for notifications

---

## Deployment Pipeline

### Automatic Deployment (CI/CD)

Every push to the `main` branch triggers the following pipeline:

```
1. Checkout source code
2. Login to Azure via service principal (AZURE_CREDENTIALS)
3. Login to Azure Container Registry
4. Build Docker image
5. Push image with two tags:
   - inflexisacr.azurecr.io/mao:latest
   - inflexisacr.azurecr.io/mao:{git-sha}
6. Sync secrets to Container App (Key Vault references)
7. Update Container App with new image SHA
8. Set environment variables (references Key Vault secrets via secretref:)
9. Ensure Azure Files volume is mounted at /app/data
10. Confirm Managed Identity has Key Vault access
11. Output deployment URL
```

### Manual Deployment (Workflow Dispatch)

The deployment workflow can also be triggered manually from GitHub Actions → `deploy-azure.yml` → "Run workflow".

### Rollback

To roll back to a previous deployment, update the Container App image to a specific SHA:

```bash
az containerapp update \
  --name mao-platform \
  --resource-group inflexis-rg \
  --image inflexisacr.azurecr.io/mao:{previous-sha}
```

Previous image SHAs are available in:
- GitHub Actions run history (each run shows the SHA deployed)
- Azure Container Registry → `mao` repository → Tags

---

## Azure Files Persistent Volume

The most critical deployment detail: without the Azure Files volume, all ingested knowledge base data is lost on every container restart or redeploy.

The deployment pipeline automatically:
1. Links `inflexismaostorage` to the Container Apps environment as `mao-persistent-data`
2. Mounts the `mao-data` file share to `/app/data` inside the container

This means the knowledge base (168K+ chunks), ADR records, meeting data, and user registrations all survive restarts and deployments.

To verify the mount is active:

```bash
az containerapp show \
  --name mao-platform \
  --resource-group inflexis-rg \
  --query "properties.template.volumes"
```

---

## Custom Domain Setup

Both `app.inflexis.ai` and `app.aixaas.org` use Azure-managed TLS with auto-renewing certificates.

To add a new custom domain:

```bash
# Add domain binding
az containerapp hostname add \
  --name mao-platform \
  --resource-group inflexis-rg \
  --hostname app.yourclient.com

# Get the verification token
az containerapp hostname list \
  --name mao-platform \
  --resource-group inflexis-rg

# After DNS verification, bind the managed certificate
az containerapp ssl upload \
  --name mao-platform \
  --resource-group inflexis-rg \
  --hostname app.yourclient.com \
  --certificate-type managed
```

---

## Health Verification

After deployment, verify the platform is healthy:

```bash
# Check health endpoint
curl -u 'user:pass' https://app.inflexis.ai/health

# Expected response
{
  "status": "healthy",
  "kb_ready": true,
  "storage_mode": "azure+keyword"
}
```

Note: `kb_ready` may be `false` for the first 30 seconds after startup while the keyword store loads from disk. This is normal.

```bash
# Verify Application Insights telemetry
az monitor app-insights query \
  --app mao-insights \
  --resource-group inflexis-rg \
  --analytics-query "requests | take 3 | project timestamp, name, resultCode"
```

---

## Scaling Configuration

Current scaling policy:
- Minimum replicas: 1 (always warm — no cold start)
- Maximum replicas: 3
- Scale trigger: HTTP concurrency

To adjust:
```bash
az containerapp update \
  --name mao-platform \
  --resource-group inflexis-rg \
  --min-replicas 1 \
  --max-replicas 5
```

---

## Local Development

For local development without Azure dependencies:

```bash
# Clone repository (private — access required)
git clone https://github.com/inflexis-ai/mao-platform.git
cd mao-platform

# Install dependencies
pip install -r requirements-prod.txt

# Configure .env (copy from .env.example)
cp .env.example .env
# Add ANTHROPIC_API_KEY at minimum

# Start server
python server.py
# → http://localhost:8000
```

The local server uses the keyword fallback store by default. Azure services are used when their respective keys are present in `.env`.

---

*For API usage examples, see [API Reference](../integrations/api-reference.md)*
