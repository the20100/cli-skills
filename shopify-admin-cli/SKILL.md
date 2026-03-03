---
name: shopify-admin
version: 2.0.0
description: Use when the user wants to manage their Shopify store — products, collections, orders, customers, inventory, metafields, metaobjects, webhooks, discounts, analytics, fulfillments, markets, or store settings — using the shopify-admin CLI. Trigger on requests like "list my products", "show Shopify orders", "create a product", "check inventory", "run a ShopifyQL query", "set a metafield", "list webhooks", etc.
---

# Shopify Admin CLI

The `shopify-admin` CLI manages a Shopify store via the Shopify Admin GraphQL API (2026-01).

Run commands with the Bash tool. Find the binary by running `which shopify-admin` or look in the standard PATH. The binary name is `shopify-admin`.

**If the `shopify-admin` binary is not found**, install it first:

```bash
# Check if available
which shopify-admin

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/shopify-admin-cli
cd shopify-admin-cli
go build -o shopify-admin .
mv shopify-admin /usr/local/bin/
cd ..
rm -rf shopify-admin-cli

# If found, check if the last version is installed
shopify-admin update
```

**Authentication (new in v2):**
Shopify no longer issues permanent tokens from the admin UI. You must use OAuth with a custom app (Client ID + Client Secret from the Shopify Partners dashboard or Dev Dashboard).

Recommended setup (one-time, per store):
```bash
shopify-admin auth configure <client-id> <client-secret>
shopify-admin auth login --shop <shop> --no-browser
```
After that, **tokens are auto-refreshed silently** on every command (tokens last 24 h; the CLI refreshes via Client Credentials Grant when needed).

Credentials are stored in:
- macOS: `~/Library/Application Support/shopify-admin/config.json`
- Linux: `~/.config/shopify-admin/config.json`
- Windows: `%AppData%\shopify-admin\config.json`

**Important:** If `SHOPIFY_ACCESS_TOKEN` and `SHOPIFY_SHOP` env vars are set they take priority over the config file and bypass auto-refresh. Remove them if you want the config-based OAuth flow to work.

---

## Environment variables

**Access Token** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `SHOPIFY_ACCESS_TOKEN` | Primary (canonical) |
| `SHOPIFY_TOKEN` | Short form |
| `SHOPIFY_API_TOKEN` | API token form |
| `SHOPIFY_API_KEY` | API key form |
| `SHOPIFY_KEY` | Short key form |
| `SHOPIFY_API` | Short form |
| `API_KEY_SHOPIFY` | Reversed API key |
| `API_SHOPIFY` | Short reversed |
| `SHOPIFY_SECRET_KEY` | Secret key form |
| `SHOPIFY_API_SECRET` | API secret form |
| `SHOPIFY_SECRET` | Short secret |
| `SHOPIFY_SK` | Secret key shorthand (suffix) |
| `SK_SHOPIFY` | Secret key shorthand (prefix) |

**Shop Domain** — the following env var names are accepted:

| Variable | Notes |
|----------|-------|
| `SHOPIFY_SHOP` | Primary (canonical) |
| `SHOPIFY_STORE` | Store naming |
| `SHOPIFY_DOMAIN` | Domain naming |
| `SHOPIFY_SHOP_URL` | URL form |
| `SHOPIFY_STORE_URL` | Store URL form |
| `SHOP_DOMAIN` | Short form |

Both token AND shop must be set for env var resolution to work.

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |

Output is **auto-detected**: JSON when piped, human-readable tables in terminal.

---

## search-syntax  ← USE THIS BEFORE FILTERING

**Always call this before constructing a `--query` string** to know what fields and values are valid for each resource. No authentication required.

```bash
# List all filterable fields for every resource
shopify-admin search-syntax --json

# Fields for a specific resource (structured JSON, ideal for agents)
shopify-admin search-syntax products --json
shopify-admin search-syntax orders   --json
shopify-admin search-syntax customers --json
shopify-admin search-syntax collections --json
shopify-admin search-syntax inventory --json
shopify-admin search-syntax discounts --json
```

**JSON response structure:**
```json
{
  "resource": "orders",
  "command": "shopify-admin orders list --query \"...\"",
  "fields": [
    { "field": "financial_status", "type": "enum",   "examples": ["financial_status:paid", "financial_status:pending"] },
    { "field": "fulfillment_status","type": "enum",   "examples": ["fulfillment_status:unfulfilled", "fulfillment_status:fulfilled"] },
    { "field": "email",             "type": "string", "examples": ["email:john@example.com"] },
    { "field": "created_at",        "type": "date",   "examples": ["created_at:>2024-01-01", "created_at:<2024-12-31"] }
  ],
  "operators": [
    "field:value              exact match",
    "field:>value             greater than (dates, numbers)",
    "field:<value             less than",
    "field:value1 field:value2  AND (space-separated)",
    "(field:a OR field:b)     OR",
    "NOT field:value          negation",
    "field:\"multi word\"      phrase — use quotes"
  ]
}
```

