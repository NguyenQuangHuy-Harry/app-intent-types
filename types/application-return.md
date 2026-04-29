# `application/return`

Sidekick intent type for composing or editing a **return** (RMA) — a request to take back items from an order, optionally including a refund, restocking fee, and return shipping arrangement.

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/return.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/return.json)

## When to register an intent for this type

Register an `application/return` intent when your app can take a partial return spec — order, line items, reason — and either (a) start a new return or (b) modify an in-progress one.

Typical examples:
- A **returns management app** registering `create` for "start a return on order #1234 — customer says one of the shirts arrived damaged."
- A **returns app** registering `edit` for "add the second pair of jeans to RMA #567 and waive the restocking fee."
- A **support app with returns capability** registering `create` for "process the return the customer just asked about in this email thread."

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "process-return"
handle = "process-return"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/ProcessReturn.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/return"
  action = "create"
  schema = "./return-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./return-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/return.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The return ID. Used for `edit` actions. |

`additionalProperties: true`. Schema is a stub at this stage.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `order_id` — the order being returned against
- `line_items` — array of `{ line_item_id, quantity, reason_code }`
- `reason_code` — standardized return reason (`"damaged" | "wrong_item" | "size" | "no_longer_wanted" | "other"`)
- `return_method` — `"mail" | "in_store" | "carrier_pickup"`
- `restocking_fee` — `{ amount, currency }`
- `refund` — `{ amount, currency, method }` (`method`: `"original" | "store_credit" | "exchange"`)
- `return_label` — reference to a shipping label (a related [`application/shipment`](./application-shipment.md))

## Common pitfalls

- **Treating the intent as authorization to refund.** It isn't. The intent passes through your UI; the merchant still has to approve the refund. Don't auto-refund on intent receipt.
- **Standardizing reason codes locally.** If your app has its own taxonomy, map *to* the standardized codes when emitting; otherwise other Sidekick-aware apps can't reason across return data.
- **Skipping the return shipping flow.** A return without a shipping arrangement is half a return. Either embed the label-creation step in your UI or hand off to an [`application/shipment`](./application-shipment.md) intent at the right moment.

## Related types

- **[`application/shipment`](./application-shipment.md)** — for the return shipping label specifically.
- **[`application/ticket`](./application-ticket.md)** — when the return originates from a support thread.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
