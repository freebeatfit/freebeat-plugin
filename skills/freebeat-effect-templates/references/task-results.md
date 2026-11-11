# Effect Results

`effect run --wait` returns final result data. `effect create` and `effect run` without `--wait` return a legacy task ID.

```bash
freebeat task status <task-id>
freebeat task wait <task-id>
freebeat task result <task-id>
freebeat --json task result <task-id>
```

After `task wait` succeeds, use `task result` to retrieve `video_url` and `cover_url`.

Custom timing:

```bash
freebeat task wait <task-id> --poll-interval 30s --timeout 3h
```

On failure, report the concise `errorCode` and `errorMessage`. Keep the task ID internal unless needed for debugging or retrying.
