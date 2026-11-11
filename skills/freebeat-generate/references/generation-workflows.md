# Generation Workflows

## MCP cost approval and outcomes

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


Every `model_id` and option value comes from `list_models` and `get_model_help` at request time. The examples below use placeholders on purpose.

## `generate_image`

- Required: `model_id` (from `list_models(type=image)`) and `prompt` (1-2000 characters).
- Optional, validated per model: `size`, `resolution`, `quality`, `output_format`, `custom_resolution`, `count` (default `1`). Send only values `get_model_help` lists for this model.

## `edit_image`

- Required: `model_id` (from `list_models(type=image_edit)`), `image` (one HTTPS image URL), and `prompt`.
- Optional: the same image options as `generate_image`.
- One source image per edit. If the user gives several, ask which one to edit.

## `generate_video`

- Required: `model_id` (from `list_models(type=video)`) and `prompt` (1-2000 characters).
- `generation_type`: `text` (default), `image`, `transition`, or `reference`. Use one the model lists in `get_model_help`.
- `images`: HTTPS image URLs; none for `text`, one for `image`, two (first, last) for `transition`, one or more for `reference`.
- Optional: `duration`, `resolution`, `aspect_ratio`, `style`, `generate_audio`, `multi_clip`, `watermark`. Valid values differ per model and per generation type; take them from the `get_model_help` options for the generation type you are using.

## `generate_text`

- Required: `model` (from `list_models(type=text)`) and `messages` (`role` is `system`, `user`, or `assistant`; oldest first).
- Optional: `max_tokens`, `temperature`.
- Returns the model's reply in `text`. Billed to the user's Freebeat credits; use only when the user explicitly asks a Freebeat text model.

## Reading a create response

- `accepted_count` and `failed_count` say how many items Freebeat took.
- When every item is rejected, `message` and `items[].message` give the reason. Change the parameter it names (recheck `get_model_help`) and retry once.
- When the reason is that the account cannot pay for the run, tell the user the account cannot run it right now. Do not suggest buying credits or show a purchase link.

## Batch results

- `get_task_status` with `task_id` = `batch_id` (add `serial_no` to read one item).
- `completed`: items carry `image_url`, `edit_images`, or `video_url`.
- `failed` or `partial_failed`: items carry `error_code` and `error_message`; report them plainly.
- Any other status is still in progress; poll again.

## Examples

```text
Image:      list_models {type: "image"} -> get_model_help {type: "image", model_id} -> generate_image {model_id, prompt, <options from help>} -> get_task_status {task_id: batch_id}
Edit:       upload_image {files: [photo]} -> list_models {type: "image_edit"} -> get_model_help -> edit_image {model_id, image: image_urls[0], prompt}
Transition: upload_image {files: [first, last]} -> list_models {type: "video"} -> get_model_help -> generate_video {model_id, prompt, generation_type: "transition", images: [first, last], <duration/resolution from help>}
Text:       list_models {type: "text"} -> generate_text {model, messages: [{role: "user", content}]}
```
