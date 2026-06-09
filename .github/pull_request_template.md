<!--
Thanks for proposing a Sidekick intent type. Fill out as much as you can —
sections you can't answer yet are fine, just say so.

If this is an early-stage idea, consider opening an RFC discussion first:
https://github.com/Shopify/app-intent-types/discussions/categories/rfc
-->

## Proposed type

- **Type:** `application/...` or `shopify/...`
- **Actions:** `create` / `edit` / both, or `import` / `import+bulk` / both
- **One-line summary:**

## Why this type

What merchant intents does this capture? Real examples beat hypotheticals.

1.
2.
3.

## Who would register intents for it

Apps you've talked to (yours plus others), or app categories you'd expect to register.

-

## Schema sketch

Either a JSON Schema fenced below, or a link to a Gist / branch / external schema.

```json
{
}
```

Confirmed:
- [ ] `inputSchema` does not declare `required` fields
- [ ] Field names are consistent with neighboring types in `types/`
- [ ] For `application/*`, the schema refs the canonical application schema
- [ ] For `shopify/*`, `value` and `outputSchema` use the matching Shopify resource GID schema

## Prior art

Existing types this is closest to, and why a new type rather than extending one of them:

-

## Anything else

Open questions, things you're unsure about, areas where you'd especially like reviewer input.

-
