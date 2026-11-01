# Nyria MCP server

Send trade alerts from Claude, Cursor or any MCP client to Nyria Paper or your broker. Alert to order, sub-second.

Endpoint: `https://api.nyria.io/mcp` (Streamable HTTP, hosted by Nyria, nothing to install)

## Quickstart

1. Create a free Nyria account, a strategy and a bot on Nyria Paper.
2. Open the strategy, go to the Integration tab and select Show URL. The token is the last part of the MCP server URL.
3. Add the server to your client.

Claude Code:

```sh
export NYRIA_MCP_TOKEN="paste-your-token-here"
claude mcp add --transport http nyria https://api.nyria.io/mcp --header "Authorization: Bearer $NYRIA_MCP_TOKEN"
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

4. Ask your agent: "Run nyria_list_bots, then preview an alert to buy SPY. Send nothing."

Claude.ai and Claude Desktop take a custom connector with the URL `https://api.nyria.io/mcp/YOUR_TOKEN`. Step-by-step guides: [Claude](https://nyria.io/guides/connect-claude-to-nyria-mcp), [Cursor](https://nyria.io/guides/connect-cursor-to-nyria-mcp) and [ChatGPT](https://nyria.io/guides/connect-chatgpt-to-nyria-mcp).

## Auth

Each strategy has its own token. Send it as a header:

```
Authorization: Bearer <token>
```

The token works like a password and only reaches that one strategy. Rotate it on the Integration tab any time, which stops the old one at once. Keep it in an environment variable, out of chats and commits.

## Tools

| Tool | What it does |
|---|---|
| `nyria_preview_alert` | Shows how Nyria reads an alert. Places nothing. |
| `nyria_send_alert` | Sends an alert to the bots on the strategy. Returns an `order_id` for each order. |
| `nyria_order_status` | Looks up one order by `order_id`, with status, filled quantity and fill price. |
| `nyria_positions` | Open positions on each account with a bot on the strategy. |
| `nyria_recent_orders` | Recent orders on the strategy. |
| `nyria_list_bots` | Your bots with broker, paper or live, and status. |
| `nyria_strategy_info` | The strategy's name, status, equity type and active bot count. |

A live bot turns a sent alert into a real order. Start with a paper bot, and preview before you send.

## Alerts

An alert needs a ticker and an action. Plain text works, and so does JSON:

```json
{ "ticker": "SPY", "action": "OPEN", "direction": "LONG" }
```

The bot sets the order size. Nyria places market and limit orders, and exits are alerts of their own. Field reference: [webhook payload reference](https://docs.nyria.io/webhook-reference).

## Building a bot with an AI assistant

[Nyria-LLC/ai-trading-bot-starter](https://github.com/Nyria-LLC/ai-trading-bot-starter) has `CLAUDE.md`, `AGENTS.md`, Cursor rules and prompts for writing a Pine Script v6 strategy and testing it on paper through this server. [Nyria-LLC/trading-alert-examples](https://github.com/Nyria-LLC/trading-alert-examples) has alert messages checked against Nyria's alert tester.

## Links

- [Connect an AI agent to Nyria (MCP) in the docs](https://docs.nyria.io/alerts/mcp)
- [Nyria MCP page](https://nyria.io/signals/mcp)
- [Paper trading on Nyria](https://docs.nyria.io/paper-trading)
- [Nyria](https://nyria.io)

Paper is free. Live trading from $15 a month. Nyria is software, not a broker-dealer or an adviser, and trading involves risk of loss. See the [risk disclosure](https://docs.nyria.io/risk-disclosure).
