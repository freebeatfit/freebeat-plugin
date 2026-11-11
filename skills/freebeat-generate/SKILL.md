---
version: 0.4.5
name: freebeat-generate
description: |
  Generate or edit images and create model-based video clips with any model
  Freebeat currently publishes, through the authenticated Freebeat MCP tools.
  Use when: the user asks for image generation, one-image editing, text-to-video,
  image-to-video, first/last-frame transitions, reference video, choosing or
  comparing a Freebeat model, a model's options or price, a finished media URL
  from a Freebeat model, or explicitly asks a Freebeat text model. This is the
  default entry point for model-based media. NOT for: music, lyric, or dance
  videos from a song, original music, singing-photo or lip-sync performances,
  effect-template clips, CLI setup, or checking a task that already exists.
argument-hint: "[prompt] [model name] [image attachment or URL]"
---

# Freebeat Generate

Primary skill for model-based media: images, one-image edits, and video clips from Freebeat's model catalog, plus replies from a Freebeat text model when the user asks for one.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

## The catalog is live

Freebeat adds, removes, and reprices models every week. Anything you remember about a model (name, `model_id`, options, price) may already be wrong, which is why these instructions name none. Read the catalog fresh for every request:

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

1. `list_models(type)`: `image` for `generate_image`, `image_edit` for `edit_image`, `video` for `generate_video`, `text` for `generate_text`. Media entries carry `model_id`, name, description, and availability; text entries carry `model`.
2. Pick a model. If the user named one, match it against the live names; if it is not there, say so and offer the closest live entries. Otherwise choose the entry whose description best fits the request.
3. `get_model_help(type, model_id)` for image, image_edit, and video models. It returns the options the model accepts per generation type (aspect ratios, resolutions, durations, styles) and its pricing: `pricing[].keys` are the parameters that change the price and `pricing[].prices` contains the rule body, with combination prices in `pricing[].prices.map`; duration settings and multipliers are retained. Pass only options it lists.
4. Call `get_credit_balance` and `quote_generation` with the chosen generation tool and its exact inputs. Tell the user the list price before discounts, or that an advance price is unavailable. Then generate. Freebeat validates every request per model. A rejection comes back in the response `message` and `items[].message`: read it, change the parameter it names, and retry once. If it still fails, tell the user what Freebeat said.

## Route by intent

- New image from text -> `generate_image`.
- Edit exactly one existing image -> `edit_image` with a model from `list_models(type=image_edit)`.
- Video clip -> `generate_video` with a `generation_type` the model offers in `get_model_help`:
  - `text`: no images.
  - `image`: animate one image.
  - `transition`: two images, first frame then last frame.
  - `reference`: one or more images that guide subject or style.
- The user explicitly asks a Freebeat text model (by name, or "through Freebeat") -> `generate_text` with a model from `list_models(type=text)`. It is billed to the user's Freebeat credits, so never use it for your own reasoning or for text you can write yourself.
- Music video from a song -> `freebeat-music-video`.
- Lyric video or animated lyrics from a song -> `freebeat-lyric-video`.
- Dance video from a song -> `freebeat-dance-video`.
- Original music, instrumental track, text-to-music, or lyrics-to-music -> `freebeat-music-creation`.
- Singing photo, Photo Karaoke, pet singing, duet, or lip-sync -> `freebeat-performance-video`.
- Template effect clip -> `freebeat-effect-templates`.
- Existing task or batch -> `freebeat-task-results`.
- CLI install or auth only -> point the user to https://freebeat.ai/cli.

## Results

`generate_text.usage` contains numeric token counts and defined numeric breakdowns; it is not an actual credit charge.

`generate_image`, `edit_image`, and `generate_video` return a `batch_id`. Poll `get_task_status` with `task_id` set to that `batch_id` until it is `completed`; completed items carry `image_url`, `edit_images`, or `video_url`. Do not call `get_task_result` for these batches.

## Inputs

Image inputs (`edit_image.image`, `generate_video.images`) must be HTTPS URLs. The Freebeat server runs remotely and cannot read the user's disk, so `upload_image` rejects `file_paths` there.

- ChatGPT attachment -> `upload_image.files`, then use the returned `image_urls`.
- Public HTTPS image link -> `upload_image.urls`, or pass it directly when it is already a stable HTTPS image URL.
- Local file in Codex -> upload it with the Freebeat CLI, signed in to the same Freebeat account: `freebeat --json assets upload-image <path> [<path>]` returns `imageUrls`. Attachments reach Freebeat only in ChatGPT. Without the CLI, do not install it or ask for an attachment: ask the user to share an HTTPS link, or to install the CLI from https://freebeat.ai/cli and run `freebeat auth set-api-key` in their own terminal.

## UX rules

1. Reply in the user's language; write model names exactly as `list_models` returns them.
2. Choose sensible defaults from the request; ask one question only when a required input or a materially different creative choice is missing.
3. Return the final media URL with a one-line summary. Keep batch IDs, serial numbers, polling logs, and raw JSON out unless asked.
4. Use the live `quote_generation` result for the planned call; never infer the charge from `get_model_help`. Do not switch to a cheaper model or lower quality unless the user asks.
5. Let the host handle Freebeat sign-in; never ask for or expose an API key.

## References

- `references/generation-workflows.md` - per-tool parameters, reading create responses, batch results, and examples.
- `references/media-inputs.md` - attachments, URLs, local files, and image counts by mode.
