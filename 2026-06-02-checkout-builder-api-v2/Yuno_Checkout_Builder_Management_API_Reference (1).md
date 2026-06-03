# Yuno Checkout Builder — Management API Reference

> **Scope:** Programmatic configuration of Yuno checkouts via Yuno's public B2B API.
>
> **Status:** BETA — endpoints are stable in shape but subject to additive changes during the BETA window. Open only to allowlisted merchants. Contact your Yuno TAM for access.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Base URL & Environments](#2-base-url--environments)
3. [Authentication & Headers](#3-authentication--headers)
4. [Endpoints at a Glance](#4-endpoints-at-a-glance)
5. [Resource Model](#5-resource-model)
   - [5.1 Checkout (GET response)](#51-checkout-get-response)
   - [5.2 PaymentMethod](#52-paymentmethod)
   - [5.3 ConditionSet](#53-conditionset)
   - [5.4 RequiredFieldsOverride](#54-requiredfieldsoverride)
6. [Enum Reference](#6-enum-reference)
7. [Examples](#7-examples)
8. [Important Notes & Gotchas](#8-important-notes--gotchas)
9. [Styling & SDK Settings](#9-styling--sdk-settings)
10. [JSON Schemas](#10-json-schemas)

---

## 1. Overview

The Checkout Builder Management API lets you read and update a merchant's checkout configuration:

- Which payment methods are shown and in what order
- Per-payment-method **conditions** (currency, country, amount, metadata, additional fields) that gate when a method appears
- Per-payment-method **required fields** (CVV, installments, first name, billing address, etc.) — including separate configurations for **enrolled** (saved/tokenized) vs **non-enrolled** flows
- General settings (country documents, etc.)
- **Styling & SDK settings** — colors, fonts, button shapes, dark mode, SDK render mode, payment-method-list layout, and Payment Link branding

All endpoints sit behind `https://api.y.uno/v1/checkouts/...` and are authenticated with merchant API keys (see [§3](#3-authentication--headers)).

---

## 2. Base URL & Environments

| Environment | Base URL |
| --- | --- |
| Production | `https://api.y.uno` |
| Sandbox    | `https://api-sandbox.y.uno` |

All examples below use the production base URL. The sandbox host accepts the same endpoints with the same payload shapes — confirm the exact sandbox URL with your Yuno TAM.

---

## 3. Authentication & Headers

Every request must include:

| Header | Required | Description |
| --- | --- | --- |
| `PUBLIC-API-KEY` | Yes | Merchant public API key. Issued via your Yuno dashboard → **Settings → API Keys**. |
| `PRIVATE-SECRET-KEY` | Yes | Merchant private secret key, paired with the public key above. Treat as a password — never embed in client-side code. |
| `X-Account-Code` | Yes | UUID of the merchant account scope for the request. |
| `X-Idempotency-Key` | Recommended (PATCH) | Client-generated UUID. Re-sending the same request with the same key is safe and will not double-publish. |
| `Content-Type` | Yes (PATCH) | `application/json` |
| `Accept` | Recommended | `application/json` |

> **Product permission required.** Credentials must have the `v1/checkouts` product permission enabled — `READ` for GET endpoints, `WRITE` for PATCH endpoints. Permissions are granted per-credential in the dashboard's **API Keys** page. Contact your Yuno TAM to enable this product for your account.

---

## 4. Endpoints at a Glance

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/checkouts/{checkout_code}` | Fetch the full checkout configuration (payment methods, conditions, settings). |
| `PATCH` | `/v1/checkouts/{checkout_code}/publish` | Publish a new configuration (payment methods + conditions + required fields + general settings). See [§8](#8-important-notes--gotchas) for upsert semantics. |
| `GET` | `/v1/checkouts/payment-methods/{payment_method_type}/required-fields?type={ENROLLMENT\|PAYMENT_METHOD}` | List the configurable required fields for a payment method (and whether for the enrollment flow or the standard flow). |
| `GET` | `/v1/checkouts/country-data` | Fetch supported countries and their document-type options (used to build `general_settings.country_documents`). |
| `GET` | `/v1/checkouts/builder/settings` | Fetch styling & SDK settings (colors, fonts, layout, dark mode, Payment Link branding, fonts catalog). See [§9](#9-styling--sdk-settings). |
| `PATCH` | `/v1/checkouts/builder/settings` | Update styling & SDK settings. See [§9](#9-styling--sdk-settings). |

The two endpoints you will use most for payment-method configuration are **`GET /v1/checkouts/{checkout_code}`** (read state) and **`PATCH /v1/checkouts/{checkout_code}/publish`** (write state). To toggle a payment method on/off, set `is_active` inside the `payment_methods` array of the publish payload — there is no separate is-active endpoint.

For visual / SDK-behavior configuration, use **`GET / PATCH /v1/checkouts/builder/settings`** ([§9](#9-styling--sdk-settings)).

---

## 5. Resource Model

### 5.1 Checkout (GET response)

```json
{
  "account_code": "b91b3970-dbf7-4d6a-b34d-96adbf3a0988",
  "code": "8005ca91-dcfb-4428-b9f6-49c6e3446474",
  "name": "...",
  "description": "...",
  "is_active": true,
  "is_archive": false,
  "organization_code": "...",
  "root": true,
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-05-18T13:06:08Z",
  "payment_methods": [ /* PaymentMethod[] */ ],
  "general_settings": {
    "country_documents": [
      { "country_code": "AR", "documents": ["DNI", "CUIT"] }
    ]
  },
  "feature_flags": {
    "custom_required_fields": true,
    "custom_logo_icon": false
  }
}
```

### 5.2 PaymentMethod

```json
{
  "is_active": true,
  "payment_method_type": "CARD",
  "order_to_show": 3,
  "type": "PAYMENT_METHOD",
  "active_enrollment_type": "BOTH",
  "conditions_to_override": [ /* ConditionSet[] (optional) */ ],
  "required_fields_to_override": { /* RequiredFieldsOverride (optional) */ }
}
```

**Field-by-field:**

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `is_active` | Yes | boolean | Whether the method is offered in the checkout. |
| `payment_method_type` | Yes | enum | See [§6.1](#61-payment_method_type). |
| `order_to_show` | Yes | int | **Display order in the checkout. Lower = shown first. The position in the JSON array is not what matters — this field is.** |
| `type` | Yes | enum | `PAYMENT_METHOD` (standard) or `ENROLLMENT` (tokenization/saved-credentials flow). See [§6.2](#62-type). |
| `active_enrollment_type` | When `type=PAYMENT_METHOD` | enum | For methods that support both enrolled and non-enrolled flows (e.g. CARD): `BOTH`, `NONE`, `ENROLLMENT`, `NO_ENROLLMENT`. See [§6.3](#63-active_enrollment_type). |
| `conditions_to_override` | No | array | Conditions that gate when the method appears (currency, country, amount, metadata, etc.). See [§5.3](#53-conditionset). |
| `required_fields_to_override` | No | object | Per-field configuration of which form fields are shown, for both the enrolled and non-enrolled flows. See [§5.4](#54-requiredfieldsoverride). |

### 5.3 ConditionSet

A ConditionSet is a named group of conditions. A payment method can have several sets — the method appears if **any** set matches (sets are OR'd; conditions within a set are AND'd).

```json
{
  "name": "Argentina small amounts",
  "order": 0,
  "is_active": true,
  "payment_method_data": { "logo": null, "name": null, "description": null },
  "conditions": [
    {
      "condition_type": "CURRENCY_AND_AMOUNT",
      "conditional": "ONE_OF",
      "values": ["ARS"],
      "complex_name": "CURRENCY",
      "complex_index": 0
    },
    {
      "condition_type": "CURRENCY_AND_AMOUNT",
      "conditional": "BETWEEN",
      "values": ["4.00", "6.00"],
      "complex_name": "AMOUNT",
      "complex_index": 0
    },
    {
      "condition_type": "METADATA",
      "conditional": "ONE_OF",
      "values": ["promo_summer"],
      "metadata_key": "campaign",
      "additional_field_name": "TEXT"
    },
    {
      "condition_type": "COUNTRY",
      "conditional": "NOT_ONE_OF",
      "values": ["AF", "AX", "AL", "DZ", "AS", "AD"]
    }
  ]
}
```

**Condition fields:**

| Field | When | Description |
| --- | --- | --- |
| `condition_type` | Always | See [§6.4](#64-condition_type). |
| `conditional` | Always | Operator. See [§6.5](#65-conditional). |
| `values` | Always | Array of string values. For `BETWEEN`, exactly two values (`[min, max]`). |
| `metadata_key` | When `condition_type = METADATA` | The metadata key in the payment request to compare against. |
| `additional_field_name` | When `condition_type = METADATA` | Data type of the metadata field (e.g. `TEXT`). |
| `complex_name` | When `condition_type = CURRENCY_AND_AMOUNT` | Sub-component being compared: `CURRENCY` or `AMOUNT`. |
| `complex_index` | When `condition_type = CURRENCY_AND_AMOUNT` | Pair index — both rows of the same currency+amount tuple share the same `complex_index`. |

### 5.4 RequiredFieldsOverride

```json
{
  "is_active": true,
  "is_enrollment_active": true,
  "fields": [
    {
      "current_value": null,
      "field_name": "installment",
      "is_active": true,
      "conditions_to_override": [ /* per-field condition sets */ ]
    }
  ],
  "enrollment_fields": [
    {
      "current_value": "FIRST_TIME",
      "field_name": "security_code",
      "is_active": true,
      "conditions_to_override": [ /* ... */ ]
    }
  ]
}
```

- `fields` — controls the **non-enrolled** flow (first-time payer entering full card details).
- `enrollment_fields` — controls the **enrolled** flow (returning payer using a saved/tokenized method).
- Each field can carry its own `conditions_to_override` to make the field requirement context-dependent (e.g. "only require installments in Argentina").

To know which `field_name`s a given payment method supports, call **`GET /checkouts/payment-methods/{type}/required-fields`** (see example below).

---

## 6. Enum Reference

### 6.1 payment_method_type

Common values today:

- `CARD`
- `GOOGLE_PAY`
- `APPLE_PAY`
- `PAYPAL`
- `PAYPAL_ENROLLMENT`
- `NU_PAY`
- `NU_PAY_ENROLLMENT`
- `WALLET`
- `MODO`
- `DLOCAL_CHECKOUT`
- `PUNTO_PAGO`

The complete, authoritative list lives in Yuno's payment-method catalog service. New methods are added over time — your client should treat unknown methods gracefully rather than fail validation.

### 6.2 type

- `PAYMENT_METHOD` — standard payment method (every transaction).
- `ENROLLMENT` — tokenization/saved-credentials flow shown only when the customer enrolls.

### 6.3 active_enrollment_type

For methods that support both flows (notably CARD):

- `BOTH` — both the enrolled and the non-enrolled flow are active.
- `NONE` — neither flow is active (effectively disabled even if `is_active=true`).
- `ENROLLMENT` — only the enrolled (saved-card) flow is shown.
- `NO_ENROLLMENT` — only the non-enrolled (first-time entry) flow is shown.

### 6.4 condition_type

- `CURRENCY` — match transaction currency.
- `CURRENCY_AND_AMOUNT` — match currency **and** amount range (uses `complex_name`/`complex_index`).
- `COUNTRY` — match country (ISO-3166-1 alpha-2).
- `METADATA` — match a value sent in the payment's metadata block (requires `metadata_key`).
- `ADDITIONAL_FIELD` — match a custom field collected at checkout.
- `CARD_BIN` — match card BIN ranges.
- `CARD_ISSUER` — match issuing bank.
- `TIME_PERIOD` — restrict to a date/time window.

### 6.5 conditional

- `ONE_OF` — value is in `values`.
- `NOT_ONE_OF` — value is not in `values`.
- `EQUAL`
- `NOT_EQUAL`
- `BETWEEN` — for numeric ranges (e.g. amount); `values` is `[min, max]`.
- `GREATER_THAN`
- `LESS_THAN`

### 6.6 field_name (required fields)

Common values observed for CARD:

- `security_code` — CVV. Supports `current_value: "FIRST_TIME"` to only require it on first use.
- `installment`
- `first_name`
- `last_name`
- `document`
- `email`
- `phone`
- `billing_address`
- `shipping_address`
- `zip_code`

The set varies by payment method; always discover dynamically via `GET /checkouts/payment-methods/{type}/required-fields`.

### 6.7 additional_field_name

- `TEXT` (additional values may exist — confirm with Yuno before relying on a new value).

---

## 7. Examples

All examples below show the **production** base URL. Replace `<YOUR_PUBLIC_KEY>` and `<YOUR_PRIVATE_SECRET>` with the credentials from your Yuno dashboard, and `<YOUR_IDEMPOTENCY_UUID>` with a client-generated UUID. Browser-only headers (`sec-ch-*`, `user-agent`, `referer`, etc.) are not required when calling from a backend.

### 7.1 GET — Fetch the full checkout configuration

```bash
curl 'https://api.y.uno/v1/checkouts/8005ca91-dcfb-4428-b9f6-49c6e3446474' \
  -H 'Accept: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988'
```

Response: a Checkout object as described in [§5.1](#51-checkout-get-response).

### 7.2 GET — Discover required fields for a payment method

Use this before building a `required_fields_to_override` payload, to learn which `field_name`s are valid for the method.

```bash
curl 'https://api.y.uno/v1/checkouts/payment-methods/CARD/required-fields?type=ENROLLMENT' \
  -H 'Accept: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988'
```

The `type` query param is `ENROLLMENT` for the enrolled flow or `PAYMENT_METHOD` for the non-enrolled flow.

### 7.3 PATCH — Disable a payment method (CARD), keep ordering

This disables CARD for **both** enrolled and non-enrolled flows by sending `is_active: false` and `active_enrollment_type: "NONE"`. Other methods are listed for clarity, but per [§8](#8-important-notes--gotchas), omitting a method from the array leaves its previous state untouched — you only need to include the methods you're actually changing.

```bash
curl 'https://api.y.uno/v1/checkouts/8005ca91-dcfb-4428-b9f6-49c6e3446474/publish' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{
    "payment_methods": [
      { "is_active": false, "payment_method_type": "NU_PAY_ENROLLMENT",   "order_to_show": 1,  "type": "ENROLLMENT" },
      { "is_active": true,  "payment_method_type": "PAYPAL_ENROLLMENT",   "order_to_show": 2,  "type": "ENROLLMENT" },
      { "is_active": false, "payment_method_type": "CARD",                "order_to_show": 3,  "type": "PAYMENT_METHOD", "active_enrollment_type": "NONE" },
      { "is_active": true,  "payment_method_type": "GOOGLE_PAY",          "order_to_show": 4,  "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "APPLE_PAY",           "order_to_show": 5,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PUNTO_PAGO",          "order_to_show": 6,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PAYPAL",              "order_to_show": 7,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "WALLET",              "order_to_show": 8,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "MODO",                "order_to_show": 9,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "DLOCAL_CHECKOUT",     "order_to_show": 10, "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "NU_PAY",              "order_to_show": 11, "type": "PAYMENT_METHOD" }
    ]
  }'
```

### 7.4 PATCH — Enable CARD with two condition sets (one active, one disabled)

This re-enables CARD and attaches two `conditions_to_override` sets. The first set (`ar-low-amount`) is active — CARD will only appear when the transaction matches **all** of its conditions (ARS, amount between 4 and 6, metadata `campaign=promo_summer`, and country not in the listed set). The second set (`ar-all-amounts-disabled`) is `is_active: false`, so it is saved but not enforced.

```bash
curl 'https://api.y.uno/v1/checkouts/8005ca91-dcfb-4428-b9f6-49c6e3446474/publish' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{
    "payment_methods": [
      { "is_active": false, "payment_method_type": "NU_PAY_ENROLLMENT",   "order_to_show": 1, "type": "ENROLLMENT" },
      { "is_active": true,  "payment_method_type": "PAYPAL_ENROLLMENT",   "order_to_show": 2, "type": "ENROLLMENT" },
      {
        "is_active": true,
        "payment_method_type": "CARD",
        "order_to_show": 3,
        "type": "PAYMENT_METHOD",
        "active_enrollment_type": "BOTH",
        "conditions_to_override": [
          {
            "name": "ar-low-amount",
            "order": 0,
            "is_active": true,
            "payment_method_data": { "logo": null, "name": null, "description": null },
            "conditions": [
              { "condition_type": "CURRENCY_AND_AMOUNT", "conditional": "ONE_OF",     "values": ["ARS"],          "complex_name": "CURRENCY", "complex_index": 0 },
              { "condition_type": "CURRENCY_AND_AMOUNT", "conditional": "BETWEEN",    "values": ["4.00","6.00"],  "complex_name": "AMOUNT",   "complex_index": 0 },
              { "condition_type": "METADATA",            "conditional": "ONE_OF",     "values": ["promo_summer"],         "metadata_key": "campaign",      "additional_field_name": "TEXT" },
              { "condition_type": "COUNTRY",             "conditional": "NOT_ONE_OF", "values": ["AF","AX","AL","DZ","AS","AD"] }
            ]
          },
          {
            "name": "ar-all-amounts-disabled",
            "order": 1,
            "is_active": false,
            "payment_method_data": { "logo": null, "name": null, "description": null },
            "conditions": [
              { "condition_type": "COUNTRY", "conditional": "ONE_OF", "values": ["AR"] }
            ]
          }
        ]
      },
      { "is_active": true,  "payment_method_type": "GOOGLE_PAY",          "order_to_show": 4,  "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "APPLE_PAY",           "order_to_show": 5,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PUNTO_PAGO",          "order_to_show": 6,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PAYPAL",              "order_to_show": 7,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "WALLET",              "order_to_show": 8,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "MODO",                "order_to_show": 9,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "DLOCAL_CHECKOUT",     "order_to_show": 10, "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "NU_PAY",              "order_to_show": 11, "type": "PAYMENT_METHOD" }
    ]
  }'
```

### 7.5 PATCH — Required-field conditions for the enrolled CARD flow

This example customizes two required fields **for the enrolled (saved-card) flow** (`enrollment_fields`):

- `security_code` — required only `FIRST_TIME`, **plus** an additional condition (apply if `skip_cvv=true` in metadata).
- `installment` — required, but only when currency is not AUD AND metadata `skip_cvv=true` AND country is AR.

All other fields are left active without overrides.

```bash
curl 'https://api.y.uno/v1/checkouts/8005ca91-dcfb-4428-b9f6-49c6e3446474/publish' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{
    "payment_methods": [
      { "is_active": false, "payment_method_type": "NU_PAY_ENROLLMENT",   "order_to_show": 1, "type": "ENROLLMENT" },
      { "is_active": true,  "payment_method_type": "PAYPAL_ENROLLMENT",   "order_to_show": 2, "type": "ENROLLMENT" },
      {
        "is_active": true,
        "payment_method_type": "CARD",
        "order_to_show": 3,
        "type": "PAYMENT_METHOD",
        "active_enrollment_type": "BOTH",
        "required_fields_to_override": {
          "is_active": true,
          "is_enrollment_active": true,
          "fields": [],
          "enrollment_fields": [
            {
              "current_value": "FIRST_TIME",
              "field_name": "security_code",
              "is_active": true,
              "conditions_to_override": [
                {
                  "name": "test-meta",
                  "is_active": true,
                  "code": "3f5e3bb3-dafd-4eaa-8e18-71df2d10bcb1",
                  "conditions": [
                    { "condition_type": "METADATA", "conditional": "ONE_OF", "condition_value": ["true"], "metadata_key": "skip_cvv", "additional_field_name": "TEXT" }
                  ]
                }
              ]
            },
            {
              "current_value": null,
              "field_name": "installment",
              "is_active": true,
              "conditions_to_override": [
                {
                  "name": "test",
                  "is_active": true,
                  "code": "c5f4ac5c-1b6c-45f8-9cfa-6986331cd036",
                  "conditions": [
                    { "condition_type": "CURRENCY", "conditional": "NOT_ONE_OF", "condition_value": ["AUD"] },
                    { "condition_type": "METADATA", "conditional": "ONE_OF",     "condition_value": ["true"], "metadata_key": "skip_cvv", "additional_field_name": "TEXT" },
                    { "condition_type": "COUNTRY",  "conditional": "ONE_OF",     "condition_value": ["AR"] }
                  ]
                }
              ]
            },
            { "current_value": null, "field_name": "first_name",       "is_active": true  },
            { "current_value": null, "field_name": "last_name",        "is_active": true  },
            { "current_value": null, "field_name": "document",         "is_active": true  },
            { "current_value": null, "field_name": "email",            "is_active": true  },
            { "current_value": null, "field_name": "phone",            "is_active": true  },
            { "current_value": null, "field_name": "billing_address",  "is_active": true  },
            { "current_value": null, "field_name": "shipping_address", "is_active": true  },
            { "current_value": null, "field_name": "zip_code",         "is_active": false }
          ]
        }
      },
      { "is_active": true,  "payment_method_type": "GOOGLE_PAY",          "order_to_show": 4,  "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "APPLE_PAY",           "order_to_show": 5,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PUNTO_PAGO",          "order_to_show": 6,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PAYPAL",              "order_to_show": 7,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "WALLET",              "order_to_show": 8,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "MODO",                "order_to_show": 9,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "DLOCAL_CHECKOUT",     "order_to_show": 10, "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "NU_PAY",              "order_to_show": 11, "type": "PAYMENT_METHOD" }
    ]
  }'
```

> **Note:** when a required field carries `conditions_to_override`, each per-field condition uses `condition_value` (singular) rather than `values`. Same concept as payment-method conditions, different field name — keep this distinction in mind. See [§8](#8-important-notes--gotchas).

### 7.6 PATCH — Required-field conditions for the non-enrolled CARD flow

Same shape as the previous example, but the override is placed in `fields` (non-enrolled flow) and the per-field conditions target `installment` and `first_name`. `enrollment_fields` is left empty so the enrolled flow is unchanged.

```bash
curl 'https://api.y.uno/v1/checkouts/8005ca91-dcfb-4428-b9f6-49c6e3446474/publish' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{
    "payment_methods": [
      { "is_active": false, "payment_method_type": "NU_PAY_ENROLLMENT",   "order_to_show": 1, "type": "ENROLLMENT" },
      { "is_active": true,  "payment_method_type": "PAYPAL_ENROLLMENT",   "order_to_show": 2, "type": "ENROLLMENT" },
      {
        "is_active": true,
        "payment_method_type": "CARD",
        "order_to_show": 3,
        "type": "PAYMENT_METHOD",
        "active_enrollment_type": "BOTH",
        "required_fields_to_override": {
          "is_active": true,
          "is_enrollment_active": true,
          "fields": [
            {
              "current_value": null,
              "field_name": "installment",
              "is_active": true,
              "conditions_to_override": [
                {
                  "name": "test",
                  "is_active": true,
                  "code": "52fde118-3a7f-40e5-b928-d7d7a04e76c4",
                  "conditions": [
                    { "condition_type": "CURRENCY", "conditional": "NOT_ONE_OF", "condition_value": ["AUD"] },
                    { "condition_type": "METADATA", "conditional": "ONE_OF",     "condition_value": ["true"], "metadata_key": "skip_cvv", "additional_field_name": "TEXT" },
                    { "condition_type": "COUNTRY",  "conditional": "ONE_OF",     "condition_value": ["AR"] }
                  ]
                }
              ]
            },
            {
              "current_value": null,
              "field_name": "first_name",
              "is_active": true,
              "conditions_to_override": [
                {
                  "name": "test",
                  "is_active": true,
                  "conditions": [
                    { "condition_type": "COUNTRY", "conditional": "NOT_ONE_OF", "condition_value": ["AR","BR"] }
                  ]
                }
              ]
            },
            { "current_value": null, "field_name": "last_name",        "is_active": true },
            { "current_value": null, "field_name": "document",         "is_active": true },
            { "current_value": null, "field_name": "email",            "is_active": true },
            { "current_value": null, "field_name": "phone",            "is_active": true },
            { "current_value": null, "field_name": "billing_address",  "is_active": true },
            { "current_value": null, "field_name": "shipping_address", "is_active": true },
            { "current_value": null, "field_name": "zip_code",         "is_active": true }
          ],
          "enrollment_fields": []
        }
      },
      { "is_active": true,  "payment_method_type": "GOOGLE_PAY",          "order_to_show": 4,  "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "APPLE_PAY",           "order_to_show": 5,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PUNTO_PAGO",          "order_to_show": 6,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "PAYPAL",              "order_to_show": 7,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "WALLET",              "order_to_show": 8,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "MODO",                "order_to_show": 9,  "type": "PAYMENT_METHOD" },
      { "is_active": false, "payment_method_type": "DLOCAL_CHECKOUT",     "order_to_show": 10, "type": "PAYMENT_METHOD" },
      { "is_active": true,  "payment_method_type": "NU_PAY",              "order_to_show": 11, "type": "PAYMENT_METHOD" }
    ]
  }'
```

---

## 8. Important Notes & Gotchas

1. **Payment-method order is driven by `order_to_show`, not by array position.** To re-order the checkout, change the integer values; the rendered checkout sorts by `order_to_show` ascending. We recommend sending the array sorted by `order_to_show` anyway for readability and stability.

2. **`PATCH /v1/checkouts/{checkout_code}/publish` is a sparse upsert, not a full replace.**
   - Payment methods omitted from `payment_methods[]` retain their previous state — they are **not** removed or disabled.
   - To disable a method, include it in the array with `is_active: false`.
   - New payment-method rows cannot be created via this endpoint; methods must already exist on the checkout. To enable additional methods, contact your Yuno TAM.
   - Including `conditions_to_override` on a method **replaces** that method's condition sets in full for that method. Omitting `conditions_to_override` leaves the existing conditions intact.
   - The same rule applies to `required_fields_to_override` and to the top-level `general_settings`: **omit → untouched, include → replace**.

3. > ⚠ **Field-name watch-out:** `conditions_to_override` at the **payment-method level** uses **`values`** (array) on each condition. `conditions_to_override` at the **required-field level** uses **`condition_value`** (also an array) on each condition. Same concept, different field name — preserved for backward compatibility. Don't mix them up.

4. **Enrolled vs non-enrolled flows are independent.** When configuring required fields for CARD, you almost always want to send both `fields` and `enrollment_fields`. To leave one flow untouched, pass an empty array for that side (as shown in [§7.5](#75-patch--required-field-conditions-for-the-enrolled-card-flow) / [§7.6](#76-patch--required-field-conditions-for-the-non-enrolled-card-flow)).

5. **`active_enrollment_type` only applies to `type: PAYMENT_METHOD`** entries that support enrollment (CARD is the main one today). For `type: ENROLLMENT` entries (e.g. `PAYPAL_ENROLLMENT`, `NU_PAY_ENROLLMENT`), omit `active_enrollment_type` and toggle `is_active` instead.

6. **Required-field conditions: only the first set is persisted.** Within `required_fields_to_override.fields[].conditions_to_override` (and the enrolled equivalent), **only the first element of the array is stored**. Send a single condition set per field. Additional sets are silently dropped.

7. **Live-traffic warning.** `PATCH /v1/checkouts/{checkout_code}/publish` takes effect immediately on the merchant's live checkout. Validate payloads against the sandbox base URL ([§2](#2-base-url--environments)) before applying to production.

---

## 9. Styling & SDK Settings

This section covers the visual / behavioral configuration for the checkout and Payment Link — colors, fonts, button shapes, SDK render mode, payment-method-list layout, dark mode, secure-payment tag, and Payment Link branding.

All operations use a **single endpoint** — `GET` and `PATCH` on `/v1/checkouts/builder/settings`. There is **no separate "publish" step** for styling; the PATCH itself is the publish.

### 9.1 Endpoint

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/v1/checkouts/builder/settings` | Fetch current styling, SDK settings, payment-link styling, fonts catalog, and flags. |
| `PATCH` | `/v1/checkouts/builder/settings` | Update any subset of styling / SDK settings. |

Same headers as the rest of the API (see [§3](#3-authentication--headers)).

### 9.2 Top-level request body

The PATCH body has **five top-level keys**, all optional. The API deep-merges what you send with the existing configuration — sending only the section you want to change is fine.

```json
{
  "styles":              { /* visual styles for the checkout SDK */ },
  "settings":            { /* SDK behavior + UI options */ },
  "payment_link_styles": { /* visual styles for hosted Payment Links */ },
  "flags":               { "force_default_styles": false },
  "external_fonts":      [ /* available font families & weights */ ]
}
```

> **Empty objects are no-ops.** Sending `"styles": {}` does **not** wipe styles — it preserves the existing values. To revert visuals to the built-in defaults, send `"flags": { "force_default_styles": true }` (see [§9.10](#910-edge-cases--behavior)).

### 9.3 styles — Checkout SDK visual styles

Three nested groups: `global`, `header`, `button`.

#### 9.3.1 styles.global

Affects every screen of the SDK.

| Field | Type | Default | Affects |
| --- | --- | --- | --- |
| `accent_color` | hex string | `#282A30` | Highlights, focus rings, active states, links. |
| `primary_background_color` | hex string | `#FFFFFF` | Main checkout surface background. |
| `primary_text_color` | hex string | `#282A30` | Body text, headings, input labels. |
| `primary_button_text_color` | hex string | `#FFFFFF` | Text inside the **Pay** button and other primary actions. |
| `secondary_background_color` | hex string | `#ECEFF2` | Cards, info panels, sub-sections (e.g. saved-card panel background). |
| `secondary_text_color` | hex string | `#84807F` | Captions, helper text, secondary labels. |
| `secondary_button_background_color` | hex string | `#FFFFFF` | Background of secondary buttons (Cancel, Back, "Use other method"). |
| `secondary_button_text_color` | hex string | `#282A30` | Text inside secondary buttons. |
| `font_family` | string | `"Inter"` | Font name. Must match a `family_name` in `external_fonts`. See [§9.7](#97-external_fonts--font-catalog). |

All colors are 6-character hex strings with a leading `#`. Lowercase or uppercase both work; the API normalizes.

#### 9.3.2 styles.header

Affects the SDK header band (where the merchant logo sits).

| Field | Type | Default | Affects |
| --- | --- | --- | --- |
| `logo_border_size` | number | `0` | Width (px) of the border drawn around the logo. |
| `logo_border_color` | hex | `#282A30` | Color of that border. |
| `logo_corner_radius` | number | `8` | Corner-radius (px) for the logo container. |
| `font_size` | number | `18` | Header title text size (px). |
| `font_weight` | number | `700` | Header title weight. Use multiples of 100 (100–900). |

#### 9.3.3 styles.button

Affects all SDK buttons.

| Field | Type | Default | Affects |
| --- | --- | --- | --- |
| `corner_radius` | number | `8` | Border-radius (px) for every button. |
| `border_size` | number | `0` | Border width (px) on every button. |
| `primary_border_color` | hex | `#282A30` | Border color for primary buttons. |
| `secondary_border_color` | hex | `#282A30` | Border color for secondary buttons. |
| `font_size` | number | `18` | Button text size (px). |
| `font_weight` | number | `400` | Button text weight. |

### 9.4 settings — SDK behavior & UI options

Several nested groups. Any group can be sent as `null` to leave it untouched, but sending an explicit object lets you change individual flags.

#### 9.4.1 settings.card

Card-payment-specific behavior.

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `credit_card_only_processing` | boolean | `false` | If true, debit cards are blocked — only credit-card transactions are accepted. |
| `default_network` | string | `"DOM"` | Preferred routing network for dual-network cards. Observed value: `"DOM"`. Confirm the full enum with Yuno before using values you haven't seen in your account. |
| `enable_ocr` | boolean | `false` | Enable in-SDK card-number OCR scan on mobile. Ignored on web. |
| `enable_payment_retry` | boolean \| null | `null` | Auto-retry behavior for failed transactions. Set to `true` to opt in, `null` to inherit BFF default. |
| `save_on_success` | boolean | `false` | If true, the SDK offers / performs save-card after a successful payment. |
| `visualization_mode` | enum | `"ONE_STEP"` | `"ONE_STEP"` (everything on a single screen) or `"STEP_BY_STEP"` (wizard). |

#### 9.4.2 settings.sdk_type

Top-level SDK rendering type — independent for web and mobile.

| Field | Type | Default | Values |
| --- | --- | --- | --- |
| `web` | enum | `"SEAMLESS"` | `"SEAMLESS"` (embedded in your page) or `"FULL"` (Yuno-hosted full-page checkout). |
| `mobile` | enum | `"SEAMLESS"` | `"SEAMLESS"` (embedded in your app) or `"FULL"` (Yuno-presented native checkout). |

#### 9.4.3 settings.web_sdk

Web-SDK-only options. Ignored on mobile.

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `render_mode` | enum | `"MODAL"` | `"MODAL"` opens checkout as an overlay; `"RENDER"` inlines it into a DOM container you control. |
| `hide_pay_button` | boolean | `true` | If true, the SDK does not render its own Pay button — your page must trigger payment via the SDK's `.pay()` method. |

#### 9.4.4 settings.payment_method_list

Controls the payment-method picker.

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `unfolded_display` | boolean | `true` | If true, the first payment method is rendered expanded (form visible) instead of collapsed behind a row. |
| `condensed_checkout_view` | boolean | `false` | Compact list layout (smaller logos, tighter spacing). Useful when you have many enabled methods. |
| `preselected_payment_method` | boolean | `false` | If true, the SDK auto-selects the first available method on load (fewer clicks for the customer). |
| `edit_payment_method_list` | boolean | `false` | If true, customers can manage (add/remove) saved payment methods directly from the picker. |
| `blik_unfolded_display` | boolean \| null | `null` | BLIK-specific variant of `unfolded_display`. Set true/false to override, null to inherit. |
| `moon_active_experience` | boolean \| null | `null` | Feature flag for a Moon-Active-specific UI variant. Leave null unless explicitly told to enable. |

#### 9.4.5 settings.payment_link

Behavior of the hosted Payment Link page.

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `show_result_screen` | boolean | `false` | If true, after payment the customer sees a Yuno-hosted result screen before any merchant-defined redirect. |

#### 9.4.6 settings.ui

Top-level UI toggles.

| Field | Type | Default | Affects |
| --- | --- | --- | --- |
| `dark_mode` | boolean | `false` | If true, the SDK uses its dark theme. **See [§9.10](#910-edge-cases--behavior) — when dark mode is on, custom `styles` are suppressed in favor of the dark palette.** |
| `show_secure_payment_tag` | boolean | `true` | Toggles the "Secured by Yuno" tag at the checkout footer. |

#### 9.4.7 Sub-objects accepted but not user-configurable from this UI

The following are part of the schema but currently have no UI controls in the Yuno dashboard. They're listed here for completeness — if you need to set them, confirm the exact shape with your Yuno TAM:

- `settings.click_to_pay` — Click-to-Pay (EMVCo) integration options.
- `settings.form` — Form-field collection / validation overrides.
- `settings.google_pay` — Google Pay-specific config (button style, merchant ID, etc.).
- `settings.urls` — Custom redirect URLs (success / failure / cancel).

Send these as `null` if you don't want to set them.

### 9.5 payment_link_styles — Hosted Payment Link branding

Independent from `styles` (which targets the SDK). Applies to the Yuno-hosted Payment Link page.

```json
{
  "panel": {
    "left": {
      "logo": null,
      "background": { "color": "#ECEFF2" }
    }
  },
  "checkbox":     { "color": "#282A30" },
  "border":       { "color": "#282A30" },
  "radio_button": { "color": "#282A30" },
  "button": {
    "background": { "color": "#282A30" },
    "text":       { "color": "#FFFFFF" }
  }
}
```

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `panel.left.logo` | string (URL or data URI) \| null | `null` | Merchant logo shown in the left side panel of the Payment Link. Accepts an HTTPS URL or a base64 `data:` URI. `null` = no logo. |
| `panel.left.background.color` | hex | `#ECEFF2` | Background color of the left panel. |
| `checkbox.color` | hex | `#282A30` | Color of checkboxes (active state). |
| `border.color` | hex | `#282A30` | Default border color for inputs and dividers in the Payment Link. |
| `radio_button.color` | hex | `#282A30` | Color of radio buttons (active state). |
| `button.background.color` | hex | `#282A30` | Background color of the primary action button. |
| `button.text.color` | hex | `#FFFFFF` | Text color of the primary action button. |

### 9.6 flags

| Field | Type | Default | Affects |
| --- | --- | --- | --- |
| `force_default_styles` | boolean | `false` | If true at the moment of PATCH, the merchant's persisted styles are **ignored** and the SDK renders with the built-in defaults instead. The styles you previously stored aren't lost — they're just bypassed until you set this back to false. |

### 9.7 external_fonts — Font catalog

Array describing which font families the SDK is allowed to load and where each weight file lives. The API seeds this with 18 curated families; you can add custom entries.

Shape:

```json
[
  {
    "family_name": "Inter",
    "files": [
      { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-Regular.ttf",  "weight": 400 },
      { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-Medium.ttf",   "weight": 500 },
      { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-SemiBold.ttf", "weight": 600 },
      { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-Bold.ttf",     "weight": 700 }
    ]
  }
]
```

- `family_name` — must match exactly when used as `styles.global.font_family`.
- `files[].weight` — CSS font-weight (100–900).
- `files[].url` — Yuno-hosted `.ttf` URL **or** any HTTPS URL you control (custom fonts).

**Canonical 18 families** (default catalog, all hosted under `https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/`):

> Cormorant Garamond, Dancing Script, Inter, Lato, Libre Baskerville, Manrope, Merriweather, Montserrat, Noto Sans, Nunito, Open Sans, Oswald, PT Sans, Quicksand, Raleway, Roboto, Source Code Pro *(17 listed; the 18th depends on the account — confirm with a GET)*.

Each family ships a subset of weights — see the **GET response** for the exact weight list per family.

**Custom fonts:** add a new entry to `external_fonts` with your own family name + URLs. After the PATCH, you can reference it by name via `styles.global.font_family`. Make sure your URLs are CORS-enabled and served over HTTPS, otherwise the SDK will silently fall back to the default font.

### 9.8 GET /v1/checkouts/builder/settings

```bash
curl 'https://api.y.uno/v1/checkouts/builder/settings' \
  -H 'Accept: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988'
```

Returns the current full configuration with every section described above. Use this as the source of truth before sending a PATCH.

### 9.9 PATCH examples

#### 9.9.1 Reset styling + SDK settings to defaults

Sends the full default `styles` palette, the default `settings` block, and the canonical `external_fonts` catalog. Equivalent to the "Restore defaults" button in the Styling tab of the Yuno dashboard.

```bash
curl 'https://api.y.uno/v1/checkouts/builder/settings' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{
    "styles": {
      "button": {
        "corner_radius": 8,
        "border_size": 0,
        "primary_border_color": "#282A30",
        "secondary_border_color": "#282A30",
        "font_size": 18,
        "font_weight": 400
      },
      "global": {
        "accent_color": "#282A30",
        "primary_background_color": "#FFFFFF",
        "primary_text_color": "#282A30",
        "primary_button_text_color": "#FFFFFF",
        "secondary_background_color": "#ECEFF2",
        "secondary_text_color": "#84807F",
        "secondary_button_background_color": "#FFFFFF",
        "secondary_button_text_color": "#282A30",
        "font_family": "Inter"
      },
      "header": {
        "logo_border_size": 0,
        "logo_border_color": "#282A30",
        "logo_corner_radius": 8,
        "font_size": 18,
        "font_weight": 700
      }
    },
    "settings": {
      "card": {
        "credit_card_only_processing": false,
        "default_network": "DOM",
        "enable_ocr": false,
        "enable_payment_retry": null,
        "save_on_success": false,
        "visualization_mode": "ONE_STEP"
      },
      "click_to_pay": null,
      "form": null,
      "google_pay": null,
      "payment_link": { "show_result_screen": false },
      "payment_method_list": {
        "blik_unfolded_display": null,
        "condensed_checkout_view": false,
        "edit_payment_method_list": false,
        "moon_active_experience": null,
        "preselected_payment_method": false,
        "unfolded_display": true
      },
      "sdk_type": { "mobile": "SEAMLESS", "web": "SEAMLESS" },
      "ui": { "dark_mode": false, "show_secure_payment_tag": true },
      "urls": null,
      "web_sdk": { "hide_pay_button": true, "render_mode": "MODAL" }
    },
    "flags": { "force_default_styles": false },
    "external_fonts": [
      { "family_name": "Cormorant Garamond", "files": [
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/CormorantGaramond-Regular.ttf",  "weight": 400 },
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/CormorantGaramond-Medium.ttf",   "weight": 500 },
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/CormorantGaramond-SemiBold.ttf", "weight": 600 },
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/CormorantGaramond-Bold.ttf",     "weight": 700 }
      ]},
      { "family_name": "Inter", "files": [
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-Regular.ttf",  "weight": 400 },
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-Medium.ttf",   "weight": 500 },
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-SemiBold.ttf", "weight": 600 },
        { "url": "https://prod.y.uno/sdk-static-bundles-ms/v1/static/fonts/Inter-Bold.ttf",     "weight": 700 }
      ]}
      /* …repeat for every family you want available; omit any family you want to disable. */
    ]
  }'
```

> The full canonical font catalog (all 18 families with every weight) is too long to inline. Pull it from a GET once and reuse the exact array.

#### 9.9.2 Reset Payment Link styling to defaults

The "Restore Payment Link defaults" action sends only the `payment_link_styles` block, with empty `styles` and `settings` (no-ops, see [§9.2](#92-top-level-request-body)).

```bash
curl 'https://api.y.uno/v1/checkouts/builder/settings' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{
    "styles": {},
    "settings": {},
    "payment_link_styles": {
      "panel": {
        "left": {
          "logo": null,
          "background": { "color": "#ECEFF2" }
        }
      },
      "checkbox":     { "color": "#282A30" },
      "border":       { "color": "#282A30" },
      "radio_button": { "color": "#282A30" },
      "button": {
        "background": { "color": "#282A30" },
        "text":       { "color": "#FFFFFF" }
      }
    }
  }'
```

#### 9.9.3 Minimal PATCH — change one thing at a time

You don't need to send the full document. The API deep-merges. To turn dark mode on and nothing else:

```bash
curl 'https://api.y.uno/v1/checkouts/builder/settings' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{ "settings": { "ui": { "dark_mode": true } } }'
```

To change only the accent color:

```bash
curl 'https://api.y.uno/v1/checkouts/builder/settings' \
  -X 'PATCH' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'PUBLIC-API-KEY: <YOUR_PUBLIC_KEY>' \
  -H 'PRIVATE-SECRET-KEY: <YOUR_PRIVATE_SECRET>' \
  -H 'X-Idempotency-Key: <YOUR_IDEMPOTENCY_UUID>' \
  -H 'X-Account-Code: b91b3970-dbf7-4d6a-b34d-96adbf3a0988' \
  --data-raw '{ "styles": { "global": { "accent_color": "#1A73E8" } } }'
```

### 9.10 Edge cases & behavior

1. **Dark mode suppresses custom styles.** When `settings.ui.dark_mode` is true, the SDK applies its built-in dark palette. The Yuno dashboard disables the entire **Form Styles** tab while dark mode is active and shows a warning. The custom styles you've stored are not deleted — they're just bypassed until dark mode is turned off again. If you want both a custom palette **and** dark mode, you'll need two different account configurations.

2. **`flags.force_default_styles: true` is a kill-switch.** When this flag is true, the SDK ignores everything under `styles` and renders with the built-in light defaults. Useful as a one-line rollback while debugging a faulty palette. Set back to false to restore your custom styling.

3. **`{}` is a no-op, `null` clears.** Sending an empty object `{}` for any section means "don't touch this section" (deep-merge with nothing). Sending `null` for a sub-field that accepts null (e.g. `settings.card.enable_payment_retry`) explicitly clears the override. Sending `null` for an entire section that doesn't accept null may be rejected — when in doubt, send `{}`.

4. **Font picker depends on `external_fonts`.** Setting `styles.global.font_family` to a name that isn't present in `external_fonts` will silently fall back to the default. Always keep your desired family in the catalog.

5. **Custom fonts must be CORS-enabled.** Browsers refuse to load fonts from origins that don't return `Access-Control-Allow-Origin`. If your custom font URL doesn't satisfy this, the SDK silently uses the default — there's no visible error.

6. **`web_sdk.hide_pay_button: true` requires SDK integration changes.** When the built-in Pay button is hidden, your page must call the SDK's pay method itself (e.g. via the SDK's exposed `.pay()` API). Flipping this on without updating the host page leaves customers with no way to trigger payment.

7. **`web_sdk.render_mode: "RENDER"` requires a DOM container.** The SDK will look for the mount node you pass to its `init()` call. `"MODAL"` does not require one — it overlays on top of the page. If you switch from MODAL to RENDER without giving the SDK a container, nothing displays.

8. **`visualization_mode: "STEP_BY_STEP"` changes the customer journey.** Switching between `"ONE_STEP"` and `"STEP_BY_STEP"` is safe but visible to in-flight customers — they may see a different layout mid-session. Consider deploying outside peak hours.

9. **`sdk_type: "FULL"` versus `"SEAMLESS"`.** `"SEAMLESS"` is the embedded experience inside your own page/app. `"FULL"` redirects (web) or presents (mobile) the Yuno-hosted full-page checkout. Switching from SEAMLESS to FULL may require URL allow-listing on the merchant account — coordinate with Yuno before flipping.

10. **PATCH takes effect immediately.** There is no "publish" step. The next customer hitting your checkout will see the new styles / settings as soon as the PATCH returns `200`. Validate against the sandbox base URL ([§2](#2-base-url--environments)) before applying to production.

11. **Payment Link styling is separate.** `payment_link_styles` does **not** inherit from `styles`. If you brand both surfaces, update both blocks. The Payment Link logo accepts either an HTTPS URL (preferred) or a base64 `data:` URI; the URL form keeps the JSON payload small and cacheable.

12. **`show_secure_payment_tag: false` removes the "Secured by Yuno" footer tag.** This is allowed but discouraged — the tag reassures shoppers and improves conversion. Confirm with Yuno before turning it off if you're on a regulated vertical.

---

## 10. JSON Schemas

This section provides machine-readable [JSON Schema](https://json-schema.org/understanding-json-schema/about) (draft 2020-12) definitions for the request bodies of the three main endpoints. Use them for:

- Generating client SDKs / typed models in your language of choice.
- Validating payloads in CI before sending to the API.
- Documenting allowed values / enums to your internal consumers.

Schemas are **descriptive**, not exhaustive — they encode every field the API currently produces or accepts. The API may accept additional fields not listed here (especially under `settings.click_to_pay`, `settings.form`, `settings.google_pay`, `settings.urls`); the schemas leave those open by setting `additionalProperties: true` where appropriate.

Three schemas:

- **§10.1** — `PATCH /checkouts/publish` body (payment methods + conditions + required-field overrides + optional general settings).
- **§10.2** — `PATCH /v1/checkouts/builder/settings` body (styling + SDK behavior + payment-link styles + flags + fonts).
- **§10.3** — `general_settings` block standalone (a sub-object of §10.1, repeated there so it can be consumed independently).

> ⚠ **TODO — companion file in preparation.** The full machine-readable JSON Schemas will be published as a companion reference document (`Yuno_Checkout_Builder_JSON_Schema.md`) in the next BETA revision. Until then, treat the [§5](#5-resource-model) resource model and the [§9](#9-styling--sdk-settings) settings tables as authoritative.
