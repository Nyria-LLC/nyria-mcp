# Nyria MCP server

Nyria's MCP server lets Claude, ChatGPT, Cursor, Claude Code or any other MCP client send trade alerts to a Nyria strategy. A bot on that strategy places each alert as an order at Charles Schwab, Tradier, tastytrade or Alpaca, or fills it on Nyria Paper, the free simulator. Alert to order, sub-second.

Endpoint: `https://api.nyria.io/mcp` (Streamable HTTP, hosted by Nyria, nothing to install)

## Questions

### How do I connect Claude or ChatGPT to my broker?

Connect the broker under Integrations in Nyria: Schwab, Tradier and tastytrade through their own sign-in, Alpaca with an API key. Create a strategy with AI Agent (MCP) as the alert source and copy its MCP server URL from the Integration tab. Add that URL as a custom connector in Claude, or as a custom MCP server in ChatGPT ([step by step](https://nyria.io/guides/mcp-server-for-trading)).

### Can an AI trading agent place real orders?

Yes, when the strategy has a live bot. The agent sends the alert, and the live bot places a real order at its broker, sized by the bot's own Budget per entry. A paper bot only simulates.

### Is there an MCP server for stock trading?

Yes. Nyria runs one at `https://api.nyria.io/mcp`, listed in the official MCP registry as `io.github.Nyria-LLC/nyria`, with one token per strategy. See the [Nyria MCP page](https://nyria.io/signals/mcp).

### Can I let an AI agent trade my Alpaca or Schwab account on paper first?

Yes. Deploy a bot on Nyria Paper, which is free and needs no broker, and let the agent preview and send alerts there. Alpaca also connects with a paper API key. An agent can send a wrong or duplicate alert, so read the paper log before you deploy a second bot on the Alpaca or Schwab account. Nyria can't connect Schwab's paperMoney.

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
| `nyria_describe_states` | Plain meaning of each status value Nyria returns for bots, strategies, broker connections, orders and plans. |

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
