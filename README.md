<img width="3840" height="1920" alt="A3 _ MCP server" src="https://github.com/user-attachments/assets/21e2cf1e-69c0-414c-9cee-180a60c17f51" />

# Scribo MCP Server

> Free, AI-native e-invoicing — for any MCP client.

## Contents

- [What is Scribo?](#what-is-scribo)
- [What is the Scribo MCP server?](#what-is-the-scribo-mcp-server)
  - [Demo](#demo)
- [Getting started](#getting-started)
  - [Quickstart](#quickstart)
  - [Setup & prerequisites](#setup--prerequisites)
- [Usage](#usage)
  - [How it works](#how-it-works)
  - [Examples](#examples)
- [Compliance](#compliance)
- [FAQ](#faq)
- [Resources](#resources)
- [License](#license)

## What is Scribo?

Scribo is a free, conversational e-invoicing tool. Describe an invoice in plain language — "bill Acme GmbH €2,400 for May design work" — and it drafts a structured invoice for you to review, download and send directly.

Just ask your AI agent about Scribo: it ships as an MCP server, a CLI, and a Claude/Codex skill, alongside a public HTTP API and a web app. Free forever — no credit card. You just need a sender email.

**Built by Causa Prima** — Scribo is built by [**Causa Prima**](https://causaprima.com/), a company building agentic AI for the CFO office. It's our first freely available skill — an early glimpse of what we're building.

## What is the Scribo MCP server?

The Scribo MCP server gives your AI assistant the ability to create invoices for you. It's a hosted [Model Context Protocol](https://modelcontextprotocol.io) endpoint — add its URL to your assistant once, then just ask for an invoice and it gathers the details and hands back a finished, compliant invoice in the chat, ready to download and send.

Because it's hosted, there's nothing to install or run locally. It works with any MCP-aware client — **Claude Desktop, Claude.ai, Cursor, Cline, ChatGPT App, the OpenAI Codex CLI**, and others. Free, no signup — your email is all you need.

### Demo

https://github.com/user-attachments/assets/3e1fcca8-580b-448f-bb1c-bee5ee016b54

## Getting started

### Quickstart

Scribo's MCP server is hosted — there's nothing to install. Point your client at:

```
https://scribo.causaprima.com/mcp
```

In **Claude Desktop or Claude.ai**: **Customize → Connectors → Add custom connector**, name it `Scribo`, paste the URL, and save. Then just ask:

> Draft an invoice for 3 days of consulting at €1,200/day to Acme GmbH.

Scribo collects the missing details, verifies your sender email once, and returns a download link.

Using a different client? See the per-client setup below.

### Setup & prerequisites

The server speaks MCP over **Streamable HTTP** (JSON-RPC 2.0, MCP spec ≥ 2025-03-26) with **no auth**. Any MCP-aware client connects with one setting — the URL `https://scribo.causaprima.com/mcp`.

**Claude Desktop / Claude.ai** — Customize → Connectors → Add custom connector → name `Scribo`, paste the URL. On Team/Enterprise plans an org owner may need to add it once under Organization settings → Connectors.

**Cursor** — add to `~/.cursor/mcp.json` (or workspace `.cursor/mcp.json`):

```json
{ "mcpServers": { "scribo": { "url": "https://scribo.causaprima.com/mcp" } } }
```

**Cline** — add to `cline_mcp_settings.json`:

```json
{ "mcpServers": { "scribo": { "transport": "streamable-http", "url": "https://scribo.causaprima.com/mcp" } } }
```

**OpenAI Codex CLI** — add to `~/.codex/config.toml`, then restart Codex:

```toml
[mcp_servers.scribo]
url = "https://scribo.causaprima.com/mcp"
```

**Any other MCP client** — configure a remote server at the URL above, transport Streamable HTTP, no auth.

> Prefer a plain-bash setup with no connector in the loop? The [Scribo skill](https://github.com/causa-prima-ai/scribo-skill) covers Claude Code, Claude.ai, Claude Desktop, and Codex via simple helper scripts.

**Verify.** Ask your assistant: _"list the jurisdictions Scribo supports."_

## Usage

### How it works

You describe the invoice in plain language; your assistant calls Scribo's tools to build it. The server exposes four tools:

| Tool | What it does |
|---|---|
| `create_invoice` | Generates a compliant invoice from the sender, recipient, and line-item details, and returns a durable download link. |
| `verify_email_code` | Confirms ownership of the sender email using the 6-digit code Scribo emails on the first invoice. |
| `get_invoice` | Re-fetches a previously generated invoice and a fresh download link. |
| `list_supported_jurisdictions` | Lists which countries and formats are available. |

A typical flow: you ask for an invoice → your assistant calls `create_invoice` and collects anything missing → on your first invoice from a new sender email, Scribo asks you to paste a 6-digit code (confirmed via `verify_email_code`) → your assistant retries and hands back a download link — a PDF with the matching EN 16931 XML embedded. On clients that render widgets (e.g. the ChatGPT App), the invoice appears as a card with a download button.

The server also registers five workflow **prompts** (`generate_invoice`, `german_b2b_invoice`, `german_b2g_xrechnung`, `us_plain_pdf_invoice`, `redownload_invoice`) — clients that surface prompts show them as slash-command-style starters.

Full tool schemas and sample payloads are in [`schemas/`](./schemas) and [`examples/`](./examples), and in the [MCP docs](https://scribo.causaprima.com/docs/mcp).

### Examples

Things you can say to your assistant once Scribo is connected:

**German B2B invoice (ZUGFeRD)**

> Invoice Beispiel GmbH for 12 hours of web development at €110/hour, 19% VAT. I'm Example GmbH, VAT ID DE123456789.

Returns a ZUGFeRD invoice — a PDF a human can read, with EN 16931 XML embedded for the recipient's accounting system.

**German public-sector invoice (XRechnung)**

> Same client, but it's for the City of Munich — the Leitweg-ID is 04011000-1234512345-06.

A Leitweg-ID tells Scribo this is a B2G invoice, so it produces XRechnung, the format German public authorities require.

**US invoice (plain PDF)**

> Bill Acme Inc. $4,000 for a brand strategy workshop.

A clean PDF invoice — no e-invoice XML, since the US has no e-invoicing mandate.

**Re-download a past invoice**

> Send me the download link for the invoice I made for Beispiel GmbH last week.

Scribo returns a fresh download link for an invoice you've already generated.

## Compliance

Scribo emits invoices conforming to **EN 16931**, the European e-invoicing standard, with the relevant national CIUS. Every EN 16931 output (ZUGFeRD, XRechnung) is validated against the **Invopop**-hosted validator at generate-time — output that fails the rule set never reaches the user. US plain PDFs carry no EN 16931 XML and are rendered directly.

**Supported formats**

| Jurisdiction | Format | Status |
|---|---|---|
| Germany (B2B) | **ZUGFeRD** COMFORT (PDF/A-3 hybrid + CII XML) | ✅ Live |
| Germany (B2G) | **XRechnung** (UBL / CII) | ✅ Live |
| United States | Plain PDF (no XML, no e-invoice claim) | ✅ Live |
| France | **Factur-X** EN 16931 | 🔜 Coming soon |
| Spain | **Facturae** | 🔜 Coming soon |
| Belgium / NL / LU / AT | **Peppol BIS 3.0** UBL | 🔜 Coming soon |

*Disclaimer: Scribo generates and validates compliant invoice documents. It is **not tax or legal advice** — Scribo does not determine your tax obligations, VAT treatment, or filing requirements.*

## FAQ

<details>
<summary><strong>Is Scribo really free?</strong></summary>

Yes — free forever. No credit card, no subscription, no paywall before your first invoice. You just need a sender email.

</details>

<details>
<summary><strong>Do I need an account or signup?</strong></summary>

No signup form. On your first invoice, Scribo verifies the sender email — a 6-digit code (or one-click link) arrives at that address; one verification covers ~30 minutes of invoicing. The same email doubles as your magic-link login for re-downloads later.

</details>

<details>
<summary><strong>Which countries and formats are supported?</strong></summary>

Live today: **Germany** — ZUGFeRD (B2B) and XRechnung (B2G) — and the **United States** (plain PDF). Coming next: **France** (Factur-X), **Spain** (Facturae), and **Belgium / NL / LU / AT** (Peppol BIS 3.0).

</details>

<details>
<summary><strong>What does "EN 16931-compliant" actually mean here?</strong></summary>

Every EN 16931 output (ZUGFeRD, XRechnung) is validated against the **EN 16931-1:2017** rule set (via the Invopop-hosted validator) before it's returned. Output that fails validation never reaches you. US plain PDFs carry no EN 16931 XML, so no schematron validation applies.

</details>

<details>
<summary><strong>Is the US version compliant?</strong></summary>

Yes. There is no US e-invoicing mandate, so Scribo produces a fully compliant plain PDF.

</details>

<details>
<summary><strong>Does Scribo give tax or legal advice?</strong></summary>

No. Scribo generates and validates compliant invoice *documents*. It does not determine your tax obligations, VAT treatment, or filing requirements.

</details>

<details>
<summary><strong>How do AI agents / LLMs use Scribo?</strong></summary>

Scribo is built to be operated by an agent. It ships as an **MCP server**, a **CLI**, and a **Claude/Codex skill**, plus a public **HTTP API** and a **web app** — all on the same backend. An agent can discover Scribo, create an invoice, and return the file on a user's behalf.

</details>

<details>
<summary><strong>Who builds Scribo?</strong></summary>

[Causa Prima](https://causaprima.com/) — a company building agentic AI for the CFO office. Operated by Causa Prima Germany GmbH, Munich.

</details>

## Resources

- **Web app** — [scribo.causaprima.com](https://scribo.causaprima.com)
- **Documentation** — [scribo.causaprima.com/docs](https://scribo.causaprima.com/docs)
- **Compliance & trust** — [scribo.causaprima.com/compliance](https://scribo.causaprima.com/compliance)
- **Causa Prima** — [causaprima.com](https://causaprima.com)

**Other Scribo surfaces**

- Skill (Claude / Codex) — [`scribo-skill`](https://github.com/causa-prima-ai/scribo-skill) · [docs](https://scribo.causaprima.com/docs/skill)
- MCP server — [`scribo-mcp`](https://github.com/causa-prima-ai/scribo-mcp) · [docs](https://scribo.causaprima.com/docs/mcp)
- CLI — [`scribo-cli`](https://github.com/causa-prima-ai/scribo-cli) · [docs](https://scribo.causaprima.com/docs/cli)
- HTTP API — [`scribo-api-docs`](https://github.com/causa-prima-ai/scribo-api-docs) · [docs](https://scribo.causaprima.com/docs/api)
- Brand hub — [`scribo`](https://github.com/causa-prima-ai/scribo)

## License

Proprietary — `UNLICENSED`. © Causa Prima Germany GmbH. All rights reserved. Distributed for use against the public Scribo API; not open-source. See [LICENSE](./LICENSE).

