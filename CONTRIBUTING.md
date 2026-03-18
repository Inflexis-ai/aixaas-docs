# Contributing to AIXaaS™ Documentation

Thank you for helping improve the AIXaaS™ platform documentation. This guide covers what we accept, how to submit changes, and our review process.

---

## What We Accept

This is a **documentation repository** for the AIXaaS™ / MAO Platform. We welcome:

✅ **Bug fixes** — broken links, outdated CLI commands, incorrect API schemas
✅ **Clarity improvements** — rewriting confusing sections, adding missing context
✅ **New documentation** — covering platform capabilities that aren't documented yet
✅ **Example additions** — new curl examples, code snippets, usage patterns

❌ **We do not accept**:
- Platform source code (the MAO engine is proprietary)
- Documentation for unreleased features
- Third-party integrations not officially supported

---

## Before You Submit

1. **Check existing issues** — your topic may already be tracked at [Issues](https://github.com/inflexis-ai/aixaas-docs/issues)
2. **Test any code examples** against the live platform if possible
3. **Verify all links** are live and point to the correct content

---

## Submitting a PR

1. **Fork** this repository
2. **Create a branch**: `git checkout -b docs/fix-api-response-schema`
3. **Make your changes** — keep scope small; one topic per PR
4. **Test locally**: `npx serve .` or any static file server to preview Markdown
5. **Open a PR** using the [PR template](.github/pull_request_template.md)

### Branch naming convention

| Type | Pattern | Example |
|---|---|---|
| Bug fix | `fix/description` | `fix/broken-deployment-link` |
| New doc | `docs/description` | `docs/add-postgres-guide` |
| Update | `update/description` | `update/azure-cli-commands` |

---

## Documentation Standards

### File structure

- Place files in the appropriate subdirectory: `docs/architecture/`, `docs/integrations/`, `docs/guides/`
- Use lowercase hyphenated filenames: `my-new-guide.md`
- Update the parent `README.md` table of contents when adding a new file

### Writing style

- **Active voice**: "The platform detects PII" not "PII is detected by the platform"
- **Present tense**: "The router selects" not "The router will select"
- **Specific over vague**: Include actual resource names, CLI commands, and response examples
- **No marketing language**: This is technical documentation, not a landing page

### Code blocks

Always specify the language for syntax highlighting:

````markdown
```bash
az containerapp show --name mao-platform --resource-group inflexis-rg
```

```python
import requests
response = requests.post(...)
```

```json
{
  "status": "success",
  "chunks_created": 47
}
```
````

### Tables

Use consistent table formatting:
```markdown
| Column 1 | Column 2 | Column 3 |
|---|---|---|
| Value | Value | Value |
```

---

## Review Process

All PRs are reviewed by the Inflexis team. Typical response time: 1–3 business days.

For urgent issues (broken critical docs), email [bryan.shaw@inflexis.ai](mailto:bryan.shaw@inflexis.ai) directly.

---

## Questions?

📧 [bryan.shaw@inflexis.ai](mailto:bryan.shaw@inflexis.ai)
🌐 [inflexis.ai](https://inflexis.ai)
