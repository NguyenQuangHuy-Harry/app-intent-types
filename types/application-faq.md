# `application/faq`

Sidekick intent type for composing or editing an **FAQ entry** — a question/answer pair shown to shoppers on the storefront, in a help center, or in a chatbot knowledge base.

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/faq.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/faq.json)

## When to register an intent for this type

Register an `application/faq` intent when your app can take a question/answer pair and either (a) add a new FAQ entry or (b) revise an existing one.

Typical examples:
- A **help-center app** registering `create` for "add an FAQ about our new return window."
- A **storefront FAQ widget** registering `edit` for "the shipping FAQ is outdated — update it to reflect the new carrier."
- A **support-app sidekick** registering `create` for "I keep getting this question — turn it into an FAQ" (driven by ticket trends).

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "manage-faq"
handle = "manage-faq"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/ManageFaq.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/faq"
  action = "create"
  schema = "./faq-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./faq-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/faq.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The FAQ ID. Used for `edit` actions. |

`additionalProperties: true`. Schema is a stub at this stage.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `question` — the FAQ question text
- `answer` — the answer body (plain text or Markdown)
- `category` — grouping (e.g. "Shipping", "Returns", "Sizing")
- `slug` — URL-safe identifier
- `language` — IETF language tag (e.g. `"en"`, `"fr-CA"`)
- `visibility` — `"public" | "draft" | "archived"`
- `tags` — array of free-form tags

## Common pitfalls

- **Conflating FAQ with help article.** FAQs are short — one question, one answer. If the answer needs subheadings or steps, it's a help article, not an FAQ.
- **Putting policy or legal text in an FAQ.** Returns *policy* lives on a policy page; the FAQ summarizes and links. Don't let Sidekick draft binding policy language under this type.
- **Skipping language.** Storefronts increasingly serve multiple locales. Even single-locale stores benefit from being explicit so a future translation app can find the entry.

## Related types

- **[`application/ticket`](./application-ticket.md)** — when an FAQ is being created in response to a recurring ticket trend, the source ticket is a `ticket` intent.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
