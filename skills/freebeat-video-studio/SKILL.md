---
version: 0.4.5
name: freebeat-video-studio
description: |
  Create model-based video clips from text, one image, first/last frames, or
  reference images with any video model Freebeat currently publishes, through
  the authenticated Freebeat MCP tools. Use when: the user explicitly wants
  text-to-video, image-to-video, a transition, reference video, aspect-ratio
  variants, generated audio, styles, or multi-clip output. NOT for: music, lyric,
  or dance videos from a song, effect-template clips, still-image editing, or
  checking a task that already exists.
argument-hint: "[prompt] [model name] [image attachments or URLs]"
---

# Freebeat Video Studio

Use this focused skill for model-based video clips. For an ambiguous image-or-video request, use `freebeat-generate` as the primary router. For a video built around a song, use `freebeat-music-video`, `freebeat-dance-video`, or `freebeat-lyric-video`.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

## The catalog is live

Freebeat adds, removes, and reprices video models every week, so anything you remember about a model may be wrong. Read it fresh for every request:

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

1. `list_models(type=video)`. Each entry carries `model_id`, name, description, and availability.
2. Pick the model the user named (match the live names; if it is absent, say so and offer the closest live entries), otherwise the entry whose description best fits the request.
3. `get_model_help(type=video, model_id)` lists, per generation type, the aspect ratios, resolutions, durations, and styles the model accepts, plus pricing (`pricing[].keys` are the parameters that change the price; `pricing[].prices` is the rule body, combination tables are in `pricing[].prices.map`, and duration settings and multipliers are retained). Confirm the model offers the generation type you need, then pass only options listed for that type.
4. Call `get_credit_balance` and `quote_generation` with the planned `generate_video` inputs. State the list price before discounts, or that an advance price is unavailable. Then submit. If Freebeat rejects the request, `message` and `items[].message` say why: change that parameter and retry once, then report Freebeat's message if it still fails.

## Pick the generation type

- `text`: prompt only; no source image.
- `image`: animate exactly one image.
- `transition`: exactly two images, first frame then last frame.
- `reference`: one or more images guiding subject or style; pass only the images the request needs.

Call `generate_video(model_id, prompt, generation_type, images, ...)`. It returns a `batch_id`; poll `get_task_status` with `task_id` set to it until `completed`, then return the item `video_url`. Do not call `get_task_result` for video batches.

## Inputs

`generate_video.images` accepts HTTPS image URLs only. The Freebeat server runs remotely and cannot read the user's disk, so `upload_image` rejects `file_paths` there.

- ChatGPT attachment -> `upload_image.files` (one or two per call), then use the returned `image_urls`.
- Public HTTPS image link -> `upload_image.urls`, or pass it directly when it is already a stable HTTPS image URL.
- Local file in Codex -> `freebeat --json assets upload-image <path> [<path>]` with the Freebeat CLI, signed in to the same Freebeat account; it returns `imageUrls`. Attachments reach Freebeat only in ChatGPT. Without the CLI, do not install it or ask for an attachment: ask the user to share an HTTPS link, or to install the CLI from https://freebeat.ai/cli and run `freebeat auth set-api-key` in their own terminal.

## UX rules

1. Reply in the user's language; write model names exactly as `list_models` returns them.
2. Infer the generation type from the supplied media. Ask one question only when a required image or an essential choice is missing.
3. Return the final video URL with a short summary. Keep batch IDs, serial numbers, polling messages, and raw JSON out unless asked.
4. Use the live `quote_generation` result for the planned call; never infer the charge from `get_model_help`. Do not switch to a cheaper model or lower quality unless the user asks for speed or budget.
5. Let the host handle Freebeat sign-in; never ask for or expose an API key.

## References

- `references/video-workflows.md` - parameters per generation type, rejection handling, results, and examples.
