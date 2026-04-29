# `application/review`

Sidekick intent type for composing or editing a **product review** — a customer-authored rating and comment about a product, or a merchant's reply to one.

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/review.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/review.json)

## When to register an intent for this type

Register an `application/review` intent when your app can take a partial review spec — product, rating, body — and either (a) submit a new review or (b) modify an existing one.

Typical examples:
- A **review-collection app** registering `create` for "log a review for the customer who just left feedback in our DMs."
- A **review moderation app** registering `edit` for "edit the response to the 1-star review on the wireless earbuds."
- A **review-response automation** registering `edit` for "draft a reply to the latest review for product X" (where the *reply* is a field on the review, not a separate type).

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "manage-review"
handle = "manage-review"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/ManageReview.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/review"
  action = "edit"
  schema = "./review-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./review-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/review.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The review ID. Used for `edit` actions. |

`additionalProperties: true`. Schema is a stub at this stage.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `product_id` — the product being reviewed
- `rating` — integer 1–5
- `title` — short headline
- `body` — review body text
- `author` — `{ name, email }` (consumer attribution)
- `verified_purchase` — boolean
- `language` — IETF language tag
- `merchant_reply` — `{ body, replied_at }` for the merchant-facing response
- `status` — `"published" | "pending" | "rejected"` (for moderation flows)

## Common pitfalls

- **Letting the merchant edit the customer's review text.** Editing a customer's words is a serious trust and compliance issue. Most review apps only let merchants reply or moderate — not rewrite. Make sure your UI scopes `edit` to fields the merchant is allowed to change.
- **Conflating review with reply.** Some apps model the merchant's reply as a separate review. The convention here is one record per review with an embedded `merchant_reply`. Diverging from that breaks cross-app reasoning.
- **Skipping verified-purchase signals.** If your app knows whether the reviewer actually bought the item, surface it — review-credibility downstream depends on it.

## Related types

- **[`application/ticket`](./application-ticket.md)** — when a review escalates to a support issue (a 1-star review that needs follow-up).

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
