# Example payloads

Sample JSON-RPC 2.0 tool-call payloads for the three Scribo MCP
tools. Drop these into any MCP test harness or replay them with
`curl` directly against the hosted endpoint:

```sh
curl -s https://scribo.causaprima.com/mcp \
  -H "content-type: application/json" \
  -H "accept: application/json, text/event-stream" \
  -d @examples/create_invoice_de_b2b.json
```

The first call from a new sender email triggers a magic-link email
to that address. Subsequent calls with the same email skip the
magic-link send (`magic_link_sent: false` in the response).

| File | What it does |
|---|---|
| [`create_invoice_de_b2b.json`](./create_invoice_de_b2b.json) | DE → DE B2B invoice; emits ZUGFeRD COMFORT (PDF/A-3 hybrid + EN 16931 CII XML). |
| [`create_invoice_de_b2g.json`](./create_invoice_de_b2g.json) | DE → DE B2G; `recipient.leitweg_id` forces XRechnung CII (mandatory for German federal procurement). |
| [`create_invoice_us_plain.json`](./create_invoice_us_plain.json) | US → US; emits a plain PDF (no XML). |
| [`get_invoice.json`](./get_invoice.json) | Fetch a previously-generated invoice + re-mint its signed download URL. |
| [`list_supported_jurisdictions.json`](./list_supported_jurisdictions.json) | Returns the per-country format matrix. |

All invoice payloads contain stand-in business data. Replace before
running against the production endpoint.
