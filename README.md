# Nyria MCP server

Send trade alerts from Claude, Cursor or any MCP client to Nyria Paper or your broker. Alert to order, sub-second.

Endpoint: `https://api.nyria.io/mcp` (Streamable HTTP, hosted by Nyria)

## Auth

Each strategy has its own token. Send it as a header:

```
Authorization: Bearer <token>
```

Open the strategy on Nyria, go to the Integration tab and select Show URL. The token is the last part of the MCP server URL. Rotate it there any time.

## Tools

- `nyria_send_alert` sends an alert to the bots on the strategy.
- `nyria_preview_alert` shows how Nyria reads an alert. It places nothing.
- `nyria_strategy_info` shows the strategy's name, status and active bots.
- `nyria_list_bots` lists your bots with broker, paper or live, and status.
- `nyria_recent_orders` lists recent orders on the strategy.

A live bot turns a sent alert into a real order. Start with a paper bot.

## Connect

Claude Code:

```
claude mcp add --transport http nyria https://api.nyria.io/mcp --header "Authorization: Bearer YOUR_TOKEN"
```

Cursor, in `~/.cursor/mcp.json` or `.cursor/mcp.json`. Set `NYRIA_MCP_TOKEN` in the environment Cursor starts from:

```json
{
  "mcpServers": {
    "nyria": {
      "url": "https://api.nyria.io/mcp",
      "headers": {
        "Authorization": "Bearer ${env:NYRIA_MCP_TOKEN}"
      }
    }
  }
}
```

## Links

- Docs: https://docs.nyria.io/alerts/mcp
- MCP page: https://nyria.io/signals/mcp
- Nyria: https://nyria.io

Paper is free. Live trading from $15 a month.
