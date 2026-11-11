---
version: 0.4.5
name: freebeat-performance-video
description: |
  Animate a still portrait, pet photo, or two portraits into a short single-shot
  singing performance, or lip-sync an uploaded video to a track, through the
  authenticated Freebeat MCP tools.
  Use when: the user asks for a singing photo, singing image, Photo Karaoke, a
  pet singing video, a duet video from two photos, or a lip-sync video. NOT for:
  full music videos, dance videos, original music generation, or inspecting an
  existing task without creating media.
argument-hint: "--mode <photo|pet|duet|lip-sync> --image <url> --audio <path-or-url>"
---

# Freebeat Performance Video

Use this skill for short singing-photo performances (photo, pet, or duet) and lip-sync videos.

## Singing photo versus full music video

- Use `generate_singing_video` for a still portrait/photo/pet, or two portraits for duet, animated into a 1-30 second single-shot performance.
- Use `generate_music_video` (the photo in `image_urls`) for a full music-video request, a multi-scene character performance, or a request explicitly described as a music video — that route belongs to `freebeat-music-video`.
- If the user only says "make this sing" and the intended source/output is unclear, ask whether they want a short singing-photo performance or a full music video before calling a generation tool.

## Codex plugin execution

When the `freebeat` MCP server is available:

- Singing route: `upload_audio` and `upload_image`, then `generate_singing_video` with `mode: photo|pet|duet`. Photo and pet take exactly one image; duet takes exactly two.
- Lip-sync route: `upload_video` and `upload_audio`, then `generate_lip_sync_video` with the video URL, its duration in milliseconds, and exactly one audio source.

Before either generation call, use `get_credit_balance` and `quote_generation` with the exact planned tool and inputs. State the list price before discounts, or that an advance price is unavailable. Poll with `get_task_status` and fetch the final `video_url` with `get_task_result`. Let Codex handle browser OAuth; never request or expose an API key.

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

## Inputs

The Freebeat server runs remotely and cannot read the user's disk, so it rejects `file_path` and `file_paths`.

- ChatGPT attachment -> `upload_audio.file`, `upload_image.files`, or `upload_video.file`.
- A link from a music platform that `upload_audio` supports -> `upload_audio.url`; a direct HTTPS audio file can go straight into `audio_url` instead of `music_id`.
- Public HTTPS image or video link -> `upload_image.urls` or `upload_video.url`.
- Local file in Codex -> upload it with the Freebeat CLI, signed in to the same Freebeat account: `freebeat --json assets upload-audio --file <path>` returns `musicId` (use as `music_id`) and `freebeat --json assets upload-image <path> [<path>]` returns `imageUrls`. A local clip needs a public HTTPS link for `upload_video.url`; the CLI does not upload video. Attachments reach Freebeat only in ChatGPT. If the CLI is not set up, do not install it or ask for an attachment: tell the user to share a public link to the audio file or a supported music page, or to install the CLI from https://freebeat.ai/cli and run `freebeat auth set-api-key` in their own terminal.

## UX rules

1. Reply in the user's language; keep tool and field names in English.
2. Ask for the missing image, audio source, or duration instead of guessing; duet always needs two images.
3. State the balance and advance price when available; do not guess when the backend cannot quote the request.
4. Keep asset IDs, task IDs, and raw JSON out of the reply unless requested.

## References

- `references/performance-workflow.md` - inputs, cross-field rules, result fields, and one example per mode.
