# JSON Output

## Hosted MCP task outcomes

For failures or reported outcome details accompanying a pause, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. A pending refund is not a completed refund. Explain the safe reason and next step without raw stacks or internal IDs.

```json
{
  "status": "failed",
  "failure": { "stage": null, "reason": "Generation could not finish.", "retryable": null },
  "billing": { "charged": null, "refunded": null, "net": null, "refund_state": null }
}
```

For an agent video, `action_required` is a conversation waiting for the person, not a failed output. `get_video_session` exposes the agent's words and artifacts with media links; `send_video_agent_message` continues the same `task_id` when the agent is no longer running, optionally changing `auto_mode`. Initial credit consent covers continuation; no per-message approval is needed.

The CLI envelopes below are separate from the hosted MCP outcome contract.

Use `--json` when a program, agent, or shell script must consume Freebeat CLI output.

## Pattern

```bash
freebeat --json assets upload-audio --file song.mp3
freebeat --json effect create --effect-id <effectId> --music-id <musicId> --image-url <imageUrl> --prompt "anime glow"
freebeat --json task status <batch-id> --serial-no <serial-no>
```

## Success envelope

```json
{
  "success": true,
  "data": {}
}
```

## Failure envelope

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message"
  }
}
```

## Fields carried between steps

- `musicId` from audio upload.
- `imageUrls` from image upload.
- `taskId` and `taskType` from effect create or run.
- `batchId` and `items[].serialNo` from image/video generation.
- `items[].imageUrl`, `items[].editImages`, `items[].videoUrl`, `items[].errorCode`, and `items[].errorMessage` from batch status or wait.

Treat stdout as data and stderr as diagnostics. Never write raw API keys into logs or serialized artifacts.
