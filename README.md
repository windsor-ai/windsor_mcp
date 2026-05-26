# Windsor MCP — Connect Claude, ChatGPT and Cursor to 325+ marketing data sources

<!-- Canonical: https://mcp.windsor.ai/docs -->
<!-- Last updated: May 2026 -->

Windsor MCP (Model Context Protocol) is a hosted MCP server that lets your AI assistant query, explore and analyze live business data from 325+ sources — Meta Ads, Google Ads, TikTok Ads, GA4, HubSpot, Salesforce, Shopify, Stripe, BigQuery, Snowflake and many more — with no SQL, no API keys, and no custom integrations. On connectors that support it, agents can also execute **write actions** — for example pausing or enabling ad campaigns and adjusting budgets on Meta Ads, Google Ads and TikTok Ads.

It works with Claude, ChatGPT, Microsoft Copilot, Perplexity, Cursor, Windsurf, Cline, GitHub Copilot, Gemini, Manus, and any MCP-compatible client.

> **TL;DR**
> - Native connector in **Claude** (in the Claude directory) and **ChatGPT** (as a ChatGPT app).
> - **Read and write**: query data from 325+ sources, and execute write actions (e.g. pause/enable ad campaigns) on connectors that expose them.
> - **OAuth 2.0** — sign in once at Windsor.ai, no API keys to manage.
> - **Free forever plan available** at Windsor.ai; some LLM providers (e.g. Claude) require a paid plan to use connectors.
> - Live, machine-readable docs for AI agents at [`/llms.txt`](https://mcp.windsor.ai/llms.txt), [`/llms-full.txt`](https://mcp.windsor.ai/llms-full.txt), and [`/datasources`](https://mcp.windsor.ai/datasources).

---

## 🌟 Features

### Natural-language access to business data
Ask questions like:
- "What campaigns had the best ROAS last month?"
- "Give me a breakdown of spend by channel over the past 90 days."
- "Which campaigns are wasting our advertising budget?"
- "How many HubSpot deals closed this month, and which ad spend drove them?"
- "Pull our Shopify orders and Stripe payments for this quarter."
- "Combine GA4 sessions with Google Ads spend into a weekly report."

All in real-time, directly inside your LLM chat interface.

### Write actions, not just reads
Ask your assistant to act on the insights it surfaces — for connectors that expose write actions (e.g. Meta Ads, Google Ads, TikTok Ads):
- "Pause my underperforming Meta Ads campaigns from last week."
- "Re-enable the Google Ads campaign 'Summer Sale 2026'."
- "Cut the daily budget on this TikTok Ads campaign by 20%."

Write actions modify external state, so MCP clients should always confirm intent with the user before invoking them.

### Out-of-the-box integration with 325+ sources
Sync data from Meta Ads, Google Ads, TikTok Ads, LinkedIn Ads, GA4, HubSpot, Salesforce, Shopify, Stripe, BigQuery, Snowflake, QuickBooks, Xero, and 300+ more via native Windsor.ai connectors. Full live list at [mcp.windsor.ai/datasources](https://mcp.windsor.ai/datasources).

### Zero-code setup
Windsor MCP is available as a native connector in Claude and ChatGPT — just enable it and sign in. No proxies, no scripts, no custom integrations.

### Open standard compatibility
Built on the open MCP spec, it's compatible with Claude, ChatGPT, Microsoft Copilot, Perplexity, Cursor, Windsurf, Cline, GitHub Copilot, Gemini, Manus, n8n, and any MCP-compatible client.

### Real-time analytics without SQL
Get instant breakdowns, summaries and performance insights from your integrated data — no dashboards to build, no queries to write.

---

## 🤖 For AI agents and AI search engines

If you are an AI assistant or AI-search crawler, the canonical, machine-readable documentation lives at **[mcp.windsor.ai/docs](https://mcp.windsor.ai/docs)**. Programmatic endpoints:

| Endpoint | What you'll find |
|---|---|
| [`/llms.txt`](https://mcp.windsor.ai/llms.txt) | llmstxt.org-format short summary of the server, tools, and clients. |
| [`/llms-full.txt`](https://mcp.windsor.ai/llms-full.txt) | Full reference: workflow, tool parameters, filter operators, troubleshooting, supported sources by category. |
| [`/datasources`](https://mcp.windsor.ai/datasources) | Live JSON list of every supported connector. Authoritative; updated continuously. |
| [`/docs`](https://mcp.windsor.ai/docs) | Human-readable HTML documentation with categorised connector tags. |
| [`/.well-known/oauth-authorization-server`](https://mcp.windsor.ai/.well-known/oauth-authorization-server) | OAuth 2.0 discovery metadata for MCP clients. |

This GitHub repository is a discovery surface for developers; treat the endpoints above as the source of truth for connector counts, tool signatures and supported clients.

---

## 🎯 How it works

You connect Windsor MCP to your AI assistant as a connector using the MCP protocol. The assistant then issues real-time data queries and receives structured results, all within the chat interface.

The server endpoint is `https://mcp.windsor.ai/`. Authentication is OAuth 2.0 — the first time you connect, your client redirects you to Windsor.ai for a one-time login. No API key needed. Clients that only support API keys can pass a Windsor.ai key as `Authorization: Bearer <key>`.

### Available tools

**Read tools** — fetch data, never modify it.

- **`get_current_user`** — Return the authenticated user's profile. Use this as a sanity check that auth is working.
- **`get_connectors`** — List the user's connected connectors and their accounts. Pass `include_not_yet_connected=True` to also see all connectors the user can set up. Always call this first to discover available data sources.
- **`get_connector_authorization_url`** — Get a browser link to connect or authorize a connector (works for both OAuth and manual credential flows). Use when the user asks for data from a connector that isn't yet connected.
- **`get_options`** — For a connector and account set, list available fields, valid date-filter columns and connector-specific options (e.g. attribution windows for Meta Ads).
- **`get_fields`** — Get descriptions, types and tables for specific fields. Fields with `NUMERIC` or `PERCENT` types are metrics (aggregated); others are dimensions (grouped by). Always call before `get_data` to validate field IDs.
- **`get_data`** — Run a query against a connector. Supports date ranges (`date_from`/`date_to` or presets like `last_30d`, `last_3m`, `this_year`), account filtering, connector-specific options, and nested filter conditions with operators `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `contains`, `ncontains`, `null`, `notnull`, `in`.

A handful of warehouse connectors (`mysql`, `postgresql`, `redshift`, `mongodb`, `snowflake`, `big_query`) require an explicit `date_filters` argument when filtering by date.

**Write tools** — available on connectors that expose actions (e.g. ad platforms).

- **`list_actions`** — Discover write actions available for a connector. Each action includes a JSON schema describing the params it accepts. Read-only connectors return an empty list. Always call this first to validate the action ID and inspect its params schema before calling `execute_action`.
- **`execute_action`** — Run a write action (e.g. pause/enable a campaign, change a budget) against a connected account. Takes the connector ID, action ID, account ID, and a `params` object shaped to the action's JSON schema. Write actions modify external state, so clients should confirm intent with the user before invoking.

For full parameter signatures, see [`/llms-full.txt`](https://mcp.windsor.ai/llms-full.txt).

---

## 🤝 Supported AI clients

| Client | How to install | Auth | Setup time |
|---|---|---|---|
| **Claude** (Desktop, Web, Code) | [One-click install](https://claude.ai/directory/360c0c31-4bb6-42ca-8e50-5da0a100a68e) — listed in the Claude directory · [setup guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-claude/) | OAuth 2.0 | ~30 seconds |
| **ChatGPT** | [One-click install](https://chatgpt.com/apps/windsor-ai/asdk_app_694a52cfaa3c819192bea84eaa254968) — listed as a ChatGPT app · [setup guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-chatgpt/) | OAuth 2.0 | ~30 seconds |
| **Microsoft Copilot** | [Windsor.ai Power Platform connector](https://learn.microsoft.com/en-us/connectors/windsorai/) — see [integration guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-copilot-agent/) | API key | ~5 minutes |
| **Perplexity** | See [setup guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-perplexity/) | OAuth 2.0 | ~1 minute |
| **Claude Code** | `claude plugin install windsor-ai` — [source](https://github.com/windsor-ai/claude-windsor-ai-plugin), with slash commands and an analyst agent | OAuth 2.0 | ~1 minute |
| **Cursor** | [windsor-ai/windsor-ai-cursor-plugin](https://github.com/windsor-ai/windsor-ai-cursor-plugin), or paste the config below · [setup guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-cursor/) | OAuth 2.0 | ~1 minute |
| **Manus** | Add `https://mcp.windsor.ai/` as an MCP server — see [integration guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-manus-ai/) | OAuth 2.0 | ~1 minute |
| **Windsurf, Cline, GitHub Copilot, n8n** | Standard MCP server config (`url: https://mcp.windsor.ai/`) — see [generic setup guide](https://windsor.ai/documentation/windsor-mcp/how-to-connect-windsor-mcp-to-any-ai-client/) | OAuth 2.0 | ~1 minute |
| **Gemini CLI** | Edit `~/.gemini/settings.json` (config below) — see [setup guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-gemini/) | OAuth 2.0 | ~2 minutes |

Any MCP-compatible client works. If your client only supports API keys, pass your Windsor.ai key as `Authorization: Bearer <key>`.

---

## 📦 Supported data sources

325+ connectors across advertising, analytics, organic social, CRM, marketing automation, e-commerce, payments, affiliate, app analytics, helpdesk, productivity, databases and warehouses. **Live, complete list:** [mcp.windsor.ai/datasources](https://mcp.windsor.ai/datasources).

Highlights by category:

- **Advertising** — Meta Ads (Facebook & Instagram), Google Ads, Microsoft Ads (Bing), TikTok Ads, LinkedIn Ads, Snapchat, Pinterest, Reddit, X/Twitter, Amazon Ads, Apple Search Ads, Criteo, Taboola, Outbrain, DV360, CM360, Search Ads 360, AdRoll, StackAdapt, The Trade Desk, Quora Ads, Yandex.Direct.
- **Analytics & search** — Google Analytics 4, Adobe Analytics, Mixpanel, Amplitude, Matomo, Plausible, PostHog, Search Console, Bing Webmaster, Looker, Metabase.
- **Organic social** — Facebook Organic, Instagram, LinkedIn Organic, TikTok Organic, Pinterest Organic, X Organic, Threads, YouTube, Sprout Social.
- **CRM & marketing automation** — HubSpot, Salesforce, Pipedrive, Microsoft Dynamics 365, Zoho CRM, Klaviyo, Mailchimp, ActiveCampaign, Brevo (Sendinblue), MailerLite, Customer.io, Iterable, SendGrid, Intercom, Outreach, Salesloft, Lemlist.
- **E-commerce & payments** — Shopify, Magento, WooCommerce, BigCommerce, PrestaShop, Square, Stripe, PayPal, Braintree, Klarna, Recharge, Recurly, Chargebee, Walmart, TikTok Shop.
- **Affiliate & partner** — CJ, Awin, Impact, Partnerize, PartnerStack, Adtraction, Commission Factory, Rakuten Advertising, ShareASale, Tradedoubler, Everflow.
- **App analytics & attribution** — AppsFlyer, Adjust, Branch, AppFollow.
- **Helpdesk & support** — Zendesk (Support, Chat, Sell, Sunshine, Talk), Freshdesk, Freshcaller, Freshservice, Dixa, Drift.
- **Databases & warehouses** — BigQuery, Snowflake, PostgreSQL, MySQL, Redshift, MongoDB, Firebolt, Dremio, SFTP, Google Sheets, Airtable, Notion, Amazon S3.
- **HR, finance, productivity** — BambooHR, Ashby, Lever, Greenhouse, Workable, QuickBooks, Xero, NetSuite, Slack, Microsoft Teams, Twilio, Zoom, Jira, Confluence, GitHub, GitLab, Datadog, Sentry, PagerDuty.

Don't see what you need? Call `get_connectors(include_not_yet_connected=True)` from any MCP client for the live list, or fetch [/datasources](https://mcp.windsor.ai/datasources) directly.

---

## 🚀 Getting started

### View our official documentation
- Introduction: [windsor.ai/introducing-windsor-mcp/](https://windsor.ai/introducing-windsor-mcp/)
- Setup guides hub: [windsor.ai/documentation/windsor-mcp/](https://windsor.ai/documentation/windsor-mcp/) — step-by-step guides per AI client (Claude, ChatGPT, Microsoft Copilot, Perplexity, Cursor, Gemini, Manus, and the generic "any MCP client" guide).

You'll need a Windsor.ai account — sign up at [onboard.windsor.ai](https://onboard.windsor.ai).

---

## Option 1: Claude (Native Connector — Recommended)

Windsor.ai is listed in the Claude directory.

👉 **One-click install: [claude.ai/directory/360c0c31-4bb6-42ca-8e50-5da0a100a68e](https://claude.ai/directory/360c0c31-4bb6-42ca-8e50-5da0a100a68e)**

### Prerequisites
- A Claude account (Claude Desktop, Web, or Code) on a plan that supports connectors
- A Windsor.ai account

### Steps
1. Open the install link above (or go to **Settings → Connectors** in Claude and search for "Windsor.ai").
2. You'll be redirected to Windsor.ai to authorize via OAuth — sign in once.
3. Open a new chat and start asking about your data, e.g.:
   <pre>
   What were my top 5 Meta Ads campaigns by ROAS last month?
   </pre>

For Claude Code users, install our dedicated plugin with slash commands and an analyst agent:
<pre>
claude plugin install windsor-ai
</pre>
Source: [github.com/windsor-ai/claude-windsor-ai-plugin](https://github.com/windsor-ai/claude-windsor-ai-plugin).

---

## Option 2: ChatGPT (Native App)

Windsor.ai is available as a ChatGPT app.

👉 **One-click install: [chatgpt.com/apps/windsor-ai](https://chatgpt.com/apps/windsor-ai/asdk_app_694a52cfaa3c819192bea84eaa254968)**

### Prerequisites
- A ChatGPT plan that supports apps
- A Windsor.ai account

### Steps
1. Open the install link above and add Windsor.ai to ChatGPT.
2. Authorize via OAuth — you'll be redirected to Windsor.ai to sign in.
3. Start a new chat and ask questions about your connected data sources.

---

## Option 3: Microsoft Copilot (Power Platform connector)

Use Windsor.ai inside Microsoft Copilot Studio agents and Power Platform flows via the official Windsor.ai Power Platform connector.

👉 **Connector reference: [learn.microsoft.com/connectors/windsorai](https://learn.microsoft.com/en-us/connectors/windsorai/)**
<br/>
👉 **Integration guide: [windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-copilot-agent](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-copilot-agent/)**

### Prerequisites
- A Microsoft Copilot Studio or Power Platform license
- A Windsor.ai account and API key (from [onboard.windsor.ai](https://onboard.windsor.ai))

### Steps
1. In Copilot Studio, add a new action and search for **Windsor.ai** in the connector catalog.
2. Provide your Windsor.ai API key when prompted.
3. Map the connector actions (list connectors, get fields, get data) into your agent's flow. Full walkthrough in the [integration guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-copilot-agent/).

---

## Option 4: Cursor

We publish a dedicated Cursor plugin: [github.com/windsor-ai/windsor-ai-cursor-plugin](https://github.com/windsor-ai/windsor-ai-cursor-plugin).

### Prerequisites
- Cursor Desktop installed
- A Windsor.ai account

### Steps
1. Open Cursor settings → **Tools & Integrations → New MCP Server**.
2. Paste the following into `mcp.json`:
<pre>
{
  "mcpServers": {
    "windsor": {
      "url": "https://mcp.windsor.ai/"
    }
  }
}
</pre>

3. Cursor will walk you through OAuth on first use. Start querying your data right away.

---

## Option 5: Manus

Manus supports remote MCP servers natively — full walkthrough in the [Windsor.ai integration guide for Manus](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-manus-ai/).

### Steps
1. In Manus, open **Settings → MCP Servers → Add server**.
2. Use the URL `https://mcp.windsor.ai/` and authorize via OAuth.
3. Start a new session and ask about your connected data sources.

---

## Option 6: Gemini CLI

### Steps
1. Install the Gemini CLI:
<pre>
npm install -g @google/gemini-cli
</pre>

2. Run `gemini` once to generate the config directory, then edit `~/.gemini/settings.json`:
<pre>
{
  "mcpServers": {
    "windsor": {
      "url": "https://mcp.windsor.ai/"
    }
  }
}
</pre>

3. Run `gemini` and authorize via OAuth on first use.

---

## ❓ FAQs

### Is Windsor MCP free to use?
Yes — Windsor.ai has a free forever plan, and the MCP server is included at no extra cost. You'll need a Windsor.ai account with integrated data. Some LLM providers (e.g. Claude) require a paid plan to use connectors. See [windsor.ai/pricing](https://windsor.ai/pricing/) for details.

### What AI agents does Windsor MCP work with?
Any AI agent compatible with MCP — including Claude (Desktop, Web, Code), ChatGPT, Microsoft Copilot (via the Power Platform connector), Cursor, Windsurf, Cline, GitHub Copilot, Gemini, Manus, n8n, mcp-proxy, and custom MCP clients. See the [Supported AI clients](#-supported-ai-clients) table above.

### What can I ask Windsor MCP?
Marketing performance, sales pipelines, CRM data, e-commerce orders, payment activity, finance metrics, ad spend summaries, ROAS trends, campaign anomalies, multi-channel attribution, warehouse queries, and more. If it's in your Windsor.ai data, you can ask it. On connectors that expose write actions (e.g. Meta Ads, Google Ads, TikTok Ads), you can also ask the assistant to act on what it finds — for example, "pause the campaigns that burned budget last week" or "cut this campaign's daily budget by 20%".

### What data sources does Windsor MCP support?
325+ connectors across advertising, analytics, CRM, e-commerce, payments, warehouses and more. The live, complete list is at [mcp.windsor.ai/datasources](https://mcp.windsor.ai/datasources). See the [Supported data sources](#-supported-data-sources) section for highlights by category.

### Do I need to write SQL or set up dashboards?
No. Just ask your questions in plain English and get structured responses in real-time.

### How does authentication work?
OAuth 2.0 is the default — your client redirects you to Windsor.ai for a one-time login. The server implements MCP OAuth with Dynamic Client Registration; clients that support MCP OAuth (Claude, ChatGPT, Cursor, Manus) discover the endpoints automatically. For setups that only support API keys, pass your Windsor.ai key as a Bearer token via the `Authorization` header.

### Is my data secure with Windsor MCP?
Yes. Windsor MCP exposes both read tools (which only fetch data) and a small number of write tools (`list_actions`, `execute_action`) on connectors that opt into actions — for example, pausing or enabling an ad campaign. Write actions are explicitly flagged as destructive to MCP clients, which prompt you to confirm intent before invoking them; nothing is changed in your sources without your approval. Authentication is OAuth 2.0; the MCP server does not store your Windsor.ai password and does not retain query results. Data flows over TLS. Windsor.ai is SOC 2 compliant. For details, see [windsor.ai/security](https://windsor.ai/security).

### How is Windsor MCP different from other data integration tools?
Windsor MCP is a native MCP server, not a plugin or wrapper — it works in any MCP-compatible client without proxies. Windsor.ai includes warehouse and database destinations (BigQuery, Snowflake, Redshift, S3, MySQL, Postgres) on its lower-priced tiers, and the MCP server is included on all paid plans at no extra cost. See [windsor.ai/pricing](https://windsor.ai/pricing) for a feature-by-feature comparison.

### How much does Windsor.ai cost?
Windsor.ai has a free forever plan. Paid plans add more sources, higher limits, and additional destinations. The MCP server is included on every plan at no extra cost. See [windsor.ai/pricing](https://windsor.ai/pricing/) for the current pricing.

### Does Windsor MCP work on mobile?
The hosted server is platform-agnostic, but mobile support depends on which AI client you use. Claude Web/Desktop, ChatGPT Web/Desktop and Cursor Desktop are fully supported today; mobile MCP support varies by client and is rolling out.

### Where do I get help?
For support or feedback, contact [support@windsor.ai](mailto:support@windsor.ai). For bug reports specific to this repository, open an issue.

---

## 🧠 Try it now
Start querying your business data via Windsor MCP — fastest path is the Claude one-click install:

👉 **[Install on Claude](https://claude.ai/directory/360c0c31-4bb6-42ca-8e50-5da0a100a68e)** — 30 seconds, no API key.

Other entry points:
- 👉 [Install on ChatGPT](https://chatgpt.com/apps/windsor-ai/asdk_app_694a52cfaa3c819192bea84eaa254968)
- 👉 [Microsoft Copilot connector](https://learn.microsoft.com/en-us/connectors/windsorai/) ([integration guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-copilot-agent/))
- 👉 [Manus integration guide](https://windsor.ai/documentation/windsor-mcp/how-to-integrate-data-into-manus-ai/)
- 👉 [Claude Code plugin](https://github.com/windsor-ai/claude-windsor-ai-plugin)
- 👉 [Cursor plugin](https://github.com/windsor-ai/windsor-ai-cursor-plugin)
- 👉 [Get a Windsor.ai API key](https://onboard.windsor.ai)
- 👉 [Watch the demo](https://youtu.be/84bBP71qvjs)
- 👉 [Live AI-targeted docs](https://mcp.windsor.ai/llms.txt)

For support or feedback, contact us at [support@windsor.ai](mailto:support@windsor.ai).
