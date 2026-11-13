# Nyria MCP server

[![M8ven Score](https://m8ven.ai/badge/mcp/nyria-llc-nyria-mcp-1a2wsm?v=d7074a42ff68a3aa9217c2e653a58164)](https://m8ven.ai/mcp/nyria-llc-nyria-mcp-1a2wsm?s=readme)

Nyria's MCP server lets Claude, ChatGPT, Cursor, Claude Code or any other MCP client send trade alerts to a Nyria strategy. A bot on that strategy places each alert as an order at your broker, such as Charles Schwab, Alpaca, tastytrade, Tradier, Coinbase or Kraken. A bot can also fill it on Nyria Paper, the free simulator. Alert to order, sub-second.

Endpoint: `https://api.nyria.io/mcp` (Streamable HTTP, hosted by Nyria, nothing to install)

## Questions

### How do I connect Claude or ChatGPT to my broker?

Connect the broker under Integrations in Nyria: Charles Schwab, tastytrade and Tradier through their own sign-in, Hyperliquid with an API wallet you approve, the other live brokers with an API key. Create a strategy with AI Agent (MCP) as the alert source and copy its MCP server URL from the Integration tab. Add that URL as a custom connector in Claude, or as a custom MCP server in ChatGPT ([step by step](https://nyria.io/guides/mcp-server-for-trading)). OpenAI says custom MCP connectors are available on its Business, Enterprise and Edu plans.

### Can an AI trading agent place real orders?

Yes, when the strategy has a live bot. The agent sends the alert, and the live bot places a real order at its broker, sized by the bot's own Budget per entry. A paper bot only simulates.

### Is there an MCP server for stock trading?

Yes. Nyria runs one at `https://api.nyria.io/mcp`, listed in the official MCP registry as `io.github.Nyria-LLC/nyria`, with one token per strategy. See the [Nyria MCP page](https://nyria.io/signals/mcp).

### Can I let an AI agent trade my Alpaca or Schwab account on paper first?

Yes. Deploy a bot on Nyria Paper, which is free and needs no broker, and let the agent preview and send alerts there. Alpaca also connects with a paper API key, and Tradier with a sandbox token. An agent can send a wrong or duplicate alert, so read the paper log before you deploy a bot on a live account. Nyria can't connect Charles Schwab's paperMoney.

## Quickstart

1. Create a free Nyria account, a strategy with AI Agent (MCP) as the alert source, and a bot on Nyria Paper.
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

Clients that only take a URL can use `https://api.nyria.io/mcp/YOUR_TOKEN` instead.

The token works like a password and only reaches that one strategy. Rotate it on the Integration tab any time, which cuts off the old one at once. Keep it in an environment variable, out of chats and commits.

The tool list reads without a token. Calling a tool needs one.

## Tools

| Tool | What it does |
|---|---|
| `nyria_preview_alert` | Shows how Nyria reads an alert. Places no order. |
| `nyria_send_alert` | Sends an alert to every bot on the strategy. A live bot places a real order. Returns an `order_id` for each order leg. |
| `nyria_order_status` | Looks up one order by `order_id`: each bot's status, quantity, filled quantity, fill price, broker and account type. |
| `nyria_positions` | Open positions on each broker account with a bot on the strategy: account (last 4 digits), instrument, side, filled quantity and average price. Covers positions Nyria opened. |
| `nyria_recent_orders` | Recent orders on the strategy: `order_id`, instrument, action, status, quantity, price and timestamp. |
| `nyria_list_bots` | Bots on the strategy: broker, live or paper, status and position size. |
| `nyria_strategy_info` | The strategy's name, status, equity type and active bot count. |
| `nyria_describe_states` | Plain meaning of each status value Nyria returns for bots, strategies, broker connections, orders and plans. |

`nyria_send_alert` is the only tool that places orders. The other seven are read-only.

A live bot turns a sent alert into a real order. Accounts that copy the strategy receive sent alerts too. Start with a paper bot, and preview before you send.

## Alerts

An alert needs a ticker and an action. Plain text works, and so does JSON:

```json
{ "ticker": "SPY", "action": "OPEN", "direction": "LONG" }
```

The bot sets the order size. Nyria places market and limit orders, and exits are alerts of their own. Field reference: [webhook payload reference](https://docs.nyria.io/webhook-reference).

## Brokers

A bot places orders for stocks, options and crypto at the broker it runs on.

- Live: Binance, Coinbase, OKX, MEXC, Charles Schwab, Alpaca, Kraken, Hyperliquid, Kraken Futures, tastytrade, Tradier, My Funded Perps
- Beta: Webull, Robinhood (unofficial), Robinhood Crypto, E*TRADE, Gemini, Binance US, Coinbase US
- Nyria Paper: the free simulator, with no broker needed

## Webhook and REST API

Code can send alerts to a strategy without an MCP client. Every strategy has its own token.

- Webhook: `POST https://trades.nyria.io/webhooks/YOUR_STRATEGY_TOKEN`. See the [webhook payload reference](https://docs.nyria.io/webhook-reference).
- REST API: base URL `https://api.nyria.io/v1`, with the header `Authorization: Bearer YOUR_STRATEGY_TOKEN`. Routes: `POST /v1/alerts`, `GET /v1/positions`, `GET /v1/orders` and `GET /v1/orders/{order_id}`. See the [REST API docs](https://docs.nyria.io/rest-api).
- OpenAPI description: [openapi.yaml](openapi.yaml)

## Building a bot with an AI assistant

[Nyria-LLC/ai-trading-bot-starter](https://github.com/Nyria-LLC/ai-trading-bot-starter) has `CLAUDE.md`, `AGENTS.md`, Cursor rules and prompts for writing a Pine Script v6 strategy and testing it on paper through this server. [Nyria-LLC/trading-alert-examples](https://github.com/Nyria-LLC/trading-alert-examples) has alert messages checked against Nyria's alert tester.

## Links

- [Connect an AI agent to Nyria (MCP) in the docs](https://docs.nyria.io/alerts/mcp)
- [Nyria MCP page](https://nyria.io/signals/mcp)
- [Paper trading on Nyria](https://docs.nyria.io/paper-trading)
- [Nyria](https://nyria.io)

Paper is free. Live trading from $15 a month. Nyria is software, not a broker-dealer or an adviser, and trading involves risk of loss. See the [risk disclosure](https://docs.nyria.io/risk-disclosure).
