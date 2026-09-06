---
name: Summarize a conversation and generate meeting minutes with daglo
description: Use ActionPower's daglo NLP endpoints to summarize a short dialogue synchronously, split raw transcript text into paragraphs, and request asynchronous meeting minutes.
api: openapi/actionpower-daglo-cloud-api-openapi.yml
operations:
  - post-nlp-v1-sync-summaries
  - post-nlp-v1-sync-paragraphs
  - post-nlp-v1-async-minutes
  - get-nlp-v1-async-minutes-rid
generated: '2026-09-06'
method: generated
source: openapi/actionpower-daglo-cloud-api-openapi.yml, https://developers.daglo.ai/guide/en/
---

# Summarize a conversation, and turn it into minutes

Base URL `https://apis.daglo.ai`, bearer token in `Authorization`. These endpoints take **text**, so the
usual pipeline is: transcribe first (see the long-audio skill), then feed `sttResults[].transcript` here.

## Clean up the text — `post-nlp-v1-sync-paragraphs`

`POST /nlp/v1/sync/paragraphs`. Splits a wall of transcript into paragraphs. Synchronous; the result
comes back in the response. Do this before summarizing long transcripts — it is what the provider's own
product does.

## Summarize a short dialogue — `post-nlp-v1-sync-summaries`

`POST /nlp/v1/sync/summaries`. Synchronous, so it is bounded by payload size: oversized input returns
`413`, not a truncated summary. `nlpConfig` accepts a `summary.model` selection and a
`keywordExtraction.enable` flag.

## Full meeting minutes — `post-nlp-v1-async-minutes` then `get-nlp-v1-async-minutes-rid`

`POST /nlp/v1/async/minutes` returns an `rid`; `GET /nlp/v1/async/minutes/{rid}` returns the minutes.
Same asynchronous contract as transcription — a `callback: {url, headers}` object can replace polling,
and the same status values apply.

**The rids are not typed.** A transcription rid and a minutes rid are both bare UUIDs with no prefix, and
polling the wrong endpoint returns `404`, not a helpful error. Track which endpoint issued each rid.

## Language caveat

Async transcription supports Korean, English, Japanese and mixed Korean/English, but the provider states
plainly in its FAQ that **summary and the add-on features are Korean only**. Do not promise an English
summary; transcribe in English and summarize elsewhere, or expect degraded output.

## Cost shape

Minutes summary is billed per call (₩90 at the published rate), not per token, so batching a conversation
into one call is materially cheaper than iterating. Chat completion is billed per million tokens. Both
exclude VAT, and the minimum charge is ₩100.
