# `shopify/customer`

Sidekick intent type for importing one or more **Shopify customers** into an app-provided workflow and returning Shopify Customer GIDs.

- **Status:** ✅ Supported
- **Actions:** `import`, `import+bulk`
- **GID schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/customer/gid.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/customer/gid.json)

## When to register an intent for this type

Register a `shopify/customer` intent when your app operates on an existing Shopify Customer identified by a GID and produces a Shopify Customer result.

Typical examples:
- A **CRM app** registering `import` so a merchant can import a Shopify customer into a customer-enrichment workflow.
- A **loyalty app** registering `import` so a merchant can apply loyalty-specific state to a Shopify customer and return the updated customer reference.
- A **segmentation app** registering `import+bulk` so a merchant can process a set of Shopify customers in a bulk workflow.

Use `shopify/customer` when the source and output are Shopify Customer resources. If your app owns the artifact shape instead, use an `application/*` type.

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2026-04"

[[extensions]]
name = "Import customer"
description = "Take an existing Shopify customer and import them into an app workflow"
handle = "import-customer"
type = "admin_link"

  [[extensions.targeting]]
  target = "admin.app.intent.link"
  url = "/customers/{id}/import"
  tools = "./tools.json"
  instructions = "./instructions.md"

  [[extensions.targeting.intents]]
  type = "shopify/customer"
  action = "import"
  schema = "./customer-import-schema.json"
```

Three things to notice:

1. **`type`** is the Shopify resource type from this catalog. Required.
2. **`action`** is `import` for one resource or `import+bulk` for many resources. Required.
3. **`schema`** points to a local JSON Schema file. Required. Use `shopify-intent.json` for `import` and `shopify-intent-bulk.json` for `import+bulk`.

## Example: the import schema

`./customer-import-schema.json`:

```json
{
  "$schema": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify-intent.json",
  "value": {
    "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/customer/gid.json",
    "mapTo": "param",
    "fieldName": "id"
  },
  "inputSchema": {
    "type": "object",
    "properties": {
      "instructions": {
        "type": "string",
        "description": "Merchant-provided instructions for the customer workflow."
      }
    }
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "id": {
        "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/customer/gid.json"
      }
    }
  }
}
```

## Fields the schema describes

A Shopify resource intent schema has a fixed shape:

| Field | Type | Notes |
|---|---|---|
| `value` | Shopify Customer GID | Source customer passed to the app. Uses the `shopify/customer` GID schema. |
| `inputSchema` | object | App-specific input fields. Do not declare `required` fields. |
| `outputSchema.properties.id` | Shopify Customer GID | Result customer returned by `shopify.intents.response.ok`. |

For `import+bulk`, `value` is an array of Customer GIDs and the output should return an array under `outputSchema.properties.ids`.

## Common pitfalls

- **Using `create` or `edit` with `shopify/customer`.** Those verbs are reserved for Shopify-native Admin intents. Third-party app extensions use `import` or `import+bulk`.
- **Using the wrong meta-schema.** `import` requires `shopify-intent.json`; `import+bulk` requires `shopify-intent-bulk.json`.
- **Pointing at an application schema.** `value.$ref` must point to the Customer GID schema, not an `application/*` schema.
- **Treating the URL `{id}` as a full GID.** When mapped as a `param`, Sidekick passes the bare tail ID into the URL path. Read the full GID from the intent payload if you need it.

## Related types

- **[`shopify/product`](./shopify-product.md)** — for importing Shopify Product resources.
- **[`shopify/order`](./shopify-order.md)** — for importing Shopify Order resources.
- **[`application/loyalty-program`](./application-loyalty-program.md)** — for app-owned loyalty program artifacts.
- **[`application/ticket`](./application-ticket.md)** — for app-owned support ticket workflows.
