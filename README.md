# Scribo MCP — public registry metadata

Public registry-facing metadata for the Scribo MCP server hosted at
**[`scribo.causaprima.ai/mcp`](https://scribo.causaprima.ai/mcp)**.

> Compliant e-invoicing for any LLM — XRechnung, ZUGFeRD, Factur-X,
> Peppol BIS UBL, Spanish Facturae, plain US PDF. Free; the sender's
> email is the login.

This repo contains:

- [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json) — Claude Code plugin manifest (one-click install via `/plugin install scribo-mcp`)
- [`server.json`](./server.json) — [MCP Registry](https://registry.modelcontextprotocol.io/) metadata
- [`schemas/`](./schemas/) — JSON Schema for each tool's input
- [`examples/`](./examples/) — sample JSON-RPC 2.0 tool-call payloads

The MCP server implementation itself lives in the Causa Prima
monorepo at [`client/scribo-mcp/`](https://github.com/causa-prima-ai/causa-prima/tree/main/client/scribo-mcp).
End-users never install it locally; they configure their MCP client
to call the hosted endpoint.

## Install

### Claude Code (recommended)

```sh
/plugin marketplace add causa-prima-ai/scribo-mcp
/plugin install scribo-mcp@scribo-mcp
# or, when the name is unambiguous across your installed marketplaces:
/plugin install scribo-mcp
```

The repo ships as a single-plugin Claude Code marketplace, so no separate marketplace setup is needed. The plugin wires up the hosted MCP server at `scribo.causaprima.ai/mcp` — no manual `.mcp.json` editing.

### Claude Desktop / claude.ai

Coming soon. We're submitting `scribo-mcp` to Anthropic's hosted plugin registry; once approved, install with one click from the in-app plugin browser. Until then, edit the config manually using the snippets below.

### Manual config (any MCP client)

| Client | Config file | Snippet |
|---|---|---|
| **Claude Desktop** | `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) | `{ "mcpServers": { "scribo": { "transport": "http", "url": "https://scribo.causaprima.ai/mcp" } } }` |
| **Cursor** | `~/.cursor/mcp.json` or workspace `.cursor/mcp.json` | `{ "mcpServers": { "scribo": { "url": "https://scribo.causaprima.ai/mcp" } } }` |
| **Cline** | `~/Library/Application Support/Cline/MCP/cline_mcp_settings.json` (macOS) | `{ "mcpServers": { "scribo": { "transport": "streamable-http", "url": "https://scribo.causaprima.ai/mcp" } } }` |
| **ChatGPT App** | OpenAI Apps SDK remote registration | Point at `https://scribo.causaprima.ai/mcp` |
| **Any MCP client** | Streamable HTTP, JSON-RPC 2.0, no auth | Point at `https://scribo.causaprima.ai/mcp` |

Restart the host after editing the config.

## Tools

| Tool | Description |
|---|---|
| `create_invoice` | Generate an EN 16931-compliant invoice. Returns a 15-minute signed download URL plus a magic-link confirmation. |
| `get_invoice` | Fetch a previously generated invoice and a fresh signed download URL. |
| `list_supported_jurisdictions` | Return the per-country format matrix (which jurisdictions can emit which formats, plus the default per country). |

Full JSON schemas: [`schemas/create_invoice.json`](./schemas/create_invoice.json),
[`schemas/get_invoice.json`](./schemas/get_invoice.json),
[`schemas/list_supported_jurisdictions.json`](./schemas/list_supported_jurisdictions.json).

Sample payloads: [`examples/`](./examples/).

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
- Full GDPR posture: [issue #148](https://github.com/CausaPrimaAI/causa-prima/issues/148).

## Documentation

- Hosted docs: [`scribo.causaprima.ai/docs`](https://scribo.causaprima.ai/docs)
- MCP-specific guide: [`scribo.causaprima.ai/docs/mcp`](https://scribo.causaprima.ai/docs/mcp)
- HTTP API contract: [`scribo.causaprima.ai/docs/api`](https://scribo.causaprima.ai/docs/api)
- Source repo: [`github.com/CausaPrimaAI/causa-prima`](https://github.com/CausaPrimaAI/causa-prima)

## License

MIT.
