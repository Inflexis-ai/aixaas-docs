# Changelog — AIXaaS™ Platform Documentation

All notable documentation updates are listed here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Planned
- GraphRAG / Neo4j architecture guide
- Azure Service Bus integration guide
- n8n self-hosted workflow automation guide
- Vapi voice AI integration guide
- PostgreSQL tenant registry guide

---

## [1.1.0] — 2026-03-18

### Added
- `docs/integrations/mcp-server.md` — Full Claude Desktop MCP integration guide (8 tools, setup, usage examples, security notes)
- `docs/integrations/llm-providers.md` — Complete 6-tier model routing reference with cost tables and data residency policy
- `CONTRIBUTING.md` — Documentation contribution guidelines and writing standards
- `CHANGELOG.md` — This file
- Issue templates: bug report and documentation request
- PR template with checklist

### Improved
- `README.md` — Added shields.io badges, ecosystem reference table with live star counts, expandable architecture sections, quickstart code example
- `docs/integrations/azure.md` — Added full environment variables reference table, CLI command examples

---

## [1.0.0] — 2026-03-18

### Added (Initial Release)
- `README.md` — Platform overview with architecture diagram, feature tables, LLM tier reference
- `docs/platform-overview.md` — Design philosophy, 10-layer architecture, ingest pipeline, ADR system, meeting intelligence
- `docs/architecture/overview.md` — System topology, 4-tier storage architecture, security model, authentication
- `docs/integrations/azure.md` — All Azure service integrations with resource references and CLI commands
- `docs/integrations/api-reference.md` — Full API reference: health, chat, ingest, admin, meetings, MCP tools
- `docs/guides/deployment.md` — CI/CD pipeline, Azure Files volume setup, custom domains, health verification, rollback

---

## Platform Release History (for reference)

| Platform Version | Date | Key Features |
|---|---|---|
| Phase 6B | 2026-03-10 | ADR review UI, sector governance, custom domain launch |
| Phase 6A | 2026-03-03 | 8-role RBAC, ADR change control, semantic cache, MCP server |
| Phase 5 | 2026-02-28 | Universal file ingestion (20+ types), 3-tier storage |
| Phase 4 | 2026-02-14 | Azure AI Search, Blob backup, Key Vault, App Insights |
| Phase 3 | 2026-02-01 | PDF/DOCX/audio parsing, Pinecone cloud vector |
| Phase 2 | 2026-01-15 | RAG pipeline, compliance engine (23 frameworks) |
| Phase 1 | 2026-01-01 | Power BI integration, FastAPI foundation |
