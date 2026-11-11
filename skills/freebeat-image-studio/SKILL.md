---
version: 0.4.5
name: freebeat-image-studio
description: |
  Create still images or edit exactly one image with any image model Freebeat
  currently publishes, through the authenticated Freebeat MCP tools.
  Use when: the user explicitly wants an image, album cover, campaign visual,
  product image, poster, typography, image variation, restyle, or prompt-guided
  edit and needs focused image-model guidance. NOT for: video generation,
  multi-image compositing, song-driven music videos, or effect-template clips.
argument-hint: "[prompt] [model name] [image attachment or URL]"
---

# Freebeat Image Studio

Use this focused skill for still-image work. For a request that may become either an image or a video, use `freebeat-generate` as the primary router.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

## The catalog is live

Freebeat adds, removes, and reprices image models every week, so anything you remember about a model may be wrong. Read it fresh for every request:

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

1. `list_models(type=image)` for new images, or `list_models(type=image_edit)` for edits. Each entry carries `model_id`, name, description, and availability.
2. Pick the model the user named (match the live names; if it is absent, say so and offer the closest live entries), otherwise the entry whose description best fits the request.
3. `get_model_help(type, model_id)` lists the sizes, resolutions, qualities, output formats, counts, and pricing this model accepts. Pass only what it lists.
4. Call `get_credit_balance` and `quote_generation` with the planned generation tool and its exact inputs. State the list price before discounts, or that an advance price is unavailable. Then submit. If Freebeat rejects the request, `message` and `items[].message` say why: change that parameter and retry once, then report Freebeat's message if it still fails.

## Route by input

- No source image -> `generate_image(model_id, prompt, ...)`.
- Exactly one source image -> `edit_image(model_id, image, prompt, ...)`.
- More than one source image -> editing takes one image; ask the user which one, or route to `freebeat-generate` if they want a video.
- Unsure whether the output should be an image or a video -> `freebeat-generate`.

Both tools return a `batch_id`. Poll `get_task_status` with `task_id` set to it until `completed`, then return the item `image_url` or `edit_images`. Do not call `get_task_result` for image batches.

## Inputs

`edit_image.image` must be an HTTPS URL. The Freebeat server runs remotely and cannot read the user's disk, so `upload_image` rejects `file_paths` there.

- ChatGPT attachment -> `upload_image.files`, then use `image_urls[0]`.
- Public HTTPS image link -> `upload_image.urls`, or pass it directly when it is already a stable HTTPS image URL.
- Local file in Codex -> `freebeat --json assets upload-image <path>` with the Freebeat CLI, signed in to the same Freebeat account; it returns `imageUrls`. Attachments reach Freebeat only in ChatGPT. Without the CLI, do not install it or ask for an attachment: ask the user to share an HTTPS link, or to install the CLI from https://freebeat.ai/cli and run `freebeat auth set-api-key` in their own terminal.

## UX rules

1. Reply in the user's language; write model names exactly as `list_models` returns them.
2. Choose sensible defaults from the request. Ask one question only when the source image or an essential creative choice is missing.
3. Return the final image URL with a short description. Keep batch IDs and raw JSON out unless asked.
4. Do not promise exact text, identity, or product fidelity beyond what the model's description and help support.
5. Use the live `quote_generation` result for the planned call; never infer the charge from `get_model_help`. Let the host handle Freebeat sign-in; never ask for or expose an API key.

## References

- `references/image-workflows.md` - generate and edit parameters, rejection handling, results, and examples.
