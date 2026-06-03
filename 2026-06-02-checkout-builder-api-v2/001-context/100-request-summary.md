# Request Summary

## What changed

Aleksandra sent a revised Checkout Builder Management API reference document (`Yuno_Checkout_Builder_Management_API_Reference (1).md`). The previous version was scoped to the internal Dashboard BFF API; the new version targets Yuno's **public B2B API**.

## Key differences from v1 → v2

| | Old (Dashboard BFF) | New (Public B2B) |
|---|---|---|
| Scope | Internal dashboard API | Public B2B API |
| Status | No BETA note | **BETA** — allowlisted merchants only |
| Base URL | `https://prod.y.uno/dashboard-bff/api` | `https://api.y.uno` |
| Sandbox URL | `https://staging.y.uno/dashboard-bff/api` | `https://api-sandbox.y.uno` |
| Auth | JWT Bearer + x-account-code + x-environment + x-organization-code | `PUBLIC-API-KEY` + `PRIVATE-SECRET-KEY` + `X-Account-Code` |
| Fetch path | `/checkouts/{checkoutCode}` | `/v1/checkouts/{checkout_code}` |
| Publish path | `/checkouts/publish` (body param) | `/v1/checkouts/{checkout_code}/publish` (path param) |
| Publish body field | `paymentMethods` (camelCase) | `payment_methods` (snake_case) |
| Publish semantics | Full replace | **Sparse upsert** |
| Required-fields path | `/checkouts/payment-methods/{paymentMethodType}/...` | `/v1/checkouts/payment-methods/{payment_method_type}/...` |
| Country-data path | `/checkouts/country-data` | `/v1/checkouts/country-data` |
| Styling paths | `/v1/checkouts/builder/settings` | `/v1/checkouts/builder/settings` (unchanged) |

## Files to update

- `reference/checkout-builder/overview.mdx` — full rewrite
- `reference/checkout-builder/fetch-checkout-configuration.mdx` — openapi path ref
- `reference/checkout-builder/publish-checkout-configuration.mdx` — openapi path ref
- `reference/checkout-builder/get-required-fields.mdx` — openapi path ref
- `reference/checkout-builder/get-country-data.mdx` — openapi path ref
- `openapi/checkout-builder/fetch-checkout-configuration.json` — server + auth + path
- `openapi/checkout-builder/publish-checkout-configuration.json` — server + auth + path + body
- `openapi/checkout-builder/get-required-fields.json` — server + auth + path
- `openapi/checkout-builder/get-country-data.json` — server + auth + path
- `openapi/checkout-builder/fetch-styling-settings.json` — server + auth only
- `openapi/checkout-builder/update-styling-settings.json` — server + auth only
