---
name: Synthesize speech from text with daglo TTS
description: Turn text into a WAV file using ActionPower's daglo text-to-speech endpoint, with the two published voices and the failure modes that return no body.
api: openapi/actionpower-daglo-cloud-api-openapi.yml
operations:
  - post-tts-v1-sync-audios
generated: '2026-09-06'
method: generated
source: openapi/actionpower-daglo-cloud-api-openapi.yml, https://developers.daglo.ai/guide/Language-Text-To-Speech.html
---

# Synthesize speech

`POST https://apis.daglo.ai/tts/v1/sync/audios` with `Authorization: Bearer <API_TOKEN>` and
`Content-Type: application/json`.

Minimal body:

```json
{ "text": "안녕하세요. 액션파워입니다." }
```

The response is **not JSON** — a `200` returns `audio/wav` bytes. Write them to a file; do not try to
parse the body. Only error responses are JSON, and they use the `{"error": "<string>"}` envelope.

## Voices

The provider publishes two:

| id | language | description |
|---|---|---|
| `en_US_Olivia` | `en_US` | Olivia — adult, female, friendly |
| `ko_KR_Jimin` | `ko_KR` | Jimin — adult, female, calm |

Do not invent voice ids; anything outside this list is undocumented.

## Failure modes worth handling

- `204` — succeeded, but produced nothing. Empty or whitespace-only text is the usual cause. Check for it
  before writing a zero-byte file.
- `413` — the text is too long for the synchronous endpoint. There is no asynchronous TTS; split the text
  and concatenate the audio yourself.
- `415` — wrong `Content-Type`.
- `403` — credits exhausted, or the account is not permitted. Neither is fixable by retrying.
- `429` — over 20 requests/sec. No `Retry-After` is returned; back off on your own schedule.

## Cost

Billed per 1,000 characters (₩15 at the published rate, VAT excluded, ₩100 minimum charge). Batch text
into fewer, longer calls where the 413 limit allows — the unit is characters, so batching does not cost
more, and it burns fewer requests against the per-second limit.
