---
name: Generate ad variations from a The Brief template
description: Take an existing The Brief template, discover its editable layers and sizes, submit element changes to produce a batch of on-brand creatives, and collect the rendered files.
api: graphql/thebrief-public.graphql
surfaces: [rest, graphql]
operations: [templates, templateElements, designSizes, generateCreative, export, listCreditOperations]
rest_endpoints:
  - POST https://api.thebrief.ai/v1/auth/token
  - GET https://api.thebrief.ai/v1/templates
  - GET https://api.thebrief.ai/v1/templates/{templateHash}/elements
  - GET https://api.thebrief.ai/v1/templates/{templateHash}/design-sizes
  - POST https://api.thebrief.ai/v1/export-with-changes
  - GET https://api.thebrief.ai/v1/creative/{creativeId}
generated: '2026-08-12'
method: generated
source: >-
  Grounded in graphql/thebrief-public.graphql (live introspection 2026-08-12) and the REST
  reference at https://docs.thebrief.ai/public-api/rest-api. Every operation name and path
  above appears verbatim in the provider's own schema or documentation.
---

# Generate ad variations from a template

The Brief's core loop is: a template is a design with named, editable layers; you send a set of
layer changes; The Brief renders one creative per requested size, asynchronously.

## 1. Authenticate

Exchange the API credential pair for a JWT.

```
POST https://api.thebrief.ai/v1/auth/token
Content-Type: application/json
{"clientId": "<uuid>", "clientSecret": "<uuid>"}
→ 200 {"token": "<jwt>"}
```

Send `Authorization: Bearer <jwt>` on every subsequent request. Credentials are created in-app
under **Manage account > API credentials** (`https://app.thebrief.ai/go-to/settings/api-credentials`).
You may also sign the JWT yourself with the secret — see `authentication/thebrief-authentication.yml`.

## 2. Find the template

`GET /v1/templates` (or the `templates` GraphQL query). Filter with `keyword`, `projectId`,
`folderId`, `onlyTemplates`, `exactSearch`; page with `limit` (max 50) and `cursor`; sort with
`orderBy` (`ID|NAME|CREATED_AT|UPDATED_AT`) and `orderDirection` (`ASC|DESC`).

Note the deprecation the provider published: templates are now managed under Brand Kits as
**Brand Templates** (`GET /v1/brandkits/templates`, `brandTemplates` query). Prefer that route for
new work; `/v1/templates` still serves.

Keep the template's `hash` — it is the handle for every step that follows.

## 3. Read the editable surface before changing anything

- `GET /v1/templates/{templateHash}/elements` (`templateElements` query) returns each layer's
  `name`, `layerType`, `value`, `visible`, `slideNumber` and `gotoUrl`. **Layer `name` is the
  substitution key** — never guess it, always read it.
- `GET /v1/templates/{templateHash}/design-sizes` (`designSizes` query) returns each size variant's
  `hash`, `name`, `width`, `height`, `measureUnit`. Use these hashes in `allowedSizeHashesOnly`
  when you want a subset of a set rather than the whole set.

## 4. Check the cost first

`listCreditOperations` (`GET /v1/credits/operations`) returns each billable operation and its
credit `cost`; `getSubscriptionInfo` (`GET /v1/credits/subscription-info?teamId=`) returns the
available/locked/consumed credit balance. Read both before a large batch — generation spends
credits and there is no dry-run mode.

Teams on a legacy plan get `400 {"error": "This endpoint is not available for teams that have a
legacy plan"}` from every `/v1/credits/*` route. Treat that as "cost unknown", not as a failure.

## 5. Submit the generation

`POST /v1/export-with-changes` (`generateCreative` mutation) with:

- `templateHash` (required) — from step 2
- `type` (required) — the output format (JPG, PNG, WEBP, GIF, HTML5, MP4, PDF, AMP)
- element changes keyed by the layer names read in step 3
- `exportSettings` (optional) — format-specific: `quality` (60/80/90/100 or custom), `gifPreset`
  (`highQuality|optimized|static`), `pdfPreset` (`PDF_STANDARD|PDF_PRINT`), `slide`, `scale`,
  `targetFPS`, `retina`, `retinaOnly`, `convertCustomFonts`, `minifyHtml`, `urlTarget`,
  `useAsClickTag`, `clickTagUrl`, `includeFallbackImage`, `allowedSizeHashesOnly`,
  `responsiveScaling`, `networkId`
- `webhookUrl` (optional) — a callback for completion instead of polling

**This is the step with no safety net.** There is no `Idempotency-Key` header and no request
deduplication anywhere in The Brief's API. A retried export creates an *additional* design and
spends credits again. Record the export id before retrying anything, and prefer polling a
possibly-succeeded export over resubmitting it.

## 6. Collect the output

Poll `GET /v1/creative/{creativeId}` (or the `export` query) until `status` leaves `pending` /
`inProgress`. Terminal states are `complete`, `completeWithError`, `failed`.

An export of a size set returns one creative per size, each with its own `status` and a
**presigned S3 `url` that expires** (the documented examples carry a one-hour expiry). Download
immediately or re-request the creative for a fresh URL.

`completeWithError` is a 200 response carrying partial success — inspect `errorLog` and the
per-creative statuses rather than trusting the HTTP code.

## Rules that apply throughout

- **Rate limits**: 15 requests / 10 seconds / team on export routes, 100 requests / 10 seconds /
  team elsewhere. There are **no** `RateLimit-*` or `Retry-After` response headers, so pace
  yourself from these numbers — a 429 is the only runtime feedback you will get.
- **Pagination**: `nodes[]` + `pageInfo{hasNextPage,endCursor}` + `totalCount`. Cursors are opaque
  base64; URL-encode them and never construct one.
- **Errors**: `{"error": "<string>"}`, not RFC 9457. Note that The Brief documents `410` as "user
  lacks access to a resource" — treat it as authorization, not deletion.
- See `conventions/thebrief-conventions.yml`, `rate-limits/thebrief-rate-limits.yml`,
  `errors/thebrief-error-codes.yml`.
