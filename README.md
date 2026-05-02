# Windsor MCP — Connect Claude, ChatGPT and Cursor to 325+ marketing data sources

<!-- Canonical: https://mcp.windsor.ai/docs -->
<!-- Last updated: May 2026 -->

Windsor MCP (Model Context Protocol) is a hosted MCP server that lets your AI assistant query, explore and analyze live business data from 325+ sources — Meta Ads, Google Ads, TikTok Ads, GA4, HubSpot, Salesforce, Shopify, Stripe, BigQuery, Snowflake and many more — with no SQL, no API keys, and no custom integrations.

It works with Claude, ChatGPT, Cursor, Windsurf, Cline, GitHub Copilot, Gemini, Manus, and any MCP-compatible client.

> **TL;DR**
> - Native connector in **Claude** (in the Claude directory) and **ChatGPT** (as a ChatGPT app).
> - **OAuth 2.0** — sign in once at Windsor.ai, no API keys to manage.
> - **Free during beta**; some LLM providers (e.g. Claude) require a paid plan to use connectors.
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

### Out-of-the-box integration with 325+ sources
Sync data from Meta Ads, Google Ads, TikTok Ads, LinkedIn Ads, GA4, HubSpot, Salesforce, Shopify, Stripe, BigQuery, Snowflake, QuickBooks, Xero, and 300+ more via native Windsor.ai connectors. Full live list at [mcp.windsor.ai/datasources](https://mcp.windsor.ai/datasources).

### Zero-code setup
Windsor MCP is available as a native connector in Claude and ChatGPT — just enable it and sign in. No proxies, no scripts, no custom integrations.

### Open standard compatibility
Built on the open MCP spec, it's compatible with Claude, ChatGPT, Cursor, Windsurf, Cline, GitHub Copilot, Gemini, Manus, n8n, and any MCP-compatible client.

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

All tools are read-only — they fetch data, never modify it.

- **`get_current_user`** — Return the authenticated user's profile. Use this as a sanity check that auth is working.
- **`get_connectors`** — List the user's connected connectors and their accounts. Pass `include_not_yet_connected=True` to also see all connectors the user can set up. Always call this first to discover available data sources.
- **`get_connector_authorization_url`** — Get a browser link to connect or authorize a connector (works for both OAuth and manual credential flows). Use when the user asks for data from a connector that isn't yet connected.
- **`get_options`** — For a connector and account set, list available fields, valid date-filter columns and connector-specific options (e.g. attribution windows for Meta Ads).
- **`get_fields`** — Get descriptions, types and tables for specific fields. Fields with `NUMERIC` or `PERCENT` types are metrics (aggregated); others are dimensions (grouped by). Always call before `get_data` to validate field IDs.
- **`get_data`** — Run a query against a connector. Supports date ranges (`date_from`/`date_to` or presets like `last_30d`, `last_3m`, `this_year`), account filtering, connector-specific options, and nested filter conditions with operators `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `contains`, `ncontains`, `null`, `notnull`, `in`.

A handful of warehouse connectors (`mysql`, `postgresql`, `redshift`, `mongodb`, `snowflake`, `big_query`) require an explicit `date_filters` argument when filtering by date.

For full parameter signatures, see [`/llms-full.txt`](https://mcp.windsor.ai/llms-full.txt).

---

## 🤝 Supported AI clients

| Client | How to install | Auth | Setup time |
|---|---|---|---|
| **Claude** (Desktop, Web, Code) | [One-click install](https://claude.ai/directory/360c0c31-4bb6-42ca-8e50-5da0a100a68e) — listed in the Claude directory | OAuth 2.0 | ~30 seconds |
| **ChatGPT** | [One-click install](https://chatgpt.com/apps/windsor-ai/asdk_app_694a52cfaa3c819192bea84eaa254968) — listed as a ChatGPT app | OAuth 2.0 | ~30 seconds |
| **Claude Code** | `claude plugin install windsor-ai` — [source](https://github.com/windsor-ai/claude-windsor-ai-plugin), with slash commands and an analyst agent | OAuth 2.0 | ~1 minute |
| **Cursor** | [windsor-ai/windsor-ai-cursor-plugin](https://github.com/windsor-ai/windsor-ai-cursor-plugin), or paste the config below | OAuth 2.0 | ~1 minute |
| **Windsurf, Cline, GitHub Copilot, Manus, n8n** | Standard MCP server config (`url: https://mcp.windsor.ai/`) | OAuth 2.0 | ~1 minute |
| **Gemini CLI** | Edit `~/.gemini/settings.json` (config below) | OAuth 2.0 | ~2 minutes |

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
[https://windsor.ai/introducing-windsor-mcp/](https://windsor.ai/introducing-windsor-mcp/)

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

## Option 3: Cursor

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

## Option 4: Gemini CLI

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
Yes, it's free during our beta phase. You'll need a Windsor.ai account with integrated data. Some LLM providers (e.g. Claude) require a paid plan to use connectors.

### What AI agents does Windsor MCP work with?
Any AI agent compatible with MCP — including Claude (Desktop, Web, Code), ChatGPT, Cursor, Windsurf, Cline, GitHub Copilot, Gemini, Manus, n8n, mcp-proxy, and custom MCP clients. See the [Supported AI clients](#-supported-ai-clients) table above.

### What can I ask Windsor MCP?
Marketing performance, sales pipelines, CRM data, e-commerce orders, payment activity, finance metrics, ad spend summaries, ROAS trends, campaign anomalies, multi-channel attribution, warehouse queries, and more. If it's in your Windsor.ai data, you can ask it.

### What data sources does Windsor MCP support?
325+ connectors across advertising, analytics, CRM, e-commerce, payments, warehouses and more. The live, complete list is at [mcp.windsor.ai/datasources](https://mcp.windsor.ai/datasources). See the [Supported data sources](#-supported-data-sources) section for highlights by category.

### Do I need to write SQL or set up dashboards?
No. Just ask your questions in plain English and get structured responses in real-time.

### How does authentication work?
OAuth 2.0 is the default — your client redirects you to Windsor.ai for a one-time login. The server implements MCP OAuth with Dynamic Client Registration; clients that support MCP OAuth (Claude, ChatGPT, Cursor, Manus) discover the endpoints automatically. For setups that only support API keys, pass your Windsor.ai key as a Bearer token via the `Authorization` header.

### Is my data secure with Windsor MCP?
Yes. All Windsor MCP tools are read-only — they fetch data, they cannot write or modify anything in your sources. Authentication is OAuth 2.0; the MCP server does not store your Windsor.ai password and does not retain query results. Data flows over TLS. Windsor.ai is SOC 2 compliant. For details, see [windsor.ai/security](https://windsor.ai/security).

### How is Windsor MCP different from other data integration tools?
Windsor MCP is a native MCP server, not a plugin or wrapper — it works in any MCP-compatible client without proxies. Windsor.ai includes warehouse and database destinations (BigQuery, Snowflake, Redshift, S3, MySQL, Postgres) on its lower-priced tiers, and the MCP server is included on all paid plans at no extra cost. See [windsor.ai/pricing](https://windsor.ai/pricing) for a feature-by-feature comparison.

### How much does Windsor.ai cost?
The MCP server is free during beta. Windsor.ai data plans start at $19/month (annual) for 3 sources. See [windsor.ai/pricing](https://windsor.ai/pricing) for the current pricing.

### Does Windsor MCP work on mobile?
The hosted server is platform-agnostic, but mobile support depends on which AI client you use. Claude Web/Desktop, ChatGPT Web/Desktop and Cursor Desktop are fully supported today; mobile MCP support varies by client and is rolling out.

### Where do I get help?
For support or feedback, contact [support@windsor.ai](mailto:support@windsor.ai). For bug reports specific to this repository, open an issue.

---

## 🧪 Beta Status
Windsor MCP is currently in beta. All features are fully functional, but you may encounter occasional quirks. We're actively improving performance, authentication, compatibility and feature coverage.

## 🧠 Try it now
Start querying your business data via Windsor MCP — fastest path is the Claude one-click install:

👉 **[Install on Claude](https://claude.ai/directory/360c0c31-4bb6-42ca-8e50-5da0a100a68e)** — 30 seconds, no API key.

Other entry points:
- 👉 [Install on ChatGPT](https://chatgpt.com/apps/windsor-ai/asdk_app_694a52cfaa3c819192bea84eaa254968)
- 👉 [Claude Code plugin](https://github.com/windsor-ai/claude-windsor-ai-plugin)
- 👉 [Cursor plugin](https://github.com/windsor-ai/windsor-ai-cursor-plugin)
- 👉 [Get a Windsor.ai API key](https://onboard.windsor.ai)
- 👉 [Watch the demo](https://youtu.be/84bBP71qvjs)
- 👉 [Live AI-targeted docs](https://mcp.windsor.ai/llms.txt)

For support or feedback, contact us at [support@windsor.ai](mailto:support@windsor.ai).
