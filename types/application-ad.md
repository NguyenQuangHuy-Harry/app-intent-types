# `application/ad`

Sidekick intent type for composing or editing a **paid advertising creative** — a single ad asset that runs on a paid channel (Google, Meta, TikTok, Shopify Audiences, etc.).

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/ad.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/ad.json)

## When to register an intent for this type

Register an `application/ad` intent when your app can take a partial ad spec — creative, copy, target URL — and either (a) draft a new ad or (b) modify an existing one in your app's UI.

Typical examples:
- An **ads management app** registering `create` so Sidekick can hand off "draft a Meta ad promoting the new collection" to your ad builder.
- A **paid-marketing optimization app** registering `edit` so a merchant can ask Sidekick to "tighten up the copy on yesterday's ad and pause it after Friday."
- A **Shopify Audiences** integration registering `create` for "spin up a retargeting ad for cart abandoners."

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "draft-ad"
handle = "draft-ad"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/DraftAd.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/ad"
  action = "create"
  schema = "./ad-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required. Register separate blocks if you handle both.
3. **`schema`** is your local JSON Schema. It must `$ref` the canonical schema and **must not declare `required` fields**.

## Example: the input schema

`./ad-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/ad.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The ad ID. Used for `edit` actions. |

`additionalProperties: true` — apps may pass extra fields, but Sidekick won't validate them. The schema is intentionally light at this stage; the conversation about what fields it should capture is open.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `name` — internal label for the ad
- `platform` — `"google" | "meta" | "tiktok" | "pinterest" | "audiences"`
- `format` — `"image" | "video" | "carousel" | "text"`
- `headline`, `primary_text`, `description` — the ad copy
- `image_url`, `video_url` — creative assets (URI references)
- `landing_url` — destination URL
- `audience_id` — reference to a saved audience
- `budget` — `{ amount, currency, schedule }`

## Common pitfalls

- **Pretending the schema is platform-specific.** `application/ad` is intentionally platform-neutral. Don't reject the intent if `platform` is missing — let Sidekick collect it via UI, or default to your app's primary platform.
- **Spending money implicitly.** Ad budgets touch real spend. Even though the intent passes through your UI, treat the budget field as advisory — always show the merchant the final spend before launch.
- **Conflating ad with campaign.** A single ad creative is `application/ad`. The container that schedules and budgets across multiple ads is [`application/campaign`](./application-campaign.md).

## Related types

- **[`application/campaign`](./application-campaign.md)** — when the ad is part of a coordinated multi-channel push.
- **[`application/email`](./application-email.md)** — owned-channel equivalent (email instead of paid placement).

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
