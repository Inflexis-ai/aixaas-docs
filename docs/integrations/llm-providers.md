# LLM Provider Configuration

AIXaaS™ uses a cost-aware model router that selects the cheapest LLM provider meeting the quality and compliance requirements for each query. All providers are optional — the system degrades gracefully when providers are not configured.

---

## Provider Tier Summary

| Tier | Provider | Model | Input ($/1M) | Output ($/1M) | Data Residency |
|---|---|---|---|---|---|
| 0 | Ollama (local) | Any local model | Free | Free | ✅ On-premise |
| 1 | Google Gemini | gemini-2.0-flash | $0.075 | $0.30 | ❌ Google Cloud |
| 2 | Azure OpenAI | gpt-4.1-nano | $0.10 | $0.40 | ✅ Azure tenant |
| 3 | Anthropic | claude-haiku-4-5 | $0.80 | $4.00 | ❌ Anthropic |
| 4 | Azure OpenAI | gpt-4.1-mini | $0.40 | $1.60 | ✅ Azure tenant |
| 5 | Azure OpenAI | gpt-4o | $2.50 | $10.00 | ✅ Azure tenant |
| 6 | Anthropic | claude-sonnet-4-5 | $3.00 | $15.00 | ❌ Anthropic |

---

## Role-Based Routing Policy

The minimum tier is determined by the user's role. Cheaper tiers are used when quality requirements are met:

| Role | Minimum Tier | Provider |
|---|---|---|
| `demo` | 1 | Gemini 2.0 Flash |
| `member` | 1 | Gemini 2.0 Flash |
| `client` | 2 | GPT-4.1-nano (Azure) |
| `marketing` | 2 | GPT-4.1-nano (Azure) |
| `developer` | 2 | GPT-4.1-nano (Azure) |
| `cro` | 3 | Claude Haiku |
| `founder` | 3 | Claude Haiku |
| `admin` | 3 | Claude Haiku |

Compliance-sensitive namespaces (`compliance`, `audit`, `documents`) automatically enforce a minimum of Tier 3 (Claude Haiku), regardless of role.

---

## Data Residency Enforcement

For clients with regulatory requirements mandating that data not leave a specific Azure region, the model router enforces Azure-only routing:

```
require_data_residency=True
  → Skips: Gemini (Tier 1), Claude Haiku (Tier 3), Claude Sonnet (Tier 6)
  → Uses only: Azure OpenAI Tier 2, 4, or 5
  → All LLM calls stay within Azure tenant
```

This is automatically applied when the namespace is in a compliance set and the role is `founder` or `admin`. It can also be forced per request.

---

## Provider Configuration

### Anthropic Claude (Tiers 3 & 6)

```bash
# Key Vault (production)
az keyvault secret set --vault-name inflexis-keyvault --name anthropic-api-key --value "sk-ant-..."

# Local development (.env)
ANTHROPIC_API_KEY=sk-ant-...
```

Models used:
- Tier 3: `claude-haiku-4-5-20251001` (configurable via `CHAT_MODEL` env var)
- Tier 6: `claude-sonnet-4-5-20251001`

### Azure OpenAI (Tiers 2, 4 & 5)

Requires Azure OpenAI Service resource in your Azure subscription. Request access at [Azure Portal → Create → Azure OpenAI](https://portal.azure.com).

```bash
# Key Vault (production)
az keyvault secret set --vault-name inflexis-keyvault --name azure-openai-key --value "..."
# Environment variable (direct config)
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o
```

For multi-deployment routing (nano, mini, 4o), configure per-deployment overrides:
```
AZURE_OPENAI_DEPLOYMENT_GPT_4_1_NANO=gpt-4-1-nano-deployment
AZURE_OPENAI_DEPLOYMENT_GPT_4_1_MINI=gpt-4-1-mini-deployment
AZURE_OPENAI_DEPLOYMENT_GPT_4O=gpt-4o-deployment
```

### Google Gemini (Tier 1)

Obtain an API key at [aistudio.google.com](https://aistudio.google.com/app/apikey).

```bash
# Key Vault (production)
az keyvault secret set --vault-name inflexis-keyvault --name gemini-api-key --value "AIza..."
# Local development (.env)
GEMINI_API_KEY=AIza...
```

### Ollama / Local LLM (Tier 0)

For air-gapped or development deployments. Requires Ollama running locally or on a Docker container:

```bash
# Install Ollama
curl https://ollama.ai/install.sh | sh

# Pull a model
ollama pull mistral

# MAO automatically detects and uses Ollama when available
# No configuration required — detected via HTTP ping to localhost:11434
```

---

## Fallback Behavior

If a provider call fails (API error, timeout, or key not configured), the router automatically falls back to the next tier. This cascades until a response is obtained.

If all providers fail, the system returns the most relevant raw knowledge base chunk as a structured response — no LLM inference, but still grounded in your data.

```
Tier 3 fails → try Tier 4 → try Tier 5 → try Tier 6 → chunk display
```

No query ever returns a generic "I don't know" — the system always returns the best available answer from your knowledge base, even without LLM inference.

---

## Cost Optimization

**Semantic cache** — Queries within ~85% semantic similarity of a recent query are served from cache, with zero LLM cost. Cache hit rates of 30–60% are typical in production environments where the same team queries similar information repeatedly.

**Zero-token pre-filter** — Factual and navigation queries (date questions, UI navigation questions) are answered deterministically without any LLM call.

**Token budget governance** — Per-user and per-role daily token budgets prevent runaway costs. When a budget is exceeded, queries fall back to local LLM or chunk display.

---

*For the full architecture of the model routing system, see [Architecture Overview](../architecture/overview.md)*
