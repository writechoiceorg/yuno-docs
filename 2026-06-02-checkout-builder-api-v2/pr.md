# PR Details

## Branch

`checkout-builder-api-v2`

## Title

`docs(checkout-builder): migrate API reference to public B2B API (v2)`

## Description

Updates the hidden Checkout Builder API reference to reflect revised specifications. The original pages documented the internal Dashboard BFF API; this PR migrates them to Yuno's **public B2B API**.

### What changed

**Authentication**
- Removed: JWT Bearer + `x-environment` + `x-organization-code`
- Added: `PUBLIC-API-KEY`, `PRIVATE-SECRET-KEY`, `X-Account-Code`, `X-Idempotency-Key`

**Base URL**
- Removed: `https://prod.y.uno/dashboard-bff/api`
- Added: `https://api.y.uno` (production), `https://api-sandbox.y.uno` (sandbox)

**Endpoint paths**
- `GET /checkouts/{checkoutCode}` → `GET /v1/checkouts/{checkout_code}`
- `PATCH /checkouts/publish` → `PATCH /v1/checkouts/{checkout_code}/publish`
- `GET /checkouts/payment-methods/{paymentMethodType}/required-fields` → `GET /v1/checkouts/payment-methods/{payment_method_type}/required-fields`
- `GET /checkouts/country-data` → `GET /v1/checkouts/country-data`
- Styling endpoints: server URL + auth only (paths were already `/v1/...`)

**Publish semantics**
- Changed from full replace to **sparse upsert** — omitted methods retain their state

**Overview page**
- Added BETA warning callout
- Updated base URLs, authentication table, key behaviors

### Files modified

- `reference/checkout-builder/overview.mdx`
- `reference/checkout-builder/fetch-checkout-configuration.mdx`
- `reference/checkout-builder/publish-checkout-configuration.mdx`
- `reference/checkout-builder/get-required-fields.mdx`
- `reference/checkout-builder/get-country-data.mdx`
- `openapi/checkout-builder/fetch-checkout-configuration.json`
- `openapi/checkout-builder/publish-checkout-configuration.json`
- `openapi/checkout-builder/get-required-fields.json`
- `openapi/checkout-builder/get-country-data.json`
- `openapi/checkout-builder/fetch-styling-settings.json`
- `openapi/checkout-builder/update-styling-settings.json`

All pages remain `hidden: true`.
