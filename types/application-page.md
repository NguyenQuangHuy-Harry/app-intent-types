# `application/page`

Sidekick intent type for composing or editing a **storefront page** — a merchant-authored page of content and layout that an app builds and publishes to the online store.

- **Status:** 🚧 Proposed
- **Actions:** `create`, `edit`
- **Schema:** _draft — see [Draft schema](#draft-schema) below. Canonical URL to be assigned on publish:_ `https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/page.json`

## When to register an intent for this type

Register an `application/page` intent when your app can take a partially-specified page — a goal, an audience, some products to feature, a rough section outline — and either (a) build a new page or (b) open an existing one for modification in your app's editor.

Typical examples:

- A **page builder** registering `create` so Sidekick can hand off "build me a landing page for the summer collection launch" to the app's editor with the collection and a section outline pre-filled.
- A **page builder** registering `edit` so a merchant can ask Sidekick to "add a testimonials section to my Black Friday page" and Sidekick can deep-link into the existing page.
- A **template / theme-section app** registering `create` for "make a comparison page for these three products", where the app owns the layout and publishes to a template suffix.
- A **CRO / landing-page app** registering `create` for "spin up a landing page for this ad campaign" as the destination step after an `application/ad` intent.

## Why this isn't covered by an existing type

The closest existing types fall short in different directions:

- **`shopify/product` (`import`)** carries a product GID into the app, which covers "build a page *for this product*". It cannot express a page that isn't anchored to one resource — an About page, a campaign landing page, a multi-product comparison — and it carries no page-level intent (title, handle, outline, goal).
- **`application/campaign`** describes a marketing campaign's schedule, audience, and channels. A page is frequently the *destination* of a campaign, not the campaign itself, and a merchant creates plenty of pages with no campaign behind them.
- **`application/faq`** overlaps only for the narrow case where the page *is* an FAQ. An app that builds an FAQ page should keep registering `application/faq`.

## Apps that would register for this type

Storefront page building is a mature, multi-vendor category on the Shopify App Store. Apps that build merchant-authored pages and would plausibly register this type include PageFly, GemPages, Shogun, Replo, EComposer, Zipify Pages, and LayoutHub, alongside theme-section and template marketplaces that publish pages on the merchant's behalf. This proposal is opened by PageFly, but the shape below is deliberately vendor-neutral — it describes the page a merchant ends up with, not any one app's document model.

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "build-page"
handle = "build-page"
type = "admin_link"

  [[extensions.targeting]]
  target = "admin.app.intent.link"
  url = "/editor/{id}"
  tools = "./tools.json"
  instructions = "./instructions.md"

  [[extensions.targeting.intents]]
  type = "application/page"
  action = "create"
  schema = "./page-schema.json"
```

Same three rules as every other type in this catalog:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required. Register the same extension twice (one block per action) if you support both.
3. **`schema`** points to a local JSON Schema file that `$ref`s the canonical schema, and **must not declare `required` fields** — Sidekick collects missing fields from the merchant before invoking your extension.

## Example: the input schema

`./page-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/page.json"
}
```

## Fields the schema describes

| Field | Type | Notes |
|---|---|---|
| `id` | string | The page ID in the app. Used for `edit` actions. |
| `title` | string | Page title as the merchant would name it. |
| `handle` | string | URL slug. |
| `pageType` | string | Storefront surface the page targets. See the vocabulary below. |
| `goal` | string | What the merchant wants the page to accomplish, in their words — "launch the summer collection", "explain our return policy". This is the field Sidekick can fill most reliably from a merchant's prompt. |
| `sections` | array of `{type, heading, body, mediaUrl}` | A rough content outline. `type` is a loose hint (`hero`, `features`, `testimonials`, `faq`, `cta`), not a closed enum — apps map it onto their own section library. |
| `products` | array of string | Product GIDs to feature on the page. |
| `collections` | array of string | Collection GIDs to feature on the page. |
| `status` | string | `draft` or `published`. |
| `seo` | object `{title, description}` | SEO overrides where they differ from `title`. |

`additionalProperties: true` — apps may pass extra fields, but Sidekick won't validate them.

### `pageType` vocabulary

Deliberately aligned to Shopify storefront surfaces rather than to any app's internal taxonomy:

`home` · `product` · `collection` · `blog-post` · `page` · `password`

`page` is the default and covers standalone pages including landing pages. Apps that distinguish "landing page" from "regular page" internally can keep doing so — that distinction is an app concern, and collapsing it here keeps the type from fragmenting.

**Fields deliberately left out**, since they describe an app's document model rather than the merchant-facing artifact: element/component trees, style objects, breakpoint definitions, theme or template file bindings, and revision history.

**Fields worth discussing for inclusion:**

- `locale` — for merchants running multi-language storefronts
- `templateSuffix` — where the app publishes to an Online Store 2.0 template rather than a standalone page
- `brandVoice` / `tone` — a hint many builders already accept
- `referenceUrl` — an existing page the merchant wants matched in style

## Common pitfalls

- **Declaring `required` in your `inputSchema`.** Sidekick will reject the extension at registration time. The schema describes the shape; Sidekick collects the data.
- **Treating the intent as a build API.** It isn't — Sidekick navigates the merchant into your app with the intent payload pre-filled. Page generation is often slow and always subjective, so the merchant reviews and confirms in your editor before anything is published.
- **Modelling your own document format in the schema.** Two page builders will never agree on a component tree. They can agree on "a page titled X, for surface Y, featuring products Z, roughly these sections."
- **Registering both `create` and `edit` against the same handler** when `edit` needs to load an existing page and `create` doesn't. Sharing is fine; branch on `action` if the flows diverge.

## Related types

- **[`application/faq`](./application-faq.md)** — when the page *is* an FAQ, register that instead.
- **[`application/campaign`](./application-campaign.md)** — for the campaign a page is the destination of. The two compose: a campaign intent and a page intent can reasonably fire in sequence.
- **[`shopify/product`](./shopify-product.md)** — for "build something from this product" where the product GID is the whole payload and the app decides the rest.
- **`application/theme-edit-task`** ([proposed](../../discussions)) — for editing theme *code*. Distinct from this type: that one targets the theme's source, this one targets a merchant-authored page.

## Open questions for reviewers

1. **Is `sections` too app-specific to be worth carrying?** It's the field most likely to vary between builders. The alternative is dropping it and letting `goal` carry all the intent, at the cost of losing structure Sidekick could otherwise fill.
2. **Should `pageType` be a closed enum or an open string?** Closed is safer for routing; open avoids a schema revision every time Shopify adds a storefront surface.
3. **Is a `publish` action out of scope?** Every `application/*` type in the catalog today declares exactly `create` and `edit`, so this proposal follows that convention — but "publish the page I made yesterday" is a plausible merchant ask, and it's a state transition rather than an edit.

## Draft schema

Inline draft for review — no `required` anywhere.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/page.json",
  "title": "Page",
  "description": "A merchant-authored storefront page that an app builds and publishes to the online store.",
  "type": "object",
  "additionalProperties": true,
  "properties": {
    "id": {
      "type": "string",
      "description": "The page ID in the app. Used for edit actions."
    },
    "title": {
      "type": "string",
      "description": "Page title as the merchant would name it."
    },
    "handle": {
      "type": "string",
      "description": "URL slug for the page."
    },
    "pageType": {
      "type": "string",
      "description": "Storefront surface the page targets.",
      "enum": ["home", "product", "collection", "blog-post", "page", "password"],
      "default": "page"
    },
    "goal": {
      "type": "string",
      "description": "What the merchant wants the page to accomplish, in their own words."
    },
    "sections": {
      "type": "array",
      "description": "Rough content outline. Apps map these onto their own section libraries.",
      "items": {
        "type": "object",
        "additionalProperties": true,
        "properties": {
          "type": {
            "type": "string",
            "description": "Loose section hint, e.g. hero, features, testimonials, faq, cta."
          },
          "heading": { "type": "string" },
          "body": { "type": "string" },
          "mediaUrl": { "type": "string", "format": "uri" }
        }
      }
    },
    "products": {
      "type": "array",
      "description": "Product GIDs to feature on the page.",
      "items": { "type": "string" }
    },
    "collections": {
      "type": "array",
      "description": "Collection GIDs to feature on the page.",
      "items": { "type": "string" }
    },
    "status": {
      "type": "string",
      "enum": ["draft", "published"],
      "default": "draft"
    },
    "seo": {
      "type": "object",
      "additionalProperties": true,
      "properties": {
        "title": { "type": "string" },
        "description": { "type": "string" }
      }
    }
  }
}
```

## Discussion history

- Original proposal: this PR.
