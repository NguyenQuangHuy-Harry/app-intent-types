# `application/ticket`

Sidekick intent type for composing or editing a **support ticket** — a threaded customer-support conversation, regardless of the channel it came in on (email, chat, phone, contact form).

- **Status:** ✅ Supported
- **Actions:** `create`, `edit`
- **Schema:** [`https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/ticket.json`](https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/ticket.json)

## When to register an intent for this type

Register an `application/ticket` intent when your app can take a partial ticket spec — customer, subject, message — and either (a) open a new ticket or (b) update an existing one (reply, change status, reassign).

Typical examples:
- A **helpdesk app** registering `create` for "open a ticket for the customer who DM'd us about a sizing question."
- A **support-agent app** registering `edit` for "reply to ticket #789 saying we're shipping a replacement and resolve it."
- A **CRM app with support features** registering `edit` for "escalate ticket #432 to high priority."

## Example: register the intent

In your extension's `shopify.extension.toml`:

```toml
api_version = "2025-04"

[[extensions]]
name = "manage-ticket"
handle = "manage-ticket"
type = "ui_extension"

  [[extensions.targeting]]
  module = "./src/ManageTicket.tsx"
  target = "admin.intent.render"

  [[extensions.targeting.intents]]
  type = "application/ticket"
  action = "edit"
  schema = "./ticket-schema.json"
```

Three things to notice:

1. **`type`** is the MIME type from this catalog. Required.
2. **`action`** is `create` or `edit`. Required. Both are common for ticket apps.
3. **`schema`** is your local JSON Schema, `$ref`-ing the canonical schema. **Must not declare `required` fields.**

## Example: the input schema

`./ticket-schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$ref": "https://extensions.shopifycdn.com/shopifycloud/schemas/v1/application/ticket.json"
}
```

## Fields the schema describes

**What the canonical schema captures today:**

| Field | Type | Notes |
|---|---|---|
| `id` | string | The ticket ID. Used for `edit` actions. |

`additionalProperties: true`. Schema is a stub at this stage.

**Fields commonly proposed for inclusion** (open an [RFC discussion](../../discussions/categories/rfc) to push for any of these):

- `customer_id` — the customer the ticket is from
- `subject` — short title for the ticket
- `body` — the latest message body (for `create`) or new reply (for `edit`)
- `channel` — `"email" | "chat" | "phone" | "form" | "social"`
- `priority` — `"low" | "normal" | "high" | "urgent"`
- `status` — `"open" | "pending" | "resolved" | "closed"`
- `assignee` — agent or team handle
- `tags` — array of free-form tags
- `order_id` — when the ticket relates to a specific order
- `internal_note` — boolean flag indicating the body is an internal-only note, not customer-visible

## Common pitfalls

- **Confusing ticket with email.** A ticket is a *thread* with status, assignee, and history. A one-shot email is [`application/email`](./application-email.md). If your app threads conversations, register `ticket`; if it just sends, register `email`.
- **Status changes leaking into create.** "Reopen ticket #X" is `edit`, not `create`. Make sure your UI branches cleanly on `action`.
- **Mixing internal notes with customer replies.** If your app supports both, surface the `internal_note` distinction prominently — sending an internal note to a customer by mistake is a serious failure mode.

## Related types

- **[`application/email`](./application-email.md)** — when the ticket originated from email or replies via email. The thread is the ticket; each send may be modeled as an email.
- **[`application/return`](./application-return.md)** — when a ticket is about a return; the RMA itself is the return type, the conversation is the ticket.
- **[`application/review`](./application-review.md)** — when a low-rated review escalates to support follow-up.

## Discussion history

- Original proposal: *(link to RFC discussion when this type was proposed)*
- Schema v1 published: *(date)*
