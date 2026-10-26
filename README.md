# Actionpower

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

ActionPower Corp. (주식회사 액션파워) is a Seoul deep-tech AI company founded in 2016 that builds its own
end-to-end speech recognition, speaker diarization, speech synthesis and small-language-model stack. It
sells that stack two ways: **daglo**, a voice-intelligence workspace used by more than two million people
to record, transcribe, summarize and translate meetings, calls and lectures; and the **daglo Cloud API**
at `https://apis.daglo.ai`, the developer platform this profile covers.

## What this profile found

| Surface | Evidence |
|---|---|
| OpenAPI 3.0.0, 9 operations (production) | `https://apis.daglo.ai/openapi.prod.yaml`, rendered at `https://apis.daglo.ai/docs` |
| OpenAPI 3.0.0, 30 operations (dev environment) | `https://apis.daglo.ai/openapi.dev.yaml` |
| proto3 gRPC contract for realtime streaming STT | published verbatim in the guide, saved to `grpc/` |
| `llms.txt` | served at `https://daglo.ai/llms.txt` |
| Published API unit pricing and three plans | `https://developers.daglo.ai/pricing` |
| Rate limit: 20 req/sec per endpoint | stated in the OpenAPI `info.description` |
| Webhook/callback event surface | `https://developers.daglo.ai/guide/Polling-and-Callback.html` |
| ISO 27001 certification claim | `https://daglo.ai/d/en/enterprise` |
| No MCP server, no A2A agent card, no `/.well-known/` documents | probed 2026-09-06, all misses recorded |

The contract was found on the API host root, not the docs host: `developers.daglo.ai` is a single-page
console that answers HTTP 200 with the same HTML shell for every path, while the real specification is
served by `apis.daglo.ai` and named in the Stoplight Elements loader on `/docs`.

## Two contracts that disagree

`apis.daglo.ai` serves both a production document and a larger dev-environment document, and the
provider's own guide teaches an endpoint (`POST /nlp/v1/sync/chat/completions`) that appears **only** in
the dev document. Image and video operations appear only there too. Both are saved verbatim under
`openapi/_original/` and the divergence is recorded in `lifecycle/actionpower-lifecycle.yml`.

## Gaps worth a provider's attention

- No idempotency mechanism anywhere; a retried transcription is a second billable job.
- No cancel, delete or any other reversal operation on the public write surface.
- No `Retry-After` or `RateLimit-*` response headers, so a caller hitting the documented 20 req/sec limit
  has no runtime signal to back off against.
- Callbacks carry no signature, HMAC or shared secret — a receiver cannot verify the sender.
- No status page, no deprecation policy, no `security.txt`, and one dated changelog entry from 2024-09-02.
- The only first-party API client (`actionpower/dagloapi-js-beta`) has no registry release and no version
  tag, so a consumer cannot pin it.

Everything above is assembled from public URLs; each artifact records the URL it came from and the HTTP
status that URL returned.
