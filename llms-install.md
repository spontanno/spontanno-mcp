# Installing the СПОНТАННО MCP server

This is a hosted remote MCP server. There is nothing to clone, build or run locally, and no API key is needed.

- Name: `spontanno`
- URL: `https://spontanno.space/mcp`
- Transport: Streamable HTTP
- Authentication: none

## Cline

Add this entry to `cline_mcp_settings.json` (keep any existing servers):

```json
{
  "mcpServers": {
    "spontanno": {
      "type": "streamableHttp",
      "url": "https://spontanno.space/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Or use the **Remote Servers** tab: name `spontanno`, URL `https://spontanno.space/mcp`, transport **Streamable HTTP**, then **Add Server**.

## Check that it works

The server exposes five tools: `search_events`, `spontanno_random_event`, `get_event`, `list_cities`, `create_shortlist`. Try: "What's on in Moscow this weekend?" — the answer should list events with links to spontanno.space.

## Notes

- Do not send GET requests to `/mcp`: the server is stateless and answers GET and DELETE with 405 by design.
- Event data is in Russian; dates and times are local to each city.
- If a tool answers that the service is busy, retry in a minute.

<!-- Формат записи Cline для удалённого сервера — https://docs.cline.bot/mcp/connecting-to-a-remote-server (проверено 02.10.2026); файл нужен для подачи в https://github.com/cline/mcp-marketplace. -->
