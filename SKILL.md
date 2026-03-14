---
name: ziptax
description: >
  Look up US and Canadian sales tax rates using the ZipTax MCP server.
  Use this skill whenever the user asks about sales tax rates, tax lookups,
  tax jurisdictions, use tax, sales tax compliance, tax rate by ZIP code,
  tax rate by address, tax rate by coordinates, Canadian sales tax,
  product taxability, tax codes (TIC), Tennessee Single Article Tax,
  historical tax rates, unincorporated area adjustments, or account
  usage metrics for ZipTax. Also use when the user mentions ZipTax,
  zip-tax, or zip.tax. This skill teaches Claude how to use the
  lookup_tax_rate and get_account_metrics MCP tools effectively.
---

# ZipTax Sales Tax Lookup

This skill provides guidance for using the ZipTax MCP server to look up US and Canadian sales tax rates. The MCP server exposes two tools: `lookup_tax_rate` and `get_account_metrics`.

## Prerequisites

- A ZipTax API key from https://platform.zip.tax
- The ZipTax MCP server connected at `https://mcp.zip-tax.com`

If the user hasn't connected the ZipTax MCP server yet, guide them to:

1. Get an API key at https://platform.zip.tax
2. Connect the MCP server using one of these authentication methods:
   - **Header auth**: Set URL to `https://mcp.zip-tax.com/` and add their API key in the `X-API-KEY` header
   - **URL parameter auth**: Set URL to `https://mcp.zip-tax.com/?key=YOUR_API_KEY` with no additional headers

## Available Tools

### lookup_tax_rate

Looks up sales and use tax rates for a US or Canadian location. Returns jurisdiction-level tax breakdowns (state, county, city, district) plus service taxability, freight taxability, and sourcing rules.

#### Choosing the Right Parameters

Use the most specific input available for the best results:

- **Street address** (`address` + `city` + `state` + `postalcode`): Most precise. Returns door-level rates including city and district taxes. Requires a geo-enabled account.
- **ZIP code only** (`postalcode`): Fast and simple. Sufficient when city/district-level precision is not needed.
- **Coordinates** (`lat` + `lng`): Useful for mobile or mapping applications. Requires a geo-enabled account.

Always include `postalcode` when possible, even alongside `address`, for the most reliable results.

#### Parameters

| Parameter | Description | When to Use |
|---|---|---|
| `postalcode` | US 5-digit ZIP or Canadian postal code | Always include when available |
| `address` | Full street address | When precise door-level rates are needed |
| `state` | Two-letter state/province code (e.g., `CA`, `ON`) | To disambiguate or filter results |
| `city` | City name | To disambiguate within a ZIP code |
| `county` | County name | Rarely needed; helps with unincorporated areas |
| `country_code` | `US` (default) or `CA` | Set to `CA` for Canadian lookups |
| `lat` / `lng` | Latitude and longitude | For coordinate-based lookups |
| `historical` | Period in `YYYYMM` format | For past tax rates (e.g., `202312`) |
| `adjustment` | Set to `auto` | For unincorporated area adjustments |
| `taxability_code` | Product taxability code (TIC) | For product-specific tax rules |
| `sat_item_total` | Item total amount | For Tennessee Single Article Tax |
| `format` | `json` (default) or `xml` | Almost always leave as default |

#### Interpreting the Response

The response contains several sections:

- **`baseRates`**: Array of individual tax rates by jurisdiction and type. Each entry has:
  - `rate`: Decimal rate (e.g., `0.0725` = 7.25%)
  - `jurType`: Jurisdiction type (e.g., `US_STATE_SALES_TAX`, `US_COUNTY_USE_TAX`, `US_CITY_SALES_TAX`)
  - `jurName`: Jurisdiction name (e.g., `CA`, `ORANGE`, `IRVINE`)

- **`taxSummaries`**: Aggregated totals. The `rate` field in each summary is the combined rate across all jurisdictions for that tax type (`SALES_TAX` or `USE_TAX`). Present these totals to the user as the headline rate.

- **`service`**: Whether services are taxable in this jurisdiction. `taxable: "N"` means services are non-taxable.

- **`shipping`**: Whether freight/shipping is taxable. `taxable: "N"` means freight is non-taxable.

- **`sourcingRules`**: Whether the jurisdiction uses origin-based or destination-based taxation. `value: "D"` = destination-based.

- **`addressDetail`**: The normalized address, incorporation status, and geocoordinates.

#### Presenting Results to the User

When displaying tax rate results:

1. **Lead with the total combined rate** from `taxSummaries` (e.g., "The total sales tax rate is 7.75%").
2. **Break down by jurisdiction** using `baseRates` — show state, county, city, and any district rates as a table.
3. **Include both sales and use tax** if they differ.
4. **Note service and freight taxability** when relevant to the user's question.
5. **Mention the normalized address** from `addressDetail` so the user can confirm the location was resolved correctly.

Example format:

```
Tax rates for 200 Spectrum Center Dr, Irvine, CA 92618:

| Jurisdiction | Sales Tax | Use Tax |
|---|---|---|
| CA (State) | 7.25% | 7.25% |
| Orange (County) | 0.50% | 0.50% |
| Irvine (City) | 0.00% | 0.00% |
| **Total** | **7.75%** | **7.75%** |

- Services: Non-taxable
- Freight/Shipping: Non-taxable
- Sourcing: Destination-based
```

### get_account_metrics

Returns account usage metrics and quota information. Takes no parameters — authentication is handled via the API key header.

Use this tool when the user asks about their API usage, remaining quota, account status, or rate limits.

## Common Workflows

### Basic Tax Lookup by Address

When the user provides a street address:

1. Parse the address into components (street, city, state, ZIP)
2. Call `lookup_tax_rate` with `address`, `city`, `state`, and `postalcode`
3. Present the total rate and jurisdiction breakdown

### Basic Tax Lookup by ZIP Code

When the user provides only a ZIP code:

1. Call `lookup_tax_rate` with `postalcode`
2. Note that results may cover a broader area than a single city
3. Present the total rate and jurisdiction breakdown

### Canadian Tax Lookup

When the user asks about Canadian sales tax:

1. Call `lookup_tax_rate` with `postalcode` and `country_code` set to `CA`
2. Present the results, noting that Canadian jurisdictions use GST/HST/PST structures

### Historical Tax Rate Lookup

When the user wants tax rates from a past period:

1. Call `lookup_tax_rate` with the location parameters plus `historical` set to the `YYYYMM` period
2. Clearly label the results as historical and state the period

### Product-Specific Tax Lookup

When the user asks about taxability for a specific product category:

1. Call `lookup_tax_rate` with the location parameters plus the appropriate `taxability_code`
2. Explain that product taxability codes (TICs) vary by jurisdiction and product type

## Error Handling

| Error Message | Meaning | Action |
|---|---|---|
| "Missing API key" | No API key provided via header or URL parameter | Ask the user to verify their MCP client configuration includes the API key (via `X-API-KEY` header, `Authorization` header, or `?key=` URL parameter) |
| "At least one of postalcode, address, or lat/lng is required" | No location was provided | Ask the user for a location (ZIP code, address, or coordinates) |
| "ZipTax API error" | The underlying API returned an error | Check the error details — could be an invalid key, expired account, or rate limit. Direct the user to https://platform.zip.tax |

## Additional Resources

- ZipTax API documentation: https://developers.zip.tax
- Get an API key: https://platform.zip.tax
- ZipTax MCP server repository: https://github.com/ZipTax/ziptax-mcp
