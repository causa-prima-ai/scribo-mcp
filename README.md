# Scribo MCP — public registry metadata

<img src="./icon.svg" alt="Scribo" width="96" align="right" />

Public registry-facing metadata for the Scribo MCP server hosted at
**[`scribo.causaprima.ai/mcp`](https://scribo.causaprima.ai/mcp)**.

> Compliant e-invoicing for any LLM — XRechnung, ZUGFeRD, Factur-X,
> Peppol BIS UBL, Spanish Facturae, plain US PDF. Free; the sender's
> email is the login.

This repo contains:

- [`server.json`](./server.json) — [MCP Registry](https://registry.modelcontextprotocol.io/) metadata
- [`schemas/`](./schemas/) — JSON Schema for each tool's input
- [`examples/`](./examples/) — sample JSON-RPC 2.0 tool-call payloads
- [`icon.svg`](./icon.svg) — server brand mark

The MCP server is a **remote, hosted** Streamable HTTP endpoint — end
users never install it locally. Configure your MCP client to call the
hosted URL and you're done.

## Install in your MCP client

| Client | Config file | Snippet |
|---|---|---|
| **Claude Desktop** | `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) | `{ "mcpServers": { "scribo": { "transport": "http", "url": "https://scribo.causaprima.ai/mcp" } } }` |
| **Cursor** | `~/.cursor/mcp.json` or workspace `.cursor/mcp.json` | `{ "mcpServers": { "scribo": { "url": "https://scribo.causaprima.ai/mcp" } } }` |
| **Cline** | `~/Library/Application Support/Cline/MCP/cline_mcp_settings.json` (macOS) | `{ "mcpServers": { "scribo": { "transport": "streamable-http", "url": "https://scribo.causaprima.ai/mcp" } } }` |
| **ChatGPT App** | OpenAI Apps SDK remote registration | Point at `https://scribo.causaprima.ai/mcp` |
| **OpenAI Codex CLI** | `~/.codex/config.toml` | `[mcp_servers.scribo]`<br>`url = "https://scribo.causaprima.ai/mcp"` |
| **Any MCP client** | Streamable HTTP, JSON-RPC 2.0, no auth | Point at `https://scribo.causaprima.ai/mcp` |

Restart the host after editing the config.

## Tools

Each tool ships with MCP `annotations` so the host can render the
right confirmation UI — `create_invoice` triggers a write-warning;
`get_invoice` and `list_supported_jurisdictions` are read-only and
safe to call freely.

| Tool | Read-only | Idempotent | Description |
|---|---|---|---|
| `create_invoice` | no | no¹ | Generate an EN 16931-compliant invoice. Returns a durable signed download URL plus a magic-link confirmation. |
| `get_invoice` | yes | yes | Fetch a previously generated invoice and a fresh signed download URL. Tenant-scoped — cross-tenant lookups return 404. |
| `list_supported_jurisdictions` | yes | yes | Return the per-country format matrix (which jurisdictions can emit which formats, plus the default per country). |

¹ `create_invoice` becomes idempotent when the optional `idempotency_key` argument is passed — same key + same payload returns the original invoice.

Full JSON schemas: [`schemas/create_invoice.json`](./schemas/create_invoice.json),
[`schemas/get_invoice.json`](./schemas/get_invoice.json),
[`schemas/list_supported_jurisdictions.json`](./schemas/list_supported_jurisdictions.json).

Sample payloads: [`examples/`](./examples/).

## Workflow prompts

Scribo exposes five MCP prompts you can invoke directly from your
host's prompt/slash-command picker — each renders a single user-turn
message that primes the agent for a specific Scribo workflow:

| Prompt | Arguments | What it does |
|---|---|---|
| `generate_invoice` | `jurisdiction?`, `description?` | Generic walk-through; falls back to the EN 16931 format priority chain when no jurisdiction is supplied. |
| `german_b2b_invoice` | `hours?`, `hourly_rate?` | ZUGFeRD COMFORT starter; calls out BR-DE-5 / BR-DE-6 contact-name + phone. |
| `german_b2g_xrechnung` | `leitweg_id?` | XRechnung CII starter; `recipient.leitweg_id` alone forces the format. |
| `us_plain_pdf_invoice` | _(none)_ | US plain PDF; no XML, no compliance friction. |
| `redownload_invoice` | `invoice_id` (required) | Fetch a prior invoice's metadata and a freshly-signed download URL. |

## Format coverage

| Jurisdiction | Default format | Notes |
|---|---|---|
| Germany (B2B) | ZUGFeRD COMFORT (PDF/A-3 hybrid + EN 16931 CII XML) | `zugferd_basic` available via `format_override` |
| Germany (B2G) | XRechnung CII | Triggered by `recipient.leitweg_id` |
| France | Factur-X | PDF/A-3 hybrid + EN 16931 CII XML |
| Spain | Facturae 3.2.2 | Spanish-specific XML syntax |
| Belgium / NL / LU / AT | Peppol BIS 3.0 UBL | Mandate transmission requires a Peppol AP partner |
| United States | Plain PDF | No XML |

## Account model

Scribo doesn't ask you to sign up. Your `sender.contact_email` on the
first `create_invoice` call is your login. Scribo emails a magic link
to that address — clicking it binds a web session at
[`scribo.causaprima.ai`](https://scribo.causaprima.ai) so you can come
back, re-download, or upgrade to a full Causa Prima account.

## Compliance & privacy

- EN 16931 conformance verified at generate-time by Invopop's hosted
  validator; in CI by Mustang (Apache 2.0 reference validator), KoSIT
  (XRechnung), and veraPDF (PDF/A-3).
- Sender + recipient + invoice content flow through Invopop (Madrid).
  DPA on file; sub-processor listed on
  [`/compliance`](https://scribo.causaprima.ai/compliance).

## Documentation

- Hosted docs: [`scribo.causaprima.ai/docs`](https://scribo.causaprima.ai/docs)
- MCP-specific guide: [`scribo.causaprima.ai/docs/mcp`](https://scribo.causaprima.ai/docs/mcp)
- HTTP API contract: [`scribo.causaprima.ai/docs/api`](https://scribo.causaprima.ai/docs/api)

## License

MIT.
