# `shopify/product`

Sidekick intent type for importing one or more **Shopify products** into an app-provided workflow and returning Shopify Product GIDs.

- **Status:** ✅ Supported
- **Actions:** `import`, `import+bulk`
- **GID schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/product/gid.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/product/gid.json)

## When to register an intent for this type

Register a `shopify/product` intent when your app operates on an existing Shopify Product identified by a GID and produces a Shopify Product result.

Typical examples:
- A **print-on-demand app** registering `import` so a merchant can start from a blank base product and create a custom-printed variant.
- A **product merchandising app** registering `import+bulk` so a merchant can import several products into a bulk optimization workflow.
- A **product enrichment app** registering `import` so a merchant can enhance a product with app-specific content, data, or assets before returning the resulting Shopify Product.

Use `shopify/product` when the source and output are Shopify Product resources. If your app owns the artifact shape instead, use an `application/*` type.

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2026-04"

[[extensions]]
name = "Customize product"
description = "Take an existing Shopify product and create a customized product result"
handle = "customize-product"
type = "admin_link"

  [[extensions.targeting]]
  target = "admin.app.intent.link"
  url = "/customize/{id}"
  tools = "./tools.json"
  instructions = "./instructions.md"

  [[extensions.targeting.intents]]
  type = "shopify/product"
  action = "import"
  schema = "./product-import-schema.json"
```

Three things to notice:

1. **`type`** is the Shopify resource type from this catalog. Required.
2. **`action`** is `import` for one resource or `import+bulk` for many resources. Required.
3. **`schema`** points to a local JSON Schema file. Required. Use `shopify-intent.json` for `import` and `shopify-intent-bulk.json` for `import+bulk`.

## Example: the import schema

`./product-import-schema.json`:

```json
{
  "$schema": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify-intent.json",
  "value": {
    "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/product/gid.json",
    "mapTo": "param",
    "fieldName": "id"
  },
  "inputSchema": {
    "type": "object",
    "properties": {
      "instructions": {
        "type": "string",
        "description": "Merchant-provided customization instructions."
      }
    }
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "id": {
        "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/shopify/product/gid.json"
      }
    }
  }
}
```

## Fields the schema describes

A Shopify resource intent schema has a fixed shape:

| Field | Type | Notes |
|---|---|---|
| `value` | Shopify Product GID | Source product passed to the app. Uses the `shopify/product` GID schema. |
| `inputSchema` | object | App-specific input fields. Do not declare `required` fields. |
| `outputSchema.properties.id` | Shopify Product GID | Result product returned by `shopify.intents.response.ok`. |

For `import+bulk`, `value` is an array of Product GIDs and the output should return an array under `outputSchema.properties.ids`.

## Common pitfalls

- **Using `create` or `edit` with `shopify/product`.** Those verbs are reserved for Shopify-native Admin intents. Third-party app extensions use `import` or `import+bulk`.
- **Using the wrong meta-schema.** `import` requires `shopify-intent.json`; `import+bulk` requires `shopify-intent-bulk.json`.
- **Pointing at an application schema.** `value.$ref` must point to the Product GID schema, not an `application/*` schema.
- **Treating the URL `{id}` as a full GID.** When mapped as a `param`, Sidekick passes the bare tail ID into the URL path. Read the full GID from the intent payload if you need it.

## Related types

- **[`shopify/customer`](./shopify-customer.md)** — for importing Shopify Customer resources.
- **[`shopify/order`](./shopify-order.md)** — for importing Shopify Order resources.
- **[`application/campaign`](./application-campaign.md)** — for app-owned marketing campaign artifacts.
