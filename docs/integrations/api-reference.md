# API Reference — AIXaaS™ / MAO Platform

All endpoints require authentication. Default: HTTP Basic Auth via `Authorization: Basic {base64}` header. Alternatively, authenticate via Azure Entra SSO and use the session cookie.

Base URL: `https://app.inflexis.ai`

---

## Authentication

### HTTP Basic Auth
```bash
curl -u 'username:password' https://app.inflexis.ai/api/chat/status
```

### Session Cookie (after SSO login)
Navigate to `/auth/login` to initiate the Entra SSO flow. After successful authentication, a signed session cookie is issued valid for 8 hours.

---

## Health & Status

### GET /health
Returns platform health and knowledge base status.

```bash
curl -u 'user:pass' https://app.inflexis.ai/health
```

Response:
```json
{
  "status": "healthy",
  "kb_ready": true,
  "storage_mode": "azure+keyword",
  "kb_stats": {
    "keyword_total": 168778,
    "azure_total": 168778
  }
}
```

`kb_ready: false` is normal for the first ~30 seconds after container startup while the keyword store loads from disk.

---

## Chat

### POST /api/chat
Submit a query to the knowledge base. Returns AI-generated response grounded in retrieved context.

```bash
curl -u 'user:pass' \
  -X POST https://app.inflexis.ai/api/chat \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "What are the key compliance requirements for NERC CIP?",
    "namespace": "compliance",
    "top_k": 8
  }'
```

Request body:
```json
{
  "query": "string (required)",
  "namespace": "string (optional — filters search to namespace)",
  "owner": "string (optional — filters to owner's documents)",
  "top_k": 8,
  "demo_mode": false,
  "force_local": false
}
```

Response:
```json
{
  "response": "string — AI-generated answer",
  "sources": [
    {
      "source": "nerc-cip-standards.pdf",
      "score": 0.923,
      "excerpt": "First 150 chars of chunk..."
    }
  ],
  "chunks_used": 8,
  "namespace_searched": "compliance",
  "model": "anthropic/claude-haiku-4-5-20251001",
  "grounded": true,
  "switched_to_local": false,
  "thank_you": "",
  "usage": {
    "used": 12,
    "limit": 100,
    "remaining": 88,
    "date": "2026-03-18"
  }
}
```

### GET /api/chat/status
Returns platform readiness, gate status, and budget usage.

```bash
curl -u 'user:pass' https://app.inflexis.ai/api/chat/status
```

Response:
```json
{
  "api_key_configured": true,
  "kb_total_chunks": 168778,
  "gate_status": "approved",
  "usage": { "used": 12, "limit": 100, "remaining": 88 },
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 600,
  "ready": true,
  "semantic_cache": { "hits": 45, "misses": 120, "size": 38 },
  "message": "Ready"
}
```

### POST /api/chat/correction
Flag an AI response as incorrect and log a correction ADR.

```bash
curl -u 'user:pass' \
  -X POST https://app.inflexis.ai/api/chat/correction \
  -H 'Content-Type: application/json' \
  -d '{
    "original_query": "What is our NERC CIP patching schedule?",
    "original_response": "The response text...",
    "correction_text": "The correct answer is...",
    "namespace": "compliance"
  }'
```

---

## Document Ingestion

### POST /api/ingest/file
Upload a document to the knowledge base.

```bash
curl -u 'user:pass' \
  -X POST https://app.inflexis.ai/api/ingest/file \
  -F 'file=@document.pdf' \
  -F 'namespace=compliance' \
  -F 'owner=bryan'
```

Response:
```json
{
  "status": "success",
  "filename": "document.pdf",
  "file_hash": "abc123def456",
  "chunks_created": 47,
  "storage_mode": "azure+keyword",
  "compliance_frameworks": ["NERC CIP", "IEC 62443"],
  "adr_id": "0001-INGEST"
}
```

Supported file types: PDF, DOCX, PPTX, XLSX, TXT, MD, RTF, HTML, XML, CSV, TSV, JSON, MP3, MP4, WAV, FLAC, WEBM, YouTube URLs, web page URLs, Pages, Numbers, Keynote.

### GET /api/ingest/stats
Returns knowledge base statistics.

```bash
curl -u 'user:pass' https://app.inflexis.ai/api/ingest/stats
```

Response:
```json
{
  "storage_mode": "azure+keyword",
  "tiers": {
    "keyword": { "total": 168778, "namespaces": 12 },
    "semantic": { "available": false },
    "azure": { "total": 168778, "index": "mao-knowledge" }
  }
}
```

### GET /api/ingest/adr/{file_hash}
Retrieve the 5-stage ADR pipeline log for a specific ingested file.

```bash
curl -u 'user:pass' https://app.inflexis.ai/api/ingest/adr/abc123def456
```

---

## Admin Endpoints

Requires `admin` or `founder` role.

### GET /api/admin/adrs/pending
List all pending ADR change requests awaiting approval.

### POST /api/admin/adrs/{adr_id}/approve
Approve a pending change request.

### POST /api/admin/adrs/{adr_id}/deny
Deny a pending change request with reason.

### GET /api/admin/adrs/history
Retrieve the full ADR change control history.

### POST /api/admin/archive
Archive a namespace or document set.

### POST /api/admin/rollback/{archive_id}
Restore a previous state from archive.

---

## Meetings

### POST /api/meetings/ingest
Upload a meeting recording or transcript for analysis.

```bash
curl -u 'user:pass' \
  -X POST https://app.inflexis.ai/api/meetings/ingest \
  -F 'file=@meeting-recording.mp4' \
  -F 'title=FRENOS Discovery Call' \
  -F 'namespace=meetings'
```

Returns: meeting summary, SPICED scorecard, action items, and ADR record.

### GET /api/meetings/{meeting_id}/summary
Retrieve analysis results for a previously ingested meeting.

---

## MCP Server (Claude Desktop)

The MAO platform includes an MCP (Model Context Protocol) server at `mcp_server.py`. When connected to Claude Desktop, it exposes the following tools:

| Tool | Description |
|---|---|
| `search_knowledge_base` | Semantic + keyword search across all accessible namespaces |
| `get_kb_stats` | Current knowledge base statistics |
| `get_pending_adrs` | Pending change control requests (admin only) |
| `get_adr_history` | Change control audit trail |
| `get_usage_summary` | Current API usage and budget status |
| `get_meeting_summary` | Retrieve analysis for a specific meeting |
| `get_my_action_items` | Outstanding action items across all meetings |
| `get_deal_context` | All meetings, decisions, and SPICED history for a company |

---

## Error Responses

| Code | Meaning |
|---|---|
| `400` | Bad request — missing or invalid parameters |
| `401` | Authentication required |
| `403 UNREGISTERED` | Email registration required before using chat |
| `403 PENDING` | Registration pending admin approval |
| `403` | Namespace access denied for your role |
| `500` | Internal server error — check App Insights for trace |

---

*For integration examples, see the [mao-examples](https://github.com/inflexis-ai/mao-examples) repository.*
