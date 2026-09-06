---
name: Stream realtime speech to text with daglo over gRPC
description: Open a bidirectional gRPC stream to ActionPower's daglo realtime STT service, send LINEAR16 audio, and consume interim and final transcripts — including the permission gate that blocks most accounts.
api: grpc/actionpower-speech.proto
operations:
  - dagloapis.speech.v1.Speech/StreamingRecognize
generated: '2026-09-06'
method: generated
source: grpc/actionpower-speech.proto, https://developers.daglo.ai/guide/Quick-Realtime-Voice-To-Text.html
---

# Realtime streaming transcription

This surface is **gRPC, not REST**. There is no HTTP endpoint and no OpenAPI operation for it; the
contract is the published proto3 file saved at `grpc/actionpower-speech.proto`.

## Check the gate first

The provider's FAQ states: *"Stream STT is only available to customers who have additional permissions."*
A self-serve account with a valid token will still be refused. Ask `api-support@daglo.ai` before building
against it — this is a human step no agent can complete on its own.

## Connect

- Server: `apis.daglo.ai` (TLS, `grpc.ssl_channel_credentials()`).
- Service: `dagloapis.speech.v1.Speech`, method `StreamingRecognize` — bidirectional streaming.
- Auth: the same bearer token, sent as gRPC **metadata**, not an HTTP header:
  `("authorization", "Bearer <API_TOKEN>")`.

Generate stubs from the published proto (`python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. ./speech.proto`).
There is no first-party Python SDK; the only first-party client is a browser JavaScript library at
`https://github.com/actionpower/dagloapi-js-beta`, which has no npm release and no version tag.

## The message contract, and the one ordering rule that breaks everything

`StreamingRecognizeRequest` is a `oneof`:

1. **The first message must carry `config` and must not carry `audio_content`.**
2. **Every message after that must carry `audio_content` and must not carry `config`.**
3. Send an empty `StreamingRecognizeRequest()` to signal end of stream.

`RecognitionConfig`:

- `language_code` — `ko-KR` (default), `en-US`, or `mixed`. Realtime supports **only** these three,
  unlike async STT.
- `interim_results` — `true` to receive partial results flagged `is_final=false`.
- `max_silence_ms` — `0`..`10000`; flush accumulated words as final after this much silence.

## The audio format is not negotiable

LINEAR16 encoding, **16000 Hz**, **mono** (1 channel). Resample before sending. A 0.25s chunk
(`16000 * 0.25` frames) is the size the provider's own example uses.

## Consume responses

`StreamingRecognizeResponse.result` is a `StreamingRecognitionResult`:

- `transcript` — the text. In space-separated languages it may begin with a leading space; concatenating
  results without a separator reconstructs the full transcript.
- `is_final` — `false` means this text **can still change**; only render it as provisional.
- `language_code` — the language actually detected.

`total_duration` on the response is the running audio duration in seconds — the billing unit. Realtime
STT is priced per minute like sync and async STT.

## Limits

- One request stream may run up to **6 hours**.
- Songs and audio with loud background music are explicitly unsupported.
- Errors arrive as `grpc.RpcError` with a code and details, not as the REST `{"error": ...}` envelope.
