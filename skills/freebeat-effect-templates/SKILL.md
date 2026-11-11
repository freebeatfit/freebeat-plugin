---
version: 0.4.5
name: freebeat-effect-templates
description: |
  Discover Freebeat effect templates and generate template-driven clips through
  the authenticated Freebeat MCP tools, with freebeat-cli as the standalone
  fallback. Use when: the user asks for effects, effect IDs, template
  previews, template defaults, a quick effect run, prepared effect submission,
  or a completed effect video URL. NOT for: general image/video model selection,
  still-image generation, song-driven music videos, or task-only inspection.
argument-hint: "--effect-id <id> --prompt <direction> [--audio <path-or-url>] [--image <path-or-url>]"
---

# Freebeat Effect Templates

Use this specialized workflow for template-driven clips. For a general model-based image or video, use `freebeat-generate`.

## Codex plugin execution

When the `freebeat` MCP server is available, call `list_effects`, optionally `upload_audio` or `upload_image`, then `get_credit_balance` and `quote_generation` with the exact planned `generate_effect` inputs. State the list price before discounts, or that an advance price is unavailable, before calling `generate_effect`. Poll with `get_task_status` and call `get_task_result` on `completed`, `failed`, or `action_required`. Let Codex handle browser OAuth; never request or expose an API key.

Every paid call needs exactly one explicit authorization: for a known quote, send the user-approved numeric `max_credits` at or above the current quote and omit `accept_unknown_cost`; for an unavailable total, explain the uncertainty, obtain explicit consent, send `accept_unknown_cost: true`, and omit `max_credits`. Missing authorization or both modes are refused. Unknown-cost consent must never replace an already approved numeric ceiling without new user approval. Never remove or raise that ceiling to bypass refusal. Agents charge in stages, including planning and generated assets; neither mode guarantees a cap on the whole run. A quote is not an actual charge.

For `failed` or `action_required`, read `failure.stage`, `failure.reason`, and `failure.retryable` together with `billing.charged`, `billing.refunded`, `billing.net`, and `billing.refund_state`; `get_task_result` exposes the same details. Missing stage, billing, or retry information is `null`, not zero or a promised refund. Explain the safe reason and next step without raw stacks or internal IDs.

Use the CLI workflow below only when the MCP server is unavailable or the user explicitly asks for CLI operation.

## CLI fallback bootstrap

Check `freebeat --help` and `freebeat auth show`. If `freebeat` is missing, tell the user the Freebeat CLI is not installed (they can install it themselves from https://freebeat.ai/cli) and stop; do not install software. If credentials are missing, ask the user to run `freebeat auth set-api-key` in their own terminal. Never request, print, or commit the raw key.

## UX rules

1. Reply in the user's language; keep CLI flags and effect IDs in English.
2. If the user did not choose a template, list effects and select a relevant one from its actual metadata. Do not invent an effect ID.
3. Prefer `effect run --wait` for interactive work. Poll silently and return the final video URL plus cover URL when available.
4. Keep template metadata, asset IDs, task IDs, polling messages, and raw JSON out of the final response unless requested.
5. Ask one concise question only when a required asset or material creative choice is missing.

## Route by input

- Unknown effect ID -> `freebeat effects list`.
- Template defaults or local/remote overrides -> `freebeat effect run`.
- Prepared `effectId`, `musicId`, and hosted image URL -> `freebeat effect create`, then `task wait` and `task result`.
- Existing task ID only -> use `freebeat-task-results`.

`effect submit` remains a CLI alias, but use the canonical `effect create` command in new instructions.

## One-command workflow

```bash
freebeat effects list
freebeat effect run --effect-id <effectId> --prompt "anime glow" --wait
freebeat effect run --effect-id <effectId> --audio song.mp3 --image ref.png --prompt "anime glow" --wait
```

## Prepared-asset workflow

`effect create` is asynchronous and does not accept `--wait`:

```bash
freebeat effect create --effect-id <effectId> --music-id <musicId> --image-url <imageUrl> --prompt "anime glow"
freebeat task wait <task-id>
freebeat task result <task-id>
```

## Guardrails

- `effect run` resolves template defaults before applying optional audio or image overrides.
- `effect create` requires a prepared numeric effect ID, numeric music ID, one hosted image URL, and a prompt.
- Add `--watermark` only when requested.
- Use `--json` for automation that must parse effect metadata or carry task IDs between steps.
- On failure, report the concise backend message. Include the task ID only when needed for debugging or retrying.

## References

- `references/effect-workflow.md` - discovery, run/create commands, and guardrails.
- `references/assets.md` - audio and image preparation.
- `references/task-results.md` - polling and final result extraction.
