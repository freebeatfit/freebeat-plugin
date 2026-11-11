---
version: 0.4.5
name: freebeat-music-creation
description: |
  Generate original music from a text prompt, with optional lyrics, through the
  authenticated Freebeat MCP tools.
  Use when: the user asks to generate music, an instrumental track, text-to-music,
  lyrics-to-music, a song from a description, or background music. NOT for: music
  videos, dance videos, singing-photo or lip-sync performances, still images, or
  inspecting an existing task without creating music.
argument-hint: "--prompt <description> [--lyrics <lyrics>] [--instrumental]"
---

# Freebeat Music Creation

Use this skill when the user wants an original music track. If the user wants a video set to music, route to `freebeat-music-video`, `freebeat-dance-video`, `freebeat-lyric-video`, or `freebeat-performance-video` instead.

## Codex plugin execution

When the `freebeat` MCP server is available, call `list_music_models`, then `generate_music`. Poll with `get_task_status` and call `get_task_result` after completion; the result exposes `audio_url` and, when Freebeat provides it, `music_id` for video tools. Let Codex handle browser OAuth; never request or expose an API key.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

If the `freebeat` MCP server is unavailable, say so and offer to continue once it is connected.

## Workflow

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

1. Call `list_music_models` and pick the model whose capabilities match the request: `requires_lyrics` models need lyrics; `supports_duration` models accept a custom length; `supports_instrumental` models can skip vocals.
2. If the model has `supports_duration: false`, tell the user that it cannot control the song length. Call `get_credit_balance` and `quote_generation` with the exact planned `generate_music` inputs; state the list price before discounts, or that an advance price is unavailable. Then call `generate_music`.
3. Poll `get_task_status`, then return the final `audio_url` from `get_task_result` with a one-line summary. If the user also wants a video, pass the returned `music_id` directly to the corresponding video tool.

## UX rules

1. Reply in the user's language; keep tool and field names in English.
2. Ask one concise question only when the request is missing an essential creative direction.
3. State the balance and advance price when available; do not guess when the backend cannot quote the request.
4. Keep task IDs and raw JSON out of the reply unless requested.

## References

- `references/music-workflow.md` - inputs, cross-field rules, result fields, and copyable examples.
