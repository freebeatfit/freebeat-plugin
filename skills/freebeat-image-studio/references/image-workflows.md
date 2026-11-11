# Image Workflows

## MCP cost approval and outcomes

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


Every `model_id` and option value comes from `list_models` and `get_model_help` at request time. The examples below use placeholders on purpose.

## Generate

- Required: `model_id` (from `list_models(type=image)`) and `prompt` (1-2000 characters).
- Optional, validated per model: `size`, `resolution`, `quality`, `output_format`, `custom_resolution`, `count` (default `1`). Send only values `get_model_help(type=image, model_id)` lists. Do not copy options from one model to another.

## Edit one image

- Required: `model_id` (from `list_models(type=image_edit)`), `image` (one HTTPS image URL), and `prompt`.
- Optional: the same options, validated against `get_model_help(type=image_edit, model_id)`.

## Rejections

A create response with every item rejected carries the reason in `message` and `items[].message`. Change the parameter it names and retry once. If the reason is that the account cannot pay for the run, tell the user the account cannot run it right now; do not suggest buying credits or show a purchase link.

## Results

`get_task_status` with `task_id` = `batch_id`:

- `completed`: items carry `image_url` (generate) or `edit_images` (edit).
- `failed` or `partial_failed`: items carry `error_code` and `error_message`.
- Anything else is still in progress; poll again.

## Examples

```text
list_models {type: "image"} -> get_model_help {type: "image", model_id} -> generate_image {model_id, prompt, <options from help>} -> get_task_status {task_id: batch_id}
upload_image {files: [photo]} -> list_models {type: "image_edit"} -> get_model_help {type: "image_edit", model_id} -> edit_image {model_id, image: image_urls[0], prompt} -> get_task_status {task_id: batch_id}
```
