# Lyric Video Workflow

## MCP cost approval and outcomes

Starting a new paid generation needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For failures or reported outcome details accompanying a pause, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


## `generate_lyric_video` fields

| Field | Required | Values |
|---|---|---|
| `auto_mode` | yes | Boolean chosen from the user's request: `true` delegates creative decisions; `false` co-creates when the agent pauses. |
| `music_id` | yes | From `upload_audio`, `musicId` from the CLI audio upload, a completed generated song (`music:<id>`), or the numeric ID returned by `preparing` |
| `prompt` | no | The user's creative direction in their own words, up to 2000 characters. Empty lets the agent decide. |
| `start_ms` / `end_ms` | no | Song window in milliseconds. Default `0` to `15000`; `end_ms` must be after `start_ms`. Standard accounts 10-15 s, VIP up to 360 s; lyric videos need at least 4 s. |
| `aspect_ratio` | no | `16:9` (default), `9:16`, `1:1`, `4:3`, `3:4` |
| `resolution` | no | `768P` (default) or `2K`. |
| `image_urls` | no | Up to 5 photo URLs of people or characters to feature, from `upload_image` (one or two per upload call) |
| `remove_watermark` | no | `false` by default |

Once creation starts, the response carries `task_id`, `status`, and an optional viewing link in `session_url`.

Creation requires `auto_mode`, inferred from the user's intent or discussed when needed: `true` lets the director decide and deliver; `false` pauses where taste matters. Stand where the freebeat.ai user stands: `get_video_session` shows the agent's words, stage statuses, artifacts and media links; `send_video_agent_message` speaks into that same `task_id`, optionally changing auto. `action_required` means waiting for the person, not failure. Decide when to involve the user from their intent. Wait until the agent stops running before sending; acceptance is receipt, so read its reply and check the session before resending an uncertain delivery. Initial credit consent covers continuation without per-message approval; relay any credit notice.

Generated songs can need a bounded preparation wait before video creation. If the tool returns `success: true`, `status: preparing`, a numeric `music_id`, and `retry_after_seconds`, no paid task has been created and there is no `task_id` yet. Wait that interval, then call the same generation tool with all parameters and cost authorization unchanged except replacing `music_id` with the returned number. Do not send the original `music:<id>` again or re-upload the song; that would repeat registration. Only announce a started run after a response supplies `task_id` and `session_url`.

## Statuses

| `get_task_status` | Meaning | Next step |
|---|---|---|
| `running` | The agent is working. | Relay the latest `message` and poll again in a few minutes; wait before sending a message. |
| `completed` | The final video is ready. | `get_task_result` -> `result.video_url`, `result.cover_url` |
| `failed` | Generation stopped. | Read `failure.reason` and `billing`, and explain the safe next step. |
| `action_required` | The agent is waiting for the person, not a failure. | Read `get_video_session` and continue through `send_video_agent_message` according to the user's intent. |

`get_task_result` preserves reported `failure` and `billing` for `failed` or `action_required`; an agent video still assembling can return `running` with a message to poll again. Continue a paused session using its existing `task_id` rather than starting a replacement.

## Examples

```text
"Make an animated lyric video for this song, with words timed to the vocals." (song attached)
upload_audio {file} -> generate_lyric_video {auto_mode: true, music_id} -> get_task_status ... -> get_task_result

"Vertical lyric video of the chorus from 0:45 to 1:00, dreamy pastel vibe."
upload_audio {file} -> generate_lyric_video {auto_mode: true, music_id, prompt: "dreamy pastel vibe", start_ms: 45000, end_ms: 60000, aspect_ratio: "9:16"}
```
