# `application/campaign`

Sidekick intent type for composing or editing a **marketing campaign** — a coordinated send across one or more channels (email, SMS, ads, push), often with a schedule, audience, and goal.

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/campaign.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/campaign.json)

## When to register an intent for this type

Register an `application/campaign` intent when your app can take a partial campaign brief — name, audience, schedule, channels — and either (a) draft a new campaign or (b) modify an existing one.

Typical examples:
- A **marketing automation app** registering `create` for "build a Black Friday campaign across email and SMS, send Friday morning."
- A **welcome-series app** registering `edit` for "tweak my abandoned-cart series — bump the second touch from day 2 to day 3."
- A **paid + owned coordination app** registering `create` for "launch a new-collection campaign — email blast, retargeting ads, and social posts."

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "build-campaign"
handle = "build-campaign"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/BuildCampaign.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/campaign"
  action = "create"
  schema = "./campaign-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./campaign-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/campaign.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The campaign ID. Used for `edit` actions. |

`additionalProperties: true`. The schema is a stub at this stage; the shape is being worked out by the community.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `name` — campaign name
- `channels` — array of `"email" | "sms" | "ad" | "push" | "social"`
- `schedule` — `{ start_at, end_at }` ISO 8601
- `audience_id` — reference to a saved segment
- `goal` — `"awareness" | "conversion" | "retention" | "winback"`
- `assets` — array of references to campaign content (`application/email` IDs, `application/ad` IDs, etc.)
- `budget` — `{ amount, currency }` for paid channels

## Common pitfalls

- **Confusing campaign with single send.** A campaign is the *container*. A one-off email is [`application/email`](./application-email.md). If your app can't handle multi-channel or scheduling, register `application/email` instead.
- **Locking in channels at registration time.** Campaigns evolve — a merchant may add SMS to an email-only campaign mid-edit. Don't reject the intent if a channel field is unset.
- **Treating audience_id as your internal ID.** If audiences live in Shopify (segments), reference the Shopify segment ID, not your internal mapping.

## Related types

- **[`application/email`](./application-email.md)** — single send, no scheduling/audience-segmentation/multi-channel logic.
- **[`application/ad`](./application-ad.md)** — single ad creative; campaigns can reference multiple ads.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
