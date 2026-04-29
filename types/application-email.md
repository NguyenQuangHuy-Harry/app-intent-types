# `application/email`

Sidekick intent type for composing or editing an **email** that an app will send on the merchant's behalf.

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/email.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/email.json)

## When to register an intent for this type

Register an `application/email` intent when your app can take a partially-specified email — recipients, subject, body — and either (a) draft a new one or (b) modify an existing one in your app's UI.

Typical examples:
- A **marketing app** registering `create` so Sidekick can hand off "draft a back-in-stock email for last week's restocked SKUs" to your campaign builder.
- A **CRM app** registering `edit` so a merchant can ask Sidekick to "tweak the welcome email I drafted yesterday" and Sidekick can deep-link into your existing draft.
- A **support / messaging app** registering `create` for "send a follow-up email to the customer from order #1234".

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "compose-email"
handle = "compose-email"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/ComposeEmail.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/email"
  action = "edit"
  schema = "./email-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required. Register the same extension twice (one block per action) if you support both.
3. **`schema`** points to a local JSON Schema file. Required. It must `$ref` the canonical schema for this type, and it **must not declare `required` fields** — Sidekick will collect missing fields from the merchant before invoking your extension.

## Example: the input schema

`./email-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/email.json"
}
```

That's it for the simple case. If your extension only handles a subset of the type — say, you only support transactional emails, not marketing campaigns — narrow the schema by composing the `$ref` with additional constraints rather than redefining fields.

## Fields the schema describes

The canonical schema is the source of truth; this is a high-level orientation:

| Field | Notes |
|---|---|
| `to`, `cc`, `bcc` | Recipient addresses. May be partial — Sidekick can fill in. |
| `subject` | Email subject line. |
| `body` | Email body. May be plain text or HTML, depending on schema sub-type. |
| `from` | Sending address. Often determined by the app, not the merchant. |
| `attachments` | Optional. Schema describes URI references, not raw bytes. |

For the authoritative field list, fetch the schema URL above.

## Common pitfalls

- **Declaring `required` in your `inputSchema`.** Sidekick will reject the extension at registration time. The pattern is "schema describes the shape, Sidekick collects the data."
- **Treating the intent as a webhook.** It's not — Sidekick *navigates the merchant into your extension's UI* with the intent payload pre-filled. Your UI is responsible for letting the merchant review and confirm before anything sends.
- **Registering both `create` and `edit` with the same handler when they need different UIs.** It's fine to share, but if "edit" requires loading an existing draft and "create" doesn't, register them separately so you can branch on `action` cleanly.

## Related types

- **`application/campaign`** — for multi-recipient marketing sends with scheduling, segmentation, and tracking. If the email is a campaign, register that type instead.
- **`application/ticket`** — for support replies threaded into an ongoing customer conversation. Use this when the "email" is really a ticket reply.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
