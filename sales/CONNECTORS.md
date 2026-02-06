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

Dataverse is tenant-specific, so there's no shared HTTP URL to put in `.mcp.json`. You install Microsoft's official local MCP proxy once and add it to your Claude Desktop config.

> **macOS / Linux:** The official proxy currently crashes on non-Windows systems due to an [unresolved MSAL Keychain bug](https://github.com/microsoft/Dataverse-MCP/issues/11) (open since July 2025). The steps below work on **Windows only** until Microsoft ships a fix.

**1. Install .NET SDK 8+** (skip if `dotnet --version` already works) — download from [dotnet.microsoft.com](https://dotnet.microsoft.com/download).

**2. Install the Dataverse MCP proxy:**

```bash
dotnet tool install --global Microsoft.PowerPlatform.Dataverse.MCP
```

**3. Enable in your Dataverse environment** — in [Power Platform admin center](https://admin.powerplatform.microsoft.com), go to your environment → Settings → Features → enable *Allow MCP clients to interact with Dataverse MCP server*.

**4. Get the connection URL:**

1. Open [Power Automate](https://make.powerautomate.com) → **Connections** (left sidebar).
2. Click **+ New connection** → search "Microsoft Dataverse" → create it.
3. Once created, click the connection to open its details page.
4. Look at the **browser address bar** — it will look like:
   `https://make.powerautomate.com/environments/<env-id>/connections/shared_commondataserviceforapps/<connection-name>/details`
5. Re-format it into the URL the tool expects by moving the parts into query parameters:
   `https://make.powerautomate.com/environments/<env-id>/connections?apiName=shared_commondataserviceforapps&connectionName=<connection-name>`

> **Example:** if your browser shows
> `.../environments/Default-abc123/connections/shared_commondataserviceforapps/shared-commondataser-xyz789/details`
> your connection URL is
> `https://make.powerautomate.com/environments/Default-abc123/connections?apiName=shared_commondataserviceforapps&connectionName=shared-commondataser-xyz789`

**5. Add to Claude Desktop config** (`%APPDATA%\Claude\claude_desktop_config.json` on Windows):

```json
{
  "mcpServers": {
    "dataverse": {
      "command": "Microsoft.PowerPlatform.Dataverse.MCP",
      "args": [
        "--ConnectionUrl", "<your connection URL from step 4>",
        "--MCPServerName", "DataverseMCPServer",
        "--TenantId", "<your tenant ID>",
        "--BackendProtocol", "HTTP"
      ]
    }
  }
}
```

Get your **Tenant ID** from [Power Apps](https://make.powerapps.com) → gear icon (top-right) → **Session details** → Tenant ID.

This gives you CRM data tools — `read_query`, `create_record`, `update_record`, `list_tables`, `search`, etc. — so the same `~~CRM` workflows (pipeline, accounts, contacts) work with Dynamics 365 just like HubSpot or Close.

Full guide: [Connect Dataverse MCP with non-Microsoft clients](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-other-clients) | [Dataverse MCP Labs (GitHub)](https://github.com/microsoft/Dataverse-MCP)
