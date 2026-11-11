---
version: 0.4.5
name: freebeat-lyric-video
description: |
  Lyric video generation: turn a song into an animated lyric video with Freebeat.
  Use when: the user asks for a lyric video generator, lyric video generation,
  animated lyrics, kinetic typography for a song, or words on screen timed to
  the vocals. NOT for: music videos without timed lyrics, dance video
  generation, writing or translating lyrics, original music, singing photos,
  lip-sync of an existing clip, or checking an existing task.
argument-hint: "<song attachment or music link> [creative direction] [aspect ratio]"
allowed-tools: Bash
---

# Freebeat Lyric Video

Use `generate_lyric_video` for animated words timed to the song. For music video generation without timed lyrics, use `freebeat-music-video`; for dancers performing to the song, use `freebeat-dance-video`.

For failures or reported outcome details accompanying a pause, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

## How the agent works

Creation requires `auto_mode`, inferred from the user's intent or discussed when needed: `true` lets the director decide and deliver; `false` pauses where taste matters. Stand where the freebeat.ai user stands: `get_video_session` shows the agent's words, stage statuses, artifacts and media links; `send_video_agent_message` speaks into that same `task_id`, optionally changing auto. `action_required` means waiting for the person, not failure. Decide when to involve the user from their intent. Wait until the agent stops running before sending; acceptance is receipt, so read its reply and check the session before resending an uncertain delivery. Initial credit consent covers continuation without per-message approval; relay any credit notice.

`generate_lyric_video` starts the same lyric-video agent a user gets on freebeat.ai. That agent is the director: it listens to the song, plans the lyric visuals, animates the words in time with the vocals, and edits the video. It treats every word of `prompt` as an instruction, so a shot list, style, or detail you add becomes something it builds. Pass the user's creative direction in the user's own words and nothing more; leave `prompt` empty when the user gave none.

A run usually takes 15-60 minutes and spends the user's Freebeat credits as it works.

## Workflow

Starting a new paid generation needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

Get the song's `music_id` from `upload_audio` or a completed `generate_music` result, and upload any supplied photos with `upload_image` (see Inputs). Read `get_credit_balance` and `quote_generation` with the exact planned `generate_lyric_video` inputs, including the required `auto_mode`. Explain the quote or unavailable total and obtain the initial credit consent above, then create with `music_id`, `auto_mode`, and only the requested options: `prompt`, `start_ms`/`end_ms`, `aspect_ratio`, `resolution`, `image_urls`, `remove_watermark`.

Handle `preparing` below before announcing a started run. Runs usually take 15-60 minutes; `session_url` is an optional viewing link. Poll `get_task_status` every few minutes while `running`, relay its latest `message`, and use the same session tools when it waits for input. After `completed`, call `get_task_result` for `result.video_url` and optional `result.cover_url`.

Generated songs can need a bounded preparation wait before video creation. If the tool returns `success: true`, `status: preparing`, a numeric `music_id`, and `retry_after_seconds`, no paid task has been created and there is no `task_id` yet. Wait that interval, then call the same generation tool with all parameters and cost authorization unchanged except replacing `music_id` with the returned number. Do not send the original `music:<id>` again or re-upload the song; that would repeat registration. Only announce a started run after a response supplies `task_id` and `session_url`.

## Song window and options

- `start_ms`/`end_ms` pick the part of the song the video covers. Default: the first 15 seconds. Standard accounts allow 10-15 seconds, VIP accounts up to 360 seconds; lyric videos need at least 4 seconds. Set them only when the user names a part of the song or a length; if Freebeat rejects the length for this account, say so plainly.
- `aspect_ratio`: `16:9` (default), `9:16`, `1:1`, `4:3`, or `3:4`. Vertical, TikTok, Reels, or Shorts means `9:16`.
- `resolution`: `768P` (default) or `2K`.
- `remove_watermark` only when the user asks.

## Inputs

The Freebeat server runs remotely and cannot read the user's disk, so it rejects `file_path` and `file_paths`.

- ChatGPT attachment -> `upload_audio.file` or `upload_image.files`.
- Original song made with `generate_music` -> use its completed result's scoped string `music_id` directly; no second upload.
- A public HTTPS link to an audio file, or a link from a music platform that `upload_audio` supports -> `upload_audio.url`. The tool description lists the platforms.
- Public HTTPS image link -> `upload_image.urls`.
- Local file in Codex -> upload it with the Freebeat CLI, signed in to the same Freebeat account. `freebeat --json assets upload-audio --file <path>` returns `musicId`; pass it as `music_id`. `freebeat --json assets upload-image <path> [<path>]` returns `imageUrls`; pass them as `image_urls`. Attachments reach Freebeat only in ChatGPT. If the CLI is not set up, do not install it or ask for an attachment: tell the user to share a public link to the audio file or a supported music page, or to install the CLI from https://freebeat.ai/cli and run `freebeat auth set-api-key` in their own terminal.
- Never use placeholder or synthetic audio for a real request.

## UX rules

1. Reply in the user's language; keep tool and field names in English.
2. Ask when the song is missing or an ambiguity changes the work; involve the user in creative choices according to their intent.
3. Keep `music_id`, `task_id`, polling chatter, and raw JSON out of replies unless asked. `session_url` is an optional viewing link; continue the conversation through the tools.
4. On an error, report Freebeat's message in plain words.
5. Let the host handle Freebeat sign-in; never ask for or expose an API key.

## References

- `references/lyric-video-workflow.md` - `generate_lyric_video` fields, statuses, results, and examples for lyric video generation.
