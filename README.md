# TheBrief

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

The Brief (formerly Creatopy) is an AI-powered advertising creation platform for brands and
agencies — Ad Studio, AI generation, a native ad server, and direct publishing to Meta, Google Ads,
CM360, DV360 and Veeva. Founded 2021, based in Romania, backed by Point Nine.

## API surface

The Brief ships a documented **Public API** on two co-equal surfaces:

| Surface | Base | Contract |
|---|---|---|
| REST | `https://api.thebrief.ai/v1` | Documented only (no OpenAPI published) |
| GraphQL | `https://graphql.thebrief.ai/public` | **Live schema** — anonymous introspection returns 186 types, 48 queries, 49 mutations |

Documentation: <https://docs.thebrief.ai/public-api> · Postman:
<https://www.postman.com/thebrieftechnical/the-brief-api/collection/1lfct1g/graphql-api>

Authentication is a JWT bearer token minted from a `clientId`/`clientSecret` pair created in-app
under *Manage account > API credentials*. The same signed token drives the **App Integration**
flow, which embeds the Ad Studio editor inside a customer's own product. There is also a team
**webhook** surface, a **Zapier** app and a **Figma** plugin.

## What this profile captures

`graphql/` holds the SDL rendered verbatim from the live introspection response, alongside the raw
JSON. `authentication/`, `conventions/`, `errors/`, `rate-limits/`, `lifecycle/`, `data-model/`,
`conformance/`, `components/`, `asyncapi/` (webhooks) and `plans/` are read from the provider's own
documentation and schema. `skills/` packages four agent-facing flows, each grounded in operation
names verified in that schema.

**Notable gaps, all recorded with evidence:** no OpenAPI, no `/.well-known/` documents on any host,
no `security.txt`, no MCP server, no A2A agent card, no status page, no changelog, no roadmap, no
first-party SDK in any package registry, no idempotency key, and no `RateLimit-*` / `Retry-After`
response headers despite published numeric limits.

Backed by: point-nine — https://www.thebrief.ai
