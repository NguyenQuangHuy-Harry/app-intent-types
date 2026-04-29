# `application/loyalty-program`

Sidekick intent type for composing or editing a **loyalty program** — the configuration that defines how customers earn and redeem rewards (points, tiers, cashback, referrals).

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/loyalty-program.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/loyalty-program.json)

## When to register an intent for this type

Register an `application/loyalty-program` intent when your app can take a partial loyalty configuration — earning rules, redemption rules, tiers — and either (a) bootstrap a new program or (b) modify an existing one.

In practice, **`edit` is far more common than `create`**. Most stores have one loyalty program; the merchant's day-to-day ask is "tweak the earning multiplier" or "add a new tier", not "spin up a second program."

Typical examples:
- A **points-based loyalty app** registering `edit` for "double the points on apparel for the next two weeks."
- A **tiered loyalty app** registering `edit` for "add a Platinum tier above Gold with 3x points."
- A **new-merchant onboarding flow** registering `create` for "set up a basic 1-point-per-dollar program."

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "tune-loyalty"
handle = "tune-loyalty"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/TuneLoyalty.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/loyalty-program"
  action = "edit"
  schema = "./loyalty-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required. Most apps will only register `edit`.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./loyalty-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/loyalty-program.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The loyalty program ID. Used for `edit` actions. |

`additionalProperties: true`. Schema is a stub at this stage.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `name` — display name for the program
- `type` — `"points" | "tiered" | "cashback" | "punch_card" | "referral"`
- `currency_name` — what the merchant calls the points (e.g. "Stars", "Coins")
- `earning_rules` — array of `{ trigger, multiplier, condition }` (e.g. `1 point per $1 spent`, `2x on apparel`)
- `redemption_rules` — array of `{ cost, reward, condition }` (e.g. `100 points = $5 off`)
- `tiers` — array of `{ name, threshold, benefits }` (only for tiered programs)
- `expiry_policy` — `{ months_inactive, action }`

## Common pitfalls

- **Treating a per-customer balance as the program.** Customer balances are separate state; this type describes the *rules*, not the ledger.
- **Letting Sidekick fill in earning/redemption math without a confirm step.** Loyalty rules compound across thousands of orders — a typo on the multiplier is expensive. Always have your UI surface a preview before save.
- **Assuming a single program.** Some merchants run a points program *and* a referral program. If your app supports both, register `edit` only after disambiguating which program the merchant means.

## Related types

- No close relatives. Loyalty stands alone. Promotions and campaigns ([`application/campaign`](./application-campaign.md)) are the closest neighbor when "boost loyalty earning for a week" is structured as a campaign.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
