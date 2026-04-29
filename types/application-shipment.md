# `application/shipment`

Sidekick intent type for composing or editing a **shipment** — a package being sent for an order (outbound to customer or inbound for a return), with carrier, service level, weight/dimensions, and tracking.

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/shipment.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/shipment.json)

## When to register an intent for this type

Register an `application/shipment` intent when your app can take a partial shipment spec — order, line items, carrier, service — and either (a) create a new shipment / label or (b) modify an unfulfilled one.

Typical examples:
- A **shipping app** registering `create` for "create a label for order #1234 — overnight, signature required."
- A **3PL integration** registering `edit` for "update the package weight on the shipment in queue, the customer changed quantities."
- A **returns app** registering `create` for "generate a prepaid return label for RMA #567" (a return shipment).

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "create-shipment"
handle = "create-shipment"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/CreateShipment.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/shipment"
  action = "create"
  schema = "./shipment-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required. `edit` is realistic only before the label is generated — most carriers don't allow post-label changes.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./shipment-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/shipment.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The shipment ID. Used for `edit` actions. |

`additionalProperties: true`. Schema is a stub at this stage.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `order_id` — the order being shipped
- `line_items` — array of `{ line_item_id, quantity }`
- `direction` — `"outbound" | "return"`
- `carrier` — `"ups" | "usps" | "fedex" | "dhl" | "canada_post" | ...`
- `service_level` — `"ground" | "priority" | "express" | "overnight"`
- `weight` — `{ value, unit }` (`unit`: `"oz" | "lb" | "g" | "kg"`)
- `dimensions` — `{ length, width, height, unit }`
- `ship_from` — address object
- `ship_to` — address object
- `tracking_number` — populated post-label
- `label_url` — URI to the printable label
- `signature_required` — boolean
- `insurance` — `{ amount, currency }`

## Common pitfalls

- **Editing a label after print.** Most carriers void rather than amend. Your UI should detect "label already generated" and switch from edit-in-place to a void+recreate flow.
- **Mixing weight units.** Some carriers default to imperial, some metric. Always include the unit; never assume.
- **Treating return shipments as a separate type.** They're not — `direction: "return"` keeps the schema unified. The associated [`application/return`](./application-return.md) is the *RMA*; the `application/shipment` is the label.

## Related types

- **[`application/return`](./application-return.md)** — the return / RMA that a return shipment is attached to.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
