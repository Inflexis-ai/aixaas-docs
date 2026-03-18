# MCP Server — Claude Desktop Integration

The MAO Platform ships a **Model Context Protocol (MCP) server** that exposes platform capabilities directly to Claude Desktop. Once connected, you can query your knowledge base, review pending change requests, check action items, and get meeting context — all from within a Claude conversation.

---

## What Is MCP?

The [Model Context Protocol](https://github.com/modelcontextprotocol/servers) (MCP) is an open standard by Anthropic for connecting Claude to external data sources and tools. The MAO MCP server implements this protocol, making MAO's 168K+ chunk knowledge base and platform management tools directly accessible in Claude Desktop without opening a browser.

---

## Available Tools

| Tool | Description | Access |
|---|---|---|
| `search_knowledge_base` | Semantic + keyword search across all your accessible namespaces | All roles |
| `get_kb_stats` | Current chunk counts, storage mode, tier availability | All roles |
| `get_pending_adrs` | List change requests awaiting admin approval | admin, founder |
| `get_adr_detail` | Full diff for a specific change request | admin, founder |
| `get_adr_history` | Complete change control audit trail | admin, founder |
| `get_usage_summary` | Current API budget usage across roles | admin, founder |
| `get_meeting_summary` | Analysis, SPICED scorecard, and action items for a meeting | cro, admin, founder |
| `get_my_action_items` | All outstanding action items from recent meetings | All roles |

---

## Setup

### Prerequisites

- Claude Desktop installed (macOS or Windows)
- MAO Platform access (credentials from admin)
- Python 3.12+ on the machine running Claude Desktop

### 1. Locate the MCP server file

The MCP server is included in the MAO Platform codebase at `mcp_server.py`. For external users, request the server file from your Inflexis account team.

### 2. Configure Claude Desktop

Edit your Claude Desktop config file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "mao-platform": {
      "command": "python",
      "args": ["C:/path/to/mao-platform/mcp_server.py"],
      "env": {
        "MAO_BASE_URL": "https://app.inflexis.ai",
        "MAO_USERNAME": "your-username",
        "MAO_PASSWORD": "your-password"
      }
    }
  }
}
```

### 3. Restart Claude Desktop

After saving the config, fully quit and relaunch Claude Desktop. The MAO tools will appear in the tool selector.

---

## Usage Examples

### Search your knowledge base

```
Tell me what our knowledge base says about NERC CIP patching obligations.
```
→ Claude invokes `search_knowledge_base` → returns grounded answer with sources from your actual documents.

### Review pending change requests

```
Are there any pending agent change requests waiting for my approval?
```
→ Claude invokes `get_pending_adrs` → lists all submissions with proposer, target, and summary.

### Check your action items

```
What action items do I have outstanding from this week's meetings?
```
→ Claude invokes `get_my_action_items` → returns items attributed to you across all recent meetings.

### Pre-meeting brief

```
Give me context on the TACenergy deal before my 2pm call.
```
→ Claude invokes `search_knowledge_base` + `get_meeting_summary` → synthesizes past meetings, SPICED scores, and open questions.

---

## Security

The MCP server authenticates with the MAO Platform using the same HTTP Basic Auth credentials as the web dashboard. Credentials are stored only in the Claude Desktop config file (local to your machine). They are never transmitted to Anthropic or any third party — all requests go directly from your machine to `app.inflexis.ai`.

The tools respect your role's access scope: a `member` querying via MCP will see only the namespaces they're permitted to access, identical to the web dashboard behavior.

---

## Troubleshooting

**Tool doesn't appear in Claude Desktop**
- Verify the config file path and JSON syntax
- Confirm Python 3.12+ is on your system PATH
- Check that all MCP server dependencies are installed: `pip install -r requirements-prod.txt`
- Fully quit Claude Desktop (not just close the window) and relaunch

**Authentication errors**
- Verify `MAO_USERNAME` and `MAO_PASSWORD` in your Claude Desktop config
- Test credentials: `curl -u 'username:password' https://app.inflexis.ai/health`

**Search returns no results**
- Check that the knowledge base has documents ingested: `get_kb_stats`
- Verify your role has access to the namespace you're querying

---

*For setup assistance, contact [bryan.shaw@inflexis.ai](mailto:bryan.shaw@inflexis.ai)*

*Related: [MCP Protocol Specification](https://github.com/modelcontextprotocol/servers) · [Anthropic MCP Docs](https://www.anthropic.com/news/model-context-protocol)*
