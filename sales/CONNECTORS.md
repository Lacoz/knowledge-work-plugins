# Connectors

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that category. For example, `~~CRM` might mean Salesforce, HubSpot, or any other CRM with an MCP server.

Plugins are **tool-agnostic** — they describe workflows in terms of categories (CRM, chat, email, etc.) rather than specific products. The `.mcp.json` pre-configures specific MCP servers, but any MCP server in that category works.

## Connectors for this plugin

| Category | Placeholder | Included servers | Other options |
|----------|-------------|-----------------|---------------|
| Calendar | `~~calendar` | Microsoft 365 | Google Calendar |
| Chat | `~~chat` | Slack | Microsoft Teams |
| CRM | `~~CRM` | HubSpot, Close | Salesforce, Pipedrive, Copper, Dynamics 365 (Dataverse) |
| Data enrichment | `~~data enrichment` | Clay, ZoomInfo | Apollo, Clearbit, Lusha |
| Email | `~~email` | Microsoft 365 | Gmail |
| Knowledge base | `~~knowledge base` | Notion | Confluence, Guru |
| Meeting transcription | `~~conversation intelligence` | Fireflies | Gong, Chorus, Otter.ai |
| Project tracker | `~~project tracker` | Atlassian (Jira/Confluence) | Linear, Asana |

## Dynamics 365 / Dataverse setup

Dataverse is tenant-specific, so there's no shared HTTP URL to put in `.mcp.json`. Instead you install a small local MCP server once. It works on **Windows, macOS, and Linux** (requires [.NET 8+](https://dotnet.microsoft.com/download)).

**Install:**

```bash
dotnet tool install --global Microsoft.PowerPlatform.Dataverse.MCP
```

**Enable in your Dataverse environment** — in [Power Platform admin center](https://admin.powerplatform.microsoft.com), go to your environment → Settings → Features → enable *Allow MCP clients to interact with Dataverse MCP server*. Then create a Dataverse connection in [Power Automate](https://make.powerautomate.com) → Connections → + New → "Microsoft Dataverse" and note the connection URL.

**Add to your MCP config** (`~/.cursor/mcp.json`, Claude Desktop config, etc.):

```json
"dataverse": {
  "command": "Microsoft.PowerPlatform.Dataverse.MCP",
  "args": [
    "--ConnectionUrl", "<your connection URL>",
    "--MCPServerName", "DataverseMCPServer",
    "--TenantId", "<your tenant ID>",
    "--BackendProtocol", "HTTP"
  ]
}
```

Get your Tenant ID from Power Apps → Settings (gear) → Session details.

This gives you CRM data tools — `read_query`, `create_record`, `update_record`, `list_tables`, `search`, etc. — so the same `~~CRM` workflows (pipeline, accounts, contacts) work with Dynamics 365 just like HubSpot or Close.

Full guide: [Connect Dataverse MCP with non-Microsoft clients](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-other-clients) | [Dataverse MCP Labs (GitHub)](https://github.com/microsoft/Dataverse-MCP)
