# Task Results Reference

## MCP outcome contract

For failures or reported outcome details accompanying a pause, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. A pending refund is not a completed refund. Explain the safe reason and next step without raw stacks or internal IDs.

The plugin supports `get_task_result` on `completed`, `failed`, and `action_required`. Agent video assembly still in progress can return `running` with a message to poll again; other running tasks return `TASK_NOT_COMPLETED`. Image/video batch URLs remain in `get_task_status` items.

Creation requires `auto_mode`, inferred from the user's intent or discussed when needed: `true` lets the director decide and deliver; `false` pauses where taste matters. Stand where the freebeat.ai user stands: `get_video_session` shows the agent's words, stage statuses, artifacts and media links; `send_video_agent_message` speaks into that same `task_id`, optionally changing auto. `action_required` means waiting for the person, not failure. Decide when to involve the user from their intent. Wait until the agent stops running before sending; acceptance is receipt, so read its reply and check the session before resending an uncertain delivery. Initial credit consent covers continuation without per-message approval; relay any credit notice.

## Commands

```bash
freebeat task status <task-id-or-batch-id>
freebeat task wait <task-id-or-batch-id>
freebeat task result <legacy-task-id>
freebeat task status <batch-id> --serial-no <serial-no>
freebeat task wait <batch-id> --serial-no <serial-no>
```

## Routing

- Legacy MV/effect task: `task status`, `task wait`, then `task result` after success.
- Open RPC image/video batch: `task status` or `task wait`; add `--serial-no` for one item.
- Batch IDs beginning with `OAI_BATCH_` or `OB` are routed to batch queries automatically.
- Any status or wait call with `--serial-no` is treated as an Open RPC batch query.
- `task result` is legacy-only.

## Procedure

1. Use `task status` for a non-blocking check.
2. Use `task wait` when the user wants a terminal result.
3. For successful legacy MV/effect tasks, run `task result` to retrieve final URLs.
4. For successful image/video batches, extract URLs directly from the batch response.
5. On failure, return the concise error code and message without dumping the full response.

## Timing

Default polling is 15 seconds and default timeout is 2 hours:

```bash
freebeat task wait <task-id-or-batch-id> --poll-interval 30s --timeout 3h
freebeat task wait <batch-id> --serial-no <serial-no> --poll-interval 30s --timeout 3h
```

Terminal statuses include `success`, `succeeded`, `completed`, `failed`, `failure`, and `rejected`.
