# Video Workflows

## MCP cost approval and outcomes

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


Every `model_id` and option value comes from `list_models` and `get_model_help` at request time. The examples below use placeholders on purpose.

## `generate_video` parameters

- Required: `model_id` (from `list_models(type=video)`) and `prompt` (1-2000 characters).
- `generation_type`: `text` (default), `image`, `transition`, or `reference`.
- `images`: HTTPS image URLs.
- Optional: `duration` (seconds), `resolution`, `aspect_ratio`, `style`, `generate_audio`, `multi_clip`, `watermark`.

## Generation types

| `generation_type` | `images` |
|---|---|
| `text` | none |
| `image` | exactly one |
| `transition` | exactly two, first frame then last frame |
| `reference` | one or more; use the fewest the request needs |

`get_model_help` reports options per generation type. A model may offer a resolution or duration for one type and not another, and may not offer a type at all. Check the type first, then the duration and resolution for that type. Send `style`, `generate_audio`, or `multi_clip` only when the model's help shows them.

## Rejections

A create response with every item rejected carries the reason in `message` and `items[].message`. Change the parameter it names and retry once. If the reason is that the account cannot pay for the run, tell the user the account cannot run it right now; do not suggest buying credits or show a purchase link.

## Results

`get_task_status` with `task_id` = `batch_id` (add `serial_no` for one item):

- `completed`: items carry `video_url`.
- `failed` or `partial_failed`: items carry `error_code` and `error_message`.
- Anything else is still in progress; poll again.

## Examples

```text
Text:       list_models {type: "video"} -> get_model_help {type: "video", model_id} -> generate_video {model_id, prompt, generation_type: "text", <duration/resolution/aspect_ratio from help>}
Image:      upload_image {files: [frame]} -> generate_video {model_id, prompt, generation_type: "image", images: [image_urls[0]], ...}
Transition: upload_image {files: [first, last]} -> generate_video {model_id, prompt, generation_type: "transition", images: [first, last], ...}
Reference:  upload_image {files: [subject]} -> generate_video {model_id, prompt, generation_type: "reference", images: [subject], ...}
Then:       get_task_status {task_id: batch_id} until completed -> items[].video_url
```
