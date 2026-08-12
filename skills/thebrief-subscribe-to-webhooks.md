---
name: Subscribe to The Brief team webhooks
description: Discover the available webhook action types, create a team webhook bound to them, and handle deliveries safely given that The Brief documents no signature verification.
api: graphql/thebrief-public.graphql
surfaces: [rest, graphql]
operations: [teamWebhookActions, teamWebhooks, createTeamWebhook, updateTeamWebhook, deleteTeamWebhook]
rest_endpoints:
  - GET https://api.thebrief.ai/v1/webhookActions
  - GET https://api.thebrief.ai/v1/webhooks
  - POST https://api.thebrief.ai/v1/webhooks
  - DELETE https://api.thebrief.ai/v1/webhooks
generated: '2026-08-12'
method: generated
source: >-
  Grounded in graphql/thebrief-public.graphql (live introspection 2026-08-12) and
  https://docs.thebrief.ai/public-api/rest-api/webhooks. Every operation name and path above
  appears verbatim in the provider's own schema or documentation.
---

# Subscribe to The Brief team webhooks

The Brief has two distinct callback mechanisms. Do not confuse them:

| | Team webhooks | Export callback |
|---|---|---|
| Scope | Team-wide subscription | One export |
| Created by | `POST /v1/webhooks` | `webhookUrl` on the export request |
| Lifetime | Persistent | Fires once |
| Events | The action-type catalogue | Export completion |

This skill covers the first. For the second see `thebrief-generate-ad-variations.md`.

## 1. Discover the action types — do not hardcode them

```
GET https://api.thebrief.ai/v1/webhookActions
→ 200 {"response": [{"id": 1, "name": "comment/create", "description": "Comment created",
                     "config": null, "createdAt": "..."}, ...]}
```

(`teamWebhookActions` query.) **The catalogue is runtime data, not a published constant.** The
docs show `comment/create` (id 1) and `session/finish` (id 5) as examples only — the ids are not
contiguous and the full list is whatever your account is served. Always read this endpoint and
map by `name`, never by a memorised id.

Names follow `<resource>/<verb>`, lower-case and slash-separated.

## 2. Create the subscription

```
POST https://api.thebrief.ai/v1/webhooks
{"name": "<label>", "url": "https://your-receiver.example/hook", "actions": [1, 5]}
```

(`createTeamWebhook` mutation.) `name`, `url` and `actions` are all required; `actions` is the
array of numeric action ids resolved in step 1. The response echoes the webhook with its `id`,
`webhookUrl`, resolved `actions[]` (each carrying the full action object), `createdAt` and
`createdByUser`.

Use `GET /v1/webhooks` (`teamWebhooks`) to list what already exists before creating — there is no
idempotency key, so re-running this creates a duplicate subscription and you will receive every
event twice.

## 3. Receive deliveries — assume nothing about authenticity

**The Brief documents no signing secret, no HMAC header, and no replay protection.** There is
nothing in the published documentation or the GraphQL schema that lets a receiver prove a delivery
came from The Brief. Therefore:

- Treat the payload as an untrusted *hint*, not as data. On receipt, call back into the API with
  your own credentials to fetch authoritative state before acting.
- Use an unguessable receiver path and, if your stack allows, IP or mTLS controls at the edge.
- Expect possible duplicates: no delivery guarantee or retry policy is published either. Make your
  handler idempotent on your own side, keyed on the resource identifier in the payload.

## 4. Change or remove it

`updateTeamWebhook` modifies the URL or the bound action set; `deleteTeamWebhook`
(`DELETE /v1/webhooks`) removes it. Deleting is the only documented way to stop delivery.

## Related: the Zapier path

If the receiver is a workflow rather than a service, The Brief publishes a Zapier app with both a
native trigger and a webhook trigger, which removes the need to run a receiver at all —
`https://docs.thebrief.ai/zapier-integration`. The token scope for it is `ZAPIER`.

## Rules that apply throughout

Bearer JWT auth (`authentication/thebrief-authentication.yml`); 100 requests / 10 seconds / team;
flat `{"error": "<string>"}` envelope. Full webhook catalogue in `asyncapi/thebrief-webhooks.yml`.
