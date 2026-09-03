# perfect-corp

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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Perfect Corp. is the beauty-tech company behind the YouCam apps and the **YouCam AI API** — a RESTful,
asynchronous, task-based platform exposing 60+ AI features for beauty, skincare, hair, fashion,
jewellery and generative image and video work, plus three hosted MCP servers for agent integration.

## What this profile holds

| Surface | What was found |
|---|---|
| **OpenAPI** | **65** per-feature OpenAPI **3.0.0** documents, **178** operations, all declaring `servers[0].url` `https://yce-api-01.makeupar.com`. Published for download at `https://docs.perfectcorp.com/_bundle/reference/<feature>.json?download` and saved verbatim in `openapi/_original/`. |
| **MCP** | **Three hosted remote servers** — beauty (47 tools), fashion (18), creators (34). `initialize` and `tools/list` answer **anonymously**, so the real tool set with per-tool JSON Schema is captured in `mcp/*-tools-list.json`. A fourth, documentation-search MCP server runs on the docs host. |
| **A2A** | An **Agent Card** served at `https://docs.perfectcorp.com/.well-known/agent-card.json`, protocolVersion 0.3.0, graded **conformant** — with an empty `skills[]` and an MCP extension as its real capability. |
| **Webhooks** | Task-completion callbacks implementing the **Standard Webhooks** specification (HMAC-SHA256, `whsec_` secrets, `webhook-id`/`-timestamp`/`-signature`). No AsyncAPI is published. |
| **/.well-known/** | Two real documents — an **RFC 8414** OAuth authorization-server metadata document on the API host (PKCE S256, RFC 7591 dynamic client registration, scopes `task.run` / `task.read`), and the agent card. |
| **llms.txt** | `https://docs.perfectcorp.com/llms.txt`, 377 KB, indexing the whole documentation set. |
| **Pricing** | Prepaid **units** ("Credits"), with a real per-feature price API — `GET /s2s/v2.0/credit/feature-cost` — and an MCP tool for it. |

## Notable findings

- **The OAuth surface is undocumented.** An RFC 8414 authorization server is served anonymously from
  the API host and names two scopes, but appears nowhere in the developer documentation and is
  required by no published operation. `scopes/` is the only public record of it.
- **No `operationId` anywhere.** All 178 operations lack one, so nothing can reference them by a
  stable identifier. Recorded in `overlays/`, never repaired into the provider's contract.
- **Errors arrive with HTTP 200.** Engine-level failures (`error_no_face`, `error_nsfw_content_detected`,
  and 20 more) come back inside a 200 body with `task_status: "error"`. See `errors/`.
- **No idempotency.** No `Idempotency-Key` on any operation; a retried task creation charges units
  again. The one idempotency signal is inbound — the retry-stable `webhook-id`.
- **One reversal, no cancel.** `POST /s2s/v2.0/task/delete` removes a finished task with its inputs
  and outputs, inside a stated 30-day retention window. There is no cancel, and no MCP tool exposes
  deletion — an agent working through MCP alone has no reversal path. See `conventions/`.
- **No first-party SDK** in any package registry, and no Perfect Corp GitHub organization
  (`github.com/perfectcorp` is an unrelated personal account with zero repositories).
- **No status page, no deprecation policy, no SOC 2 / ISO 27001**; GDPR and CCPA/CPRA are covered in
  a substantive SaaS privacy policy.
- **Rate limits are documented but not signalled.** 250 requests / 300 s per IP *and* per token, 429
  on exhaustion, and **no** `RateLimit-*` or `Retry-After` header on any response.

## Layout

`openapi/` (+ `_original/`) · `overlays/` · `mcp/` · `a2a/` · `well-known/` · `llms/` · `skills/` ·
`authentication/` · `scopes/` · `conventions/` · `errors/` · `data-model/` · `lifecycle/` ·
`changelog/` · `asyncapi/` · `rate-limits/` · `plans/` · `packages/` · `sandbox/` · `conformance/` ·
`security/`

Every artifact carries `generated` / `method` / `source` provenance frontmatter, and every probe
records the HTTP status it actually received.
