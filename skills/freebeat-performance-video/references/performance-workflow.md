# Performance video workflows

## MCP cost approval and outcomes

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


## Singing video (`generate_singing_video`)

Required inputs:

- `mode` — `photo`, `pet`, or `duet`.
- `image_urls` — exactly one public image URL for photo/pet, exactly two for duet (from `upload_image`).
- Exactly one audio source: `music_id` (from `upload_audio`) or `audio_url`.
- `duration_seconds` — 1 to 30.

Optional: `audio_start_seconds` (default 0), `aspect_ratio` (`9:16`, `16:9`, `1:1`), `scene_prompt` (up to 230 characters), `remove_watermark`.

Result: a completed `singing:<id>` task returns `video_url`.

Examples, one per mode:

```text
photo: upload_audio + upload_image -> generate_singing_video {mode: "photo", image_urls: [portrait], music_id, duration_seconds: 15}
pet:   upload_audio + upload_image -> generate_singing_video {mode: "pet", image_urls: [pet_photo], audio_url, duration_seconds: 10}
duet:  upload_audio + upload_image -> generate_singing_video {mode: "duet", image_urls: [a, b], music_id, duration_seconds: 20}
```

## Lip-sync video (`generate_lip_sync_video`)

Required inputs:

- `video_url` — HTTPS URL from `upload_video` (MP4, MOV, or WebM up to 500 MB).
- `video_duration_ms` — positive integer.
- Exactly one audio source: `music_id` or `audio_url`; `audio_duration_ms` is required with `audio_url`.

Optional: `name`, `remove_watermark`.

Result: a completed `lip_sync:<id>` task returns `video_url` plus optional `cover_url` and `duration_ms`.

Example:

```text
upload_video + upload_audio -> generate_lip_sync_video {video_url, video_duration_ms: 12000, music_id} -> get_task_status -> get_task_result
```
