# `shopify/order`

Sidekick intent type for importing one or more **Shopify orders** into an app-provided workflow and returning Shopify Order GIDs.

- **Status:** ✅ Supported
- **Actions:** `import`, `import+bulk`
- **GID schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/order/gid.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/order/gid.json)

## When to register an intent for this type

Register a `shopify/order` intent when your app operates on an existing Shopify Order identified by a GID and produces a Shopify Order result.

Typical examples:
- A **fulfillment app** registering `import` so a merchant can import an order into an app-specific fulfillment workflow.
- A **returns or support app** registering `import` so a merchant can start from a Shopify order and continue in the app's order workflow.
- An **operations app** registering `import+bulk` so a merchant can process multiple orders at once.

Use `shopify/order` when the source and output are Shopify Order resources. If your app owns the artifact shape instead, use an `application/*` type.

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2026-04"

[[extensions]]
name = "Import order"
description = "Take an existing Shopify order and import it into an app workflow"
handle = "import-order"
type = "admin_link"

  [[extensions.targeting]]
  target = "admin.app.intent.link"
  url = "/orders/{id}/import"
  tools = "./tools.json"
  instructions = "./instructions.md"

  [[extensions.targeting.intents]]
  type = "shopify/order"
  action = "import"
  schema = "./order-import-schema.json"
```

Three things to notice:

1. **`type`** is the Shopify resource type from this catalog. Required.
2. **`action`** is `import` for one resource or `import+bulk` for many resources. Required.
3. **`schema`** points to a local JSON Schema file. Required. Use `shopify-intent.json` for `import` and `shopify-intent-bulk.json` for `import+bulk`.

## Example: the import schema

`./order-import-schema.json`:

```json
{
  "$schema": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify-intent.json",
  "value": {
    "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/order/gid.json",
    "mapTo": "param",
    "fieldName": "id"
  },
  "inputSchema": {
    "type": "object",
    "properties": {
      "instructions": {
        "type": "string",
        "description": "Merchant-provided instructions for the order workflow."
      }
    }
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "id": {
        "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/order/gid.json"
      }
    }
  }
}
```

## Fields the schema describes

A Shopify resource intent schema has a fixed shape:

| Field | Type | Notes |
|---|---|---|
| `value` | Shopify Order GID | Source order passed to the app. Uses the `shopify/order` GID schema. |
| `inputSchema` | object | App-specific input fields. Do not declare `required` fields. |
| `outputSchema.properties.id` | Shopify Order GID | Result order returned by `shopify.intents.response.ok`. |

For `import+bulk`, `value` is an array of Order GIDs and the output should return an array under `outputSchema.properties.ids`.

## Common pitfalls

- **Using `create` or `edit` with `shopify/order`.** Those verbs are reserved for Shopify-native Admin intents. Third-party app extensions use `import` or `import+bulk`.
- **Using the wrong meta-schema.** `import` requires `shopify-intent.json`; `import+bulk` requires `shopify-intent-bulk.json`.
- **Pointing at an application schema.** `value.$ref` must point to the Order GID schema, not an `application/*` schema.
- **Treating the URL `{id}` as a full GID.** When mapped as a `param`, Sidekick passes the bare tail ID into the URL path. Read the full GID from the intent payload if you need it.

## Related types

- **[`shopify/customer`](./shopify-customer.md)** — for importing Shopify Customer resources.
- **[`shopify/product`](./shopify-product.md)** — for importing Shopify Product resources.
- **[`application/return`](./application-return.md)** — for app-owned return workflows.
- **[`application/shipment`](./application-shipment.md)** — for app-owned shipment workflows.
