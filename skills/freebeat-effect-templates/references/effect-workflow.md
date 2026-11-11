# Effect Workflow

## MCP cost approval and outcomes

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.


## Discover templates

```bash
freebeat effects list
freebeat --json effects list
```

Use structured output only when another process must parse effect IDs, names, preview media, defaults, or aspect ratios. Never invent an effect ID.

## Run with defaults or simple overrides

```bash
freebeat effect run --effect-id <effectId> --prompt "anime glow" --wait
freebeat effect run --effect-id <effectId> --audio song.mp3 --image ref.png --prompt "anime glow" --wait
freebeat effect run --effect-id <effectId> --audio https://example.com/song.mp3 --image https://cdn.example.com/ref.png --prompt "anime glow" --wait
```

`effect run` loads the selected template, applies optional audio or image overrides, creates the task, and can wait in one command.

## Create from prepared assets

```bash
freebeat effect create --effect-id <effectId> --music-id <musicId> --image-url <imageUrl> --prompt "anime glow"
freebeat --json effect create --effect-id <effectId> --music-id <musicId> --image-url <imageUrl> --prompt "anime glow"
```

`effect create` requires all assets, is asynchronous, and does not accept `--wait`:

```bash
freebeat task wait <task-id>
freebeat task result <task-id>
```

`effect submit` is a backward-compatible alias. Prefer `effect create` in new instructions.

Add `--watermark` only when requested.
