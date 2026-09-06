---
name: Transcribe long audio with the daglo Cloud API
description: Submit an audio or video file up to 2GB / 4 hours to ActionPower's daglo Cloud API, wait for the asynchronous job to finish, and read the transcript — including the failure states that arrive as HTTP 200.
api: openapi/actionpower-daglo-cloud-api-openapi.yml
operations:
  - post-stt-v1-async-transcripts
  - get-stt-v1-async-transctips-rid
  - get-stt-v1-async-transctips-rid-format
generated: '2026-09-06'
method: generated
source: openapi/actionpower-daglo-cloud-api-openapi.yml, https://developers.daglo.ai/guide/en/STT-Async.html
---

# Transcribe long audio

Base URL `https://apis.daglo.ai`. Auth is one account-wide bearer token issued from the Token menu of
the daglo Developers console: `Authorization: Bearer <API_TOKEN>`. There are no scopes.

## 1. Submit the job — `post-stt-v1-async-transcripts`

`POST /stt/v1/async/transcripts`

Two body shapes are accepted and they are not interchangeable:

- `multipart/form-data` with a `file` part, to upload the media directly.
- `application/json` with `audio.source.url`, to point at a publicly reachable URL.

Limits the provider publishes: 2GB, 4 hours, and a fixed list of audio/video containers. A file whose
extension matches but whose encoding does not will fail. Over the limit is `413`; an unsupported
container is `415`.

Useful `sttConfig` options, all optional:

- `language` — `ko-KR` (default), `en-US`, `ja-JP`, `mixed` for Korean/English code-switching, plus
  further BCP 47 tags in the enum.
- `speakerDiarization` — separate speakers.
- `keywordBoost` — `{enable: true, keywords: [...], boost: 1..15}` to raise recognition odds for domain terms.
- `multiChannel` — up to 2 channels transcribed separately.

Set `custom` to any object you want echoed back verbatim in the response and the callback; it is the
only correlation handle the API gives you, and it is not a dedupe key.

The response is `{"rid": "<uuid>", "fileName": "...", "custom": {...}}`. **Save the rid** — there is no
way to list your jobs and no way to look one up by filename.

## 2. Wait — poll or register a callback

Poll with `get-stt-v1-async-transctips-rid` (`GET /stt/v1/async/transcripts/{rid}`), or supply
`callback: {url, headers}` on the submit and let the provider POST progress and the result to you.
Callbacks carry **no signature** — the receiver can only authenticate the sender by headers it supplied
itself, so treat an unauthenticated callback as untrusted. Use `callback.url`, not the deprecated
`callbackUrl` field.

## 3. Read the outcome — and do not treat 200 as success

The polling response is `200` for both success and failure; the truth is in `status`:

| status | meaning | what to do |
|---|---|---|
| `ai_requested`, `uploaded`, `file_processing`, `transcribing`, `post_processing` | still working | keep waiting |
| `transcribed` | done | read `sttResults[].transcript` and `sttResults[].words[]` |
| `transcript_error` | transcription failed | wait, then re-submit |
| `file_error` | the input file is bad | fix the file, then re-submit |

A `204` means the job succeeded but produced an empty transcript. That is a legitimate result for silent
audio, not an error.

Word timings are objects, not floats: `startTime: {seconds: "0", nanos: 60000000}`.

## 4. Export a subtitle or text file — `get-stt-v1-async-transctips-rid-format`

`GET /stt/v1/async/transcripts/{rid}/{format}` returns `text/plain`. A `202` here means the job is still
running — poll again rather than treating it as an empty file.

## Rules this API imposes

- **No idempotency.** There is no `Idempotency-Key`. A retry after a timeout starts a second billable
  job. Persist the rid before you retry, and prefer polling the existing rid over re-submitting.
- **No cancel, no delete.** Once submitted, a job cannot be stopped and its result cannot be removed
  through the API. Confirm before you submit; you cannot take it back.
- **20 requests/sec per endpoint**, `429` on exceed, and **no `Retry-After` or `RateLimit-*` header** —
  choose your own back-off (exponential from ~1s is reasonable) and do not hammer, because sustained
  over-limit traffic is treated as abuse and blocked.
- **`403` is ambiguous**: either the account's credits are exhausted or the surface needs an extra grant.
  Both need a human at `api-support@daglo.ai`; do not retry into it.
- Errors are `{"error": "<string>"}` — free text, no code. Branch on the HTTP status, not the message.
