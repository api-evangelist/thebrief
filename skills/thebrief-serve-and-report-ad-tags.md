---
name: Serve a design as an ad tag and read its delivery report
description: Enable The Brief's native ad server on a design, configure the click tag and networks, generate embed code or a CM360/DV360 trafficking file, and pull the delivery report.
api: graphql/thebrief-public.graphql
surfaces: [rest, graphql]
operations: [adNetworksForAdServing, adNetworksForHTML, adTagEnable, adTagDisable, adTagUpdateSettings, adTagGenerateCode, adTagGenerateFile, adTagReportData]
rest_endpoints:
  - GET https://api.thebrief.ai/v1/adnetworks/forAdserving
  - GET https://api.thebrief.ai/v1/adnetworks/forHtml
  - POST https://api.thebrief.ai/v1/adServing/{design_hash}
  - POST https://api.thebrief.ai/v1/adServing/{design_hash}/update-settings
  - POST https://api.thebrief.ai/v1/adServing/{design_hash}/generate-code
  - GET https://api.thebrief.ai/v1/adServing/{design_hash}/generate-file?format=CSV
  - GET https://api.thebrief.ai/v1/adServing/report
  - DELETE https://api.thebrief.ai/v1/adServing/{design_hash}
generated: '2026-08-12'
method: generated
source: >-
  Grounded in graphql/thebrief-public.graphql (live introspection 2026-08-12) and
  https://docs.thebrief.ai/public-api/rest-api/ad-serving. Every operation name and path above
  appears verbatim in the provider's own schema or documentation.
---

# Serve a design as an ad tag and read its delivery report

The Brief runs its own ad server. Once a design is ad-tag enabled, the creative is hosted and
served by The Brief, so it can be updated after trafficking without re-trafficking the tag.

## 1. Pick the network target

- `adNetworksForAdServing` (`GET /v1/adnetworks/forAdserving`) — networks valid for served ad tags.
- `adNetworksForHTML` (`GET /v1/adnetworks/forHtml`) — networks valid for packaged HTML5 export.

Use these two rather than the bare `adNetworks` query — the schema marks `adNetworks` as
`@deprecated` with exactly this replacement guidance. Each network carries `id`, `name`, `type`
and the `adTag` / `html5` capability flags; keep the `id` for `networkIds`.

## 2. Enable the ad tag

```
POST https://api.thebrief.ai/v1/adServing/{design_hash}
→ 200 {"response": true}
```

(`adTagEnable` mutation.) Two documented constraints, both of which throw rather than no-op:

- Ad tags are supported on **designs, not templates**.
- Enabling an already-enabled design errors. Because there is no idempotency key, treat this
  endpoint as "check state, then act" — a blind retry of a partially-applied enable produces an
  error, not a silent success.

## 3. Configure it

`POST /v1/adServing/{design_hash}/update-settings` (`adTagUpdateSettings`). The settings object is
`AdTagDesignSettings`:

- `url` — the click-through destination
- `target` — `_blank` (new window/tab) or `_top` (top-level window), per the `AdTagClickTarget` enum
- `useAsClickTag` — expose the URL as the standard display clickTag
- `responsiveScaling` — scale the ad to its container, preserving aspect ratio
- `networkIds` — the network ids from step 1

## 4. Get something to traffic

- **Embed code**: `POST /v1/adServing/{design_hash}/generate-code` (`adTagGenerateCode`) returns the
  tag markup for the selected network.
- **Bulk trafficking sheet**: `GET /v1/adServing/{design_hash}/generate-file?format=CSV` or
  `?format=XLSX` (`adTagGenerateFile`; the `AdTagCodeFormat` enum defines exactly these two values).
  This is the CM360 / DV360 path — one row per size, ready to upload.

## 5. Read delivery back

`adTagReportData` (`GET /v1/adServing/report`) returns delivery records. Break the report down with
the `AdTagReportBreakdown` enum — the only accepted dimensions are `project`, `design`, `network`,
`design_size`, `date_daily`, `date_monthly`, `feed_variation`.

Impressions are plan-capped: 100,000/month on Amplify Ultra and Collaborate Team, custom on
Enterprise, and ad serving is not included on Create Pro. See `plans/thebrief-plans-pricing.yml`.

## 6. Turn it off

```
DELETE https://api.thebrief.ai/v1/adServing/{design_hash}
→ 200 {"response": true}
```

(`adTagDisable`.) Disabling a design that is not enabled errors. Serving stops for live placements —
this is a destructive action on anything already trafficked; confirm before calling it.

## Rules that apply throughout

Bearer JWT auth (`authentication/thebrief-authentication.yml`), 100 requests / 10 seconds / team on
these routes with no `RateLimit-*` headers (`rate-limits/thebrief-rate-limits.yml`), and the flat
`{"error": "<string>"}` error envelope (`errors/thebrief-error-codes.yml`).
