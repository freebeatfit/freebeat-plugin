---
version: 0.4.5
name: freebeat-task-results
description: |
  Check, wait for, and extract results from existing Freebeat tasks and batches.
  Use when: the user has a task ID, batch ID, serial number, JSON response,
  status question, timeout requirement, or wants final media URLs from an
  already submitted agent video, music, performance, effect, image, or video job. NOT for: creating media,
  choosing a model, uploading assets, or configuring credentials as the main task.
argument-hint: "<task-id-or-batch-id> [--serial-no <serial-no>]"
---

# Freebeat Task Results

Use this operational helper only after a Freebeat command has submitted work. Do not create a new media job from this skill.

## Codex plugin execution

When the `freebeat` MCP server is available, call `get_task_status`. When a task ID (not an image/video batch) reaches `completed`, `failed`, or `action_required`, call `get_task_result`; Open RPC image/video batch results are already included in completed status items. Let Codex handle browser OAuth; never request or expose an API key.

Agent videos from `generate_music_video`, `generate_dance_video`, or `generate_lyric_video` use scoped IDs starting `music_video:`, `dance:`, or `lyric_video:` and usually take 15-60 minutes; poll every few minutes. These IDs work through the plugin tools, not the CLI.

Creation requires `auto_mode`, inferred from the user's intent or discussed when needed: `true` lets the director decide and deliver; `false` pauses where taste matters. Stand where the freebeat.ai user stands: `get_video_session` shows the agent's words, stage statuses, artifacts and media links; `send_video_agent_message` speaks into that same `task_id`, optionally changing auto. `action_required` means waiting for the person, not failure. Decide when to involve the user from their intent. Wait until the agent stops running before sending; acceptance is receipt, so read its reply and check the session before resending an uncertain delivery. Initial credit consent covers continuation without per-message approval; relay any credit notice.

For failures or reported outcome details accompanying a pause, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

Use the CLI workflow below only when the MCP server is unavailable or the user explicitly asks for CLI operation.

## CLI fallback bootstrap

If `freebeat` is missing, tell the user the Freebeat CLI is not installed (they can install it themselves from https://freebeat.ai/cli) and stop; do not install software. If status calls are unauthenticated, have the user run `freebeat auth set-api-key` in their own terminal. Never request or print the raw key.

## UX rules

1. Reply in the user's language; keep statuses and JSON field names in English when quoting technical output.
2. For a final-result request, poll silently until a terminal state and return the media URLs or a concise failure reason.
3. Do not expose task IDs, batch IDs, serial numbers, config paths, polling logs, or raw JSON unless requested.
4. Continue agent videos through their existing session when appropriate; do not resubmit a new generation job from this skill.
5. Use `--json` only for automation, debugging, or explicit structured-output requests.

## Route by job type

- Legacy MV/effect task -> `task status`, `task wait`, then `task result` after success.
- Creative scoped IDs `music:<id>`, `singing:<id>`, `lip_sync:<id>` -> `get_task_status` and `get_task_result` on the MCP server; the published CLI does not read them.
- Open RPC image/video batch -> `task status` or `task wait`; include `--serial-no` to inspect one item.
- Unknown ID type -> start with `task status <id>`; the CLI routes known batch prefixes automatically.
- Creation or retry request -> return to the relevant generation skill.

## Commands

```bash
freebeat task status <task-id-or-batch-id>
freebeat task wait <task-id-or-batch-id>
freebeat task result <legacy-task-id>
freebeat task status <batch-id> --serial-no <serial-no>
freebeat task wait <batch-id> --serial-no <serial-no>
```

For machine-readable output:

```bash
freebeat --json task status <batch-id> --serial-no <serial-no>
freebeat --json task wait <batch-id> --serial-no <serial-no>
```

## Timing and terminal states

Default polling is 15 seconds and default timeout is 2 hours:

```bash
freebeat task wait <task-id-or-batch-id> --poll-interval 30s --timeout 3h
```

- On success, extract the primary `imageUrl`, `editImages`, `videoUrl`, legacy `video_url`, or `cover_url` fields.
- On failure or rejection, return `errorCode` and `errorMessage` in plain language.
- `task result` is for completed legacy MV/effect tasks, not Open RPC image/video batches.

## References

- `references/task-results.md` - ID routing, terminal statuses, polling, and result fields.
- `references/json-output.md` - structured output contract and fields carried between steps.
