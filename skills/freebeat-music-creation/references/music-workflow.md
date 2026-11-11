# Music creation workflow

## MCP cost approval and outcomes

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


## Required inputs

- `model_id` — from `list_music_models`.
- `prompt` — 1 to 2000 characters describing genre, mood, tempo, and instruments.

## Optional inputs and cross-field rules

- `lyrics` (1-12000 characters): required when the chosen model has `requires_lyrics: true`; rejected together with `instrumental: true`.
- `instrumental` (default false): only models with `supports_instrumental: true` accept it.
- `duration_seconds` (10-300): only models with `supports_duration: true` accept it.

## Result fields

`get_task_result` on a completed `music:<id>` task returns `audio_url` plus optional `cover_url`, `title`, `lyrics`, `duration_ms`, and a scoped string `music_id` such as `music:123`. Pass that `music_id` straight into `generate_music_video`, `generate_dance_video`, or `generate_lyric_video` without uploading the song again.

If `supports_duration` is false, a duration mentioned in the prompt cannot control the song length. Tell the user before generation.

## Examples

MCP route:

```text
list_music_models -> generate_music {model_id, prompt: "upbeat synth-pop, driving bass, bright chorus"} -> get_task_status -> get_task_result
```