**Composing queries from the fields list:**
- Use `field` from `fields[]` as the left side of the filter
- Use `examples[]` as templates — replace the value part
- Multiple filters → join with a space (AND): `"financial_status:paid fulfillment_status:unfulfilled"`
- OR logic → wrap in parentheses: `"(status:open OR status:closed)"`
- Date range → two filters: `"created_at:>2024-01-01 created_at:<2024-12-31"`
- Phrases with spaces → use double quotes: `vendor:\"Acme Corp\"`

---

## info

Show binary location, config path, active credential source, and environment.

```bash
shopify-admin info
```

---

## auth

### OAuth flow (recommended — works with Shopify's new custom app model)

```bash
# Step 1 — save app credentials from the Partners / Dev Dashboard
shopify-admin auth configure <client-id> <client-secret>

# Step 2 — install the app and obtain the API token
#   --no-browser: for remote/headless environments (prints URL, prompts for callback URL)
#   (omit --no-browser to open a browser automatically on local machines)
shopify-admin auth login --shop <shop> --no-browser
shopify-admin auth login --shop <shop>

# Optional flags for auth login:
#   --scopes <scopes>   comma-separated OAuth scopes (default: all CLI scopes)
```

**How `--no-browser` works:**
1. CLI prints the Shopify authorization URL — open it in any browser
2. Approve the app in Shopify admin
3. Browser redirects to `http://localhost/callback?code=...` (will fail to load — that's expected)
4. Copy the full URL from the browser address bar and paste it into the CLI prompt
5. CLI exchanges the code, then calls Client Credentials Grant to get the real API token

**Token auto-refresh:** Once `client_id`, `client_secret`, and `shop` are in config, every CLI command transparently refreshes the token when it is missing, expired (within 5 min), or is an `shpc_` session token.

### Manual token (legacy / env-var override)

```bash
shopify-admin auth setup <shop> <access-token>   # Manually save a shop + token
```

### Status & logout

```bash
shopify-admin auth status    # Show active credential source, token, and Client ID
shopify-admin auth logout    # Remove all saved credentials from config
```

---

## shop

### `shop info`
Show store name, email, domain, currency, country, plan, and creation date.
```bash
shopify-admin shop info
shopify-admin shop info --json
```

---

## products

### `products list`
List products. Supports Shopify search syntax via `--query`.
```bash
shopify-admin products list
shopify-admin products list --query "status:active"
shopify-admin products list --query "vendor:Nike"
shopify-admin products list --first 20 --after CURSOR
shopify-admin products list --json | jq '.[].id'
```

### `products get <id>`
Get product details including all variants.
```bash
shopify-admin products get 1234567890
```

### `products create <title>`
```bash
shopify-admin products create "T-Shirt"
shopify-admin products create "T-Shirt" --vendor Nike --status active --tags "apparel,clothing"
shopify-admin products create "T-Shirt" --type "Shirts" --desc "<p>Description</p>"
```
Flags: `--vendor`, `--type`, `--status` (active/draft/archived), `--tags` (comma-separated), `--desc` (HTML)

### `products update <id>`
Update only the fields you specify.
```bash
shopify-admin products update 1234567890 --status archived
shopify-admin products update 1234567890 --title "New Title" --vendor "New Vendor"
```
Flags: `--title`, `--vendor`, `--type`, `--status`, `--tags`, `--desc`

### `products delete <id>`
Permanently delete a product (irreversible).
```bash
shopify-admin products delete 1234567890
```

### `products variants get <variant-id>`
```bash
shopify-admin products variants get 9876543210
```

### `products variants update <variant-id>`
```bash
shopify-admin products variants update 9876543210 --price 29.99
shopify-admin products variants update 9876543210 --sku MY-SKU-001 --barcode 123456789
```
Flags: `--price`, `--sku`, `--barcode`

---

## collections

### `collections list`
```bash
shopify-admin collections list
shopify-admin collections list --query "title:Summer"
```

### `collections get <id>`
```bash
shopify-admin collections get 1234567890
```

### `collections create <title>`
```bash
shopify-admin collections create "Summer Sale" --desc "<p>Best summer items</p>"
```

### `collections update <id>`
```bash
shopify-admin collections update 1234567890 --title "New Name"
```

### `collections delete <id>`
```bash
shopify-admin collections delete 1234567890
```

---

## orders

### `orders list`
```bash
shopify-admin orders list
shopify-admin orders list --query "financial_status:paid"
shopify-admin orders list --query "fulfillment_status:unfulfilled"
shopify-admin orders list --query "created_at:>2024-01-01"
shopify-admin orders list --first 20 --json
```

### `orders get <id>`
```bash
shopify-admin orders get 1234567890
```
Shows order details, line items, customer, and shipping address.

### `orders close <id>`
```bash
shopify-admin orders close 1234567890
```

### `orders cancel <id>`
```bash
shopify-admin orders cancel 1234567890
shopify-admin orders cancel 1234567890 --reason customer --refund --restock
```
Flags: `--reason` (customer/fraud/inventory/declined/other), `--refund`, `--restock`

### `orders mark-paid <id>`
```bash
shopify-admin orders mark-paid 1234567890
```

---

## customers

### `customers list`
```bash
shopify-admin customers list
shopify-admin customers list --query "email:john@example.com"
shopify-admin customers list --query "state:enabled"
```

### `customers get <id>`
```bash
shopify-admin customers get 1234567890
```

### `customers create`
At least one of `--email` or `--phone` is required.
```bash
shopify-admin customers create --email john@example.com --first John --last Doe
shopify-admin customers create --email john@example.com --phone "+15551234567" --tags "vip"
```

### `customers update <id>`
```bash
shopify-admin customers update 1234567890 --email new@example.com
shopify-admin customers update 1234567890 --tags "wholesale,vip"
```

### `customers delete <id>`
```bash
shopify-admin customers delete 1234567890
```

---

## inventory

### `inventory locations`
```bash
shopify-admin inventory locations
shopify-admin inventory locations --json
```

### `inventory items`
```bash
shopify-admin inventory items
shopify-admin inventory items --query "sku:MY-SKU"
```

### `inventory levels --location <id>`
Show available and on-hand quantities at a location.
```bash
shopify-admin inventory levels --location 1234567890
```

### `inventory adjust`
```bash
shopify-admin inventory adjust --item <inventory-item-id> --location <location-id> --delta 10
shopify-admin inventory adjust --item 123 --location 456 --delta -5 --reason damaged
```
Flags: `--item` (required), `--location` (required), `--delta` (positive=add, negative=subtract), `--reason`

---

## metafields

### `metafields list --owner <gid>`
```bash
shopify-admin metafields list --owner "gid://shopify/Product/1234567890"
shopify-admin metafields list --owner "gid://shopify/Order/1234567890"
shopify-admin metafields list --owner "gid://shopify/Customer/1234567890"
```

### `metafields get <id>`
```bash
shopify-admin metafields get 1234567890
```

### `metafields set`
Create or update a metafield.
```bash
shopify-admin metafields set --owner "gid://shopify/Product/123" \
  --namespace custom --key my_field --value "Hello" --type single_line_text_field
```
Common types: `single_line_text_field`, `multi_line_text_field`, `number_integer`, `number_decimal`, `boolean`, `date`, `date_time`, `json`, `color`, `url`, `rating`

### `metafields delete <id>`
```bash
shopify-admin metafields delete 1234567890
```

---

## metaobjects

### `metaobjects definitions`
List all metaobject types defined in the store.
```bash
shopify-admin metaobjects definitions
```

### `metaobjects list --type <type>`
```bash
shopify-admin metaobjects list --type my_custom_type
shopify-admin metaobjects list --type product_feature --first 20
```

### `metaobjects get <id>`
```bash
shopify-admin metaobjects get 1234567890
```

### `metaobjects create`
```bash
shopify-admin metaobjects create --type my_type --field title="Hello" --field description="World"
shopify-admin metaobjects create --type my_type --handle my-handle --field key=value
```

### `metaobjects update <id>`
```bash
shopify-admin metaobjects update 1234567890 --field title="New Title"
```

### `metaobjects delete <id>`
```bash
shopify-admin metaobjects delete 1234567890
```

---

## webhooks

### `webhooks list`
```bash
shopify-admin webhooks list
shopify-admin webhooks list --json
```

### `webhooks create`
```bash
shopify-admin webhooks create --topic ORDERS_CREATE --url https://example.com/webhooks/orders
shopify-admin webhooks create --topic PRODUCTS_UPDATE --url https://myapp.com/hook
```
Common topics: `ORDERS_CREATE`, `ORDERS_UPDATED`, `PRODUCTS_CREATE`, `PRODUCTS_UPDATE`, `CUSTOMERS_CREATE`, `INVENTORY_LEVELS_UPDATE`, `CHECKOUTS_CREATE`, `REFUNDS_CREATE`

### `webhooks delete <id>`
```bash
shopify-admin webhooks delete 1234567890
```

---

## discounts

### `discounts list`
Lists all discount types (code discounts and automatic discounts).
```bash
shopify-admin discounts list
shopify-admin discounts list --query "status:active"
```

### `discounts deactivate <id>`
```bash
shopify-admin discounts deactivate 1234567890
```

---

## analytics

### `analytics query <shopifyql>`
Run a ShopifyQL query. Requires `read_reports` access scope.
```bash
shopify-admin analytics query "FROM sales SHOW SUM(net_sales) SINCE -30d UNTIL today"
shopify-admin analytics query "FROM sales SHOW SUM(net_sales) GROUP BY month SINCE -6m UNTIL today"
shopify-admin analytics query "FROM sessions SHOW sessions GROUP BY device_type SINCE -7d UNTIL today"
shopify-admin analytics query "FROM products SHOW SUM(net_quantity) AS units ORDER BY units DESC SINCE -30d"
shopify-admin analytics query "FROM orders SHOW COUNT(order_id) GROUP BY day SINCE -7d" --json
```

---

## fulfillments

### `fulfillments list <order-id>`
```bash
shopify-admin fulfillments list 1234567890
```

### `fulfillments create <fulfillment-order-id>`
```bash
shopify-admin fulfillments create 1234567890
shopify-admin fulfillments create 1234567890 --tracking UPS --number 1Z999AA1234567890
shopify-admin fulfillments create 1234567890 --tracking FedEx --number 123456789 --url https://track.example.com
```

---

## markets

### `markets list`
```bash
shopify-admin markets list
```

### `markets get <id>`
```bash
shopify-admin markets get 1234567890
```

---

## ltv

Calculate the store's Lifetime Value (LTV).

**Formula:** `LTV = (net_sales − tax) ÷ unique paying customers`

**Default period:** last 3 years, excluding the last 3 months (gives recent data time to settle).

```bash
# Default: 3y period, excl. last 3 months
shopify-admin ltv

# Custom period
shopify-admin ltv --period 2y --exclude 0        # 2 years up to today
shopify-admin ltv --period 18m --exclude 1        # 18 months, excl. last month

# Explicit dates (override everything)
shopify-admin ltv --start 2022-01-01 --end 2024-01-01

# JSON output (for agents / further processing)
shopify-admin ltv --json
shopify-admin ltv --period 2y --json
```

**Flags:**
| Flag | Default | Description |
|------|---------|-------------|
| `--start YYYY-MM-DD` | computed | Start date — overrides `--period` |
| `--end YYYY-MM-DD` | today − exclude | End date |
| `--period <N>` | `3y` | Look-back: `Ny` years, `Nm` months, `Nd` days |
| `--exclude <N>` | `3` | Skip last N months from end (0 = up to today) |

**JSON output structure:**
```json
{
  "period": { "start": "2022-01-01", "end": "2025-01-01", "exclude_months": 3, "period": "3y" },
  "net_sales": 125432.50,
  "tax": 18214.80,
  "net_revenue": 107217.70,
  "customers": 1247,
  "ltv": 85.98
}
```

All monetary values are in the store's default currency.

---

## Tips

- **Before filtering**: Always run `shopify-admin search-syntax <resource> --json` to discover valid fields and values before constructing a `--query` string
- **Authentication**: Use `auth configure` + `auth login --no-browser` once; tokens auto-refresh forever. Avoid setting `SHOPIFY_ACCESS_TOKEN` env var as it bypasses auto-refresh
- **401 errors**: If you see `Invalid API key or access token`, check `auth status` — env vars may be overriding the config with a stale token. Unset them with `unset SHOPIFY_ACCESS_TOKEN SHOPIFY_SHOP`
- **Token type**: `shpc_` tokens are session tokens (not Admin API tokens) — the CLI detects this prefix and auto-refreshes via Client Credentials Grant on the next command
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output
- **Finding IDs**: Use `shopify-admin products list --json | jq '.[].id'`
- **Pagination**: Check for `(more results — use --after CURSOR)` at the bottom of list output
- **Combining filters (AND)**: Space-separate them: `--query "financial_status:paid fulfillment_status:unfulfilled"`
- **Date range**: Two filters: `--query "created_at:>2024-01-01 created_at:<2024-12-31"`
- **GIDs**: Shopify uses global IDs like `gid://shopify/Product/123`. You can pass either the full GID or just the numeric part to most commands
