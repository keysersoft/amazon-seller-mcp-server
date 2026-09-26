# Amazon Seller Central MCP Server

**Connect Amazon Seller Central to Claude, ChatGPT and Copilot: orders, catalog, FBA inventory, fees as MCP tools.** Powered by [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

Amazon Seller Central MCP Server gives Claude, ChatGPT, Copilot and Cursor 15 tools for Amazon Seller Central: orders, catalog, FBA inventory, fees. 14 tools read and 1 can change data. It runs on AnythingMCP: one click on AnythingMCP Cloud, or self-hosted with Docker. Credentials are stored encrypted and every call is audited.

**Last verified:** 2026-07-17 against the Selling Partner API sandbox (EU) (LWA token refresh and getMarketplaceParticipations through the AnythingMCP engine; production calls need your app's production authorization).  
**Adapter synced:** <!-- synced -->2026-09-26

Maintained by [@keysersoft](https://github.com/keysersoft), an AnythingMCP maintainer. Built on [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp) by helpcode.ai.

## Quick start (AnythingMCP Cloud)

1. Sign in at [cloud.anythingmcp.com](https://cloud.anythingmcp.com) and open the [install link](https://cloud.anythingmcp.com/connectors/store?install=amazon-seller).
2. Enter `SPAPI_CLIENT_ID`, `SPAPI_CLIENT_SECRET`, `SPAPI_REFRESH_TOKEN`, `SPAPI_ENDPOINT`, `SPAPI_MARKETPLACE_ID`, `SPAPI_SELLER_ID` (see [Authentication](#authentication)).
3. Copy the URL of your MCP server under **MCP Servers** and add it to your AI client ([below](#connect-claude-chatgpt-copilot-or-cursor)).

AnythingMCP Cloud is the same open-source code, operated by helpcode.ai in Frankfurt, Germany.

## Self-hosted (Docker)

Needs Docker 24+, openssl and Node 18+.

```bash
git clone https://github.com/keysersoft/amazon-seller-mcp-server.git
cd amazon-seller-mcp-server
./scripts/install.sh
```

`install.sh` writes `.env` with fresh secrets, starts AnythingMCP, creates the first admin, installs the connector if `SPAPI_CLIENT_ID` and `SPAPI_CLIENT_SECRET` and `SPAPI_REFRESH_TOKEN` and `SPAPI_ENDPOINT` and `SPAPI_MARKETPLACE_ID` and `SPAPI_SELLER_ID` are set in `.env` and creates an MCP API key. Without credentials it prints the install link instead: `http://localhost:3000/connectors/store?install=amazon-seller`. Then check the whole chain:

```bash
npm install && node scripts/smoke.mjs
```

## Connect Claude, ChatGPT, Copilot or Cursor

- **Claude (claude.ai, Desktop, mobile):** *Customize → Connectors → Add custom connector*, paste your MCP server URL and sign in. Claude connects from Anthropic's cloud, so the URL must be public HTTPS: your AnythingMCP Cloud URL, or your own instance behind TLS.
- **Claude Code:**

  ```bash
  claude mcp add --transport http amazon-seller-mcp-server http://localhost:4000/mcp --header "X-API-Key: <MCP_API_KEY>"
  ```
- **Cursor** (`.cursor/mcp.json`) and **VS Code / GitHub Copilot** (`.vscode/mcp.json`, key `servers` instead of `mcpServers`, plus `"type": "http"`):

  ```json
  { "mcpServers": { "amazon-seller-mcp-server": { "url": "http://localhost:4000/mcp", "headers": { "X-API-Key": "<MCP_API_KEY>" } } } }
  ```
- **ChatGPT:** add the public HTTPS URL as a connector (app) in ChatGPT's settings. A `localhost` URL does not work there.

## Tools

15 tools, generated from [`adapter/amazon-seller.json`](adapter/amazon-seller.json). **read** tools cannot change anything in the source system.

<!-- tools:start (generated from adapter/*.json, do not edit) -->
| Tool | What it does | Access |
|---|---|---|
| `amazon_marketplace_ids` | Reference table of Amazon marketplace IDs and the regional SP-API endpoint that serves each. | read |
| `amazon_marketplace_participations` | List the marketplaces this seller account participates in (id, country, currency, store name). | read |
| `amazon_list_orders` | List orders in the configured marketplace. | read |
| `amazon_get_order` | Fetch a single order by its AmazonOrderId (format 3-7-7, e.g. | read |
| `amazon_get_order_items` | List the line items of an order: SellerSKU, ASIN, title, quantity ordered/shipped, item price, taxes, promotion discounts. | read |
| `amazon_search_catalog` | Search the Amazon product catalog of the configured marketplace by keywords or identifiers (ASIN/EAN/GTIN/UPC/ISBN). | read |
| `amazon_get_catalog_item` | Fetch a single catalog item by ASIN with the requested data sets (summaries, attributes, images, salesRanks, dimensions…). | read |
| `amazon_fba_inventory` | FBA inventory summaries for the configured marketplace: fulfillable, inbound, reserved, unfulfillable and researching quantities per SKU/ASIN. | read |
| `amazon_get_item_offers` | Current offers for an ASIN in the configured marketplace: buy-box winner, lowest prices by fulfillment channel, offer counts, shipping. | read |
| `amazon_get_fees_estimate` | Estimate the Amazon fees (referral fee, FBA fulfillment fee…) you would pay for an ASIN at a given selling price. | read |
| `amazon_list_financial_events` | Financial events (shipments settlements, refunds, fees, service charges, adjustments) posted in a date window. | read |
| `amazon_get_listings_item` | Fetch one of YOUR listings by seller SKU: summaries, attributes, issues (quality/compliance problems), offers and fulfillment availability. | read |
| `amazon_create_report` | Request generation of a report (step 1 of 3). | write |
| `amazon_get_report` | Check a report's processing status (step 2 of 3). | read |
| `amazon_get_report_document` | Get the download descriptor of a finished report (step 3 of 3). | read |
<!-- tools:end -->

## Example prompts

- Which orders came in yesterday on amazon.de, and which of them are not shipped yet?
- Show the items of order 302-1234567-1234567.
- How much FBA stock do I have for SKU A-1043, and how much is inbound?
- Who holds the buy box for ASIN B0C1234567, and at what price?
- What would Amazon charge in fees if I sell ASIN B0C1234567 at 24.90 EUR with FBA?
- Which of my listings have quality or compliance issues?

More in [examples/prompts.md](examples/prompts.md).

## Authentication

This connector wraps Amazon's **Selling Partner API (SP-API)** — the official successor of MWS — using Login-with-Amazon (LWA) OAuth2. No AWS keys or SigV4 signing are needed (Amazon removed that requirement).

## Setup (one-time, ~15 min + Amazon approval)

1. **Register as a developer**: Seller Central → Apps & Services → *Develop Apps* (you may first need to complete the developer profile in the Solution Provider Portal; Amazon reviews it manually, allow a few days).
2. **Create a private app**: *Develop Apps* → **Add new app client** → type *Sellers* → select the roles you need (e.g. "Inventory and Order Tracking", "Pricing", "Amazon Fulfillment", "Finance and Accounting", "Selling Partner Insights"). PII roles (buyer names/addresses) are restricted and NOT required for this connector.
3. **Get LWA credentials**: on the app row choose *View* → copy **Client ID** (`amzn1.application-oa2-client.…`) and **Client Secret** (`amzn1.oa2-cs.v1.…`).
4. **Self-authorize**: on the app row choose **Authorize** → Amazon shows the **Refresh Token** (`Atzr|…`). This token is long-lived; the connector exchanges it automatically for 1-hour access tokens (sent in the `x-amz-access-token` header — handled for you).
5. Fill the env vars:
   - `SPAPI_CLIENT_ID`, `SPAPI_CLIENT_SECRET`, `SPAPI_REFRESH_TOKEN` — from steps 3-4.
   - `SPAPI_ENDPOINT` — your region's API host:
     - Europe (DE/UK/IT/FR/ES/NL/SE/PL/TR/AE/IN/…): `https://sellingpartnerapi-eu.amazon.com`
     - North America (US/CA/MX/BR): `https://sellingpartnerapi-na.amazon.com`
     - Far East (JP/AU/SG): `https://sellingpartnerapi-fe.amazon.com`
   - `SPAPI_MARKETPLACE_ID` — the marketplace you sell in (call `amazon_marketplace_ids` for the full table; e.g. DE `A1PA6795UKMFR9`, US `ATVPDKIKX0DER`, IT `APJ6JRA9NG5V4`).
   - `SPAPI_SELLER_ID` — your merchant token (Seller Central → Account Info → *Merchant Token*). Only needed for `amazon_get_listings_item`; set any placeholder otherwise.

**First call**: run `amazon_marketplace_participations` — it needs no parameters and confirms auth + region are correct.

## Sandbox

Set `SPAPI_ENDPOINT` to `https://sandbox.sellingpartnerapi-eu.amazon.com` (or `-na`/`-fe`) to hit the **static sandbox**: it returns canned responses when requests match predefined parameter patterns (e.g. Orders requires `CreatedAfter=TEST_CASE_200`). Same LWA credentials work; a draft app's refresh token may be sandbox-only (production returns 403 until the authorization is production-grade).

## Marketplace scoping

All tools are pinned to `SPAPI_MARKETPLACE_ID`. To work across multiple marketplaces of the same region, create one connector per marketplace (cheap) — the underlying seller account and credentials can be identical.

## What v1 deliberately does NOT do

- **Buyer PII** (names, shipping addresses, phone numbers): requires a Restricted Data Token and an Amazon-approved restricted role. Order data returned here is the non-PII view (IDs, status, totals, SKUs, quantities).
- **Writes** (price updates, listings edits, feeds): planned for a later version once the read workflows are proven.

## Pagination & rate limits

List endpoints return a `NextToken`/`pageToken` — pass it back to fetch the next page. SP-API rate limits are per-endpoint (typically 0.0055–2 req/s with small bursts); on `429` the connector retries automatically with backoff, but space out heavy report loops.

## Reports flow (3 steps)

1. `amazon_create_report` (e.g. `reportType: "GET_FLAT_FILE_OPEN_LISTINGS_DATA"`) → returns `reportId`.
2. Poll `amazon_get_report` until `processingStatus` = `DONE` → gives `reportDocumentId`.
3. `amazon_get_report_document` → returns a **pre-signed download `url`** (valid ~5 min; document may be gzip). Fetch that URL yourself to read the content.

## Security

- **Read or write is your choice.** 14 of the tools only read; `amazon_create_report` can change data. Assign the connector to an MCP server whose role whitelists only the tools you want, and the rest are invisible to that client.
- **Credentials** are encrypted with AES-256-GCM and never shown to the model.
- **Response mapping** drops or reshapes fields per tool before they reach the model, e.g. bank details or personal data.
- **Audit log:** every call is recorded with input, output, duration and status, in your own database when self-hosted.
- **SSO, RBAC and SCIM** are included in the self-hosted build.

## FAQ

### Is there an Amazon Seller Central MCP server?
Yes, this one. It connects the official Selling Partner API (SP-API) to Claude, ChatGPT and Copilot through AnythingMCP: 15 tools for orders, catalog, FBA inventory, offers, fee estimates, financial events, listings and reports.

### What do I need to connect it?
A private Selling Partner API app (Seller Central → Apps & Services → Develop Apps), its LWA client ID and secret, a refresh token from authorising the app for your seller account, the regional endpoint and your marketplace and seller IDs. The Authentication section lists each one.

### Does it work in the SP-API sandbox?
Yes. Point the endpoint at the sandbox host to try it; the sandbox returns Amazon's canned responses. Real data needs the app's production authorization.

### Can the AI change prices or listings?
No. This connector only reads, apart from requesting a report. Price and listing updates are not part of it.

### Can it see buyer names and addresses?
No. That personal data needs Amazon's Restricted Data Token flow, which this connector does not use, so orders come back without buyer PII.

### Does it work with ChatGPT and Copilot?
Yes. The same MCP server works in ChatGPT (with a public HTTPS URL such as AnythingMCP Cloud), GitHub Copilot in VS Code, Cursor and Claude Code.

## Troubleshooting

| Problem | Fix |
|---|---|
| `401` / `403` from the vendor | The credentials are wrong or lack rights. Re-enter them on the connector page; the import runs a test call and shows the result. |
| Tools missing in the AI client | The connector is not assigned to the MCP server the client uses. Check **MCP Servers**, then run `node scripts/smoke.mjs`. |
| The host is on your internal network | Self-host AnythingMCP on that network and add the hostname to `SSRF_ALLOWED_HOSTS`, or the outbound guard blocks the call. |
| Works locally, fails on AnythingMCP Cloud | The system must be reachable from the internet with a valid TLS certificate. |

## Related

- [ecommerce-mcp-server](https://github.com/HelpCode-ai/ecommerce-mcp-server): E-commerce MCP server: connect Amazon, eBay, WooCommerce, Shopware, Kaufland, OTTO and 7 more to Claude & ChatGPT.
- [kaufland-mcp-server](https://github.com/kochfreiburg/kaufland-mcp-server): Kaufland Marketplace MCP server: Claude & ChatGPT read your Kaufland seller orders, units, shipments, tickets and storefronts.
- [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp): the open-source MCP server and gateway this repository is built on.

## License

AGPL-3.0-only. The adapter definition in `adapter/` comes from AnythingMCP (AGPL-3.0).
