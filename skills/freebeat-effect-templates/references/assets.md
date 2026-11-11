# Effect Assets

## Quick workflow

Pass a local path or remote URL directly to `effect run`:

```bash
freebeat effect run --effect-id <effectId> --audio song.mp3 --image ref.jpg --prompt "anime glow" --wait
```

If audio or image is omitted, `effect run` uses the selected template's defaults.

## Prepared assets

Upload separately only when `effect create` needs explicit IDs and hosted URLs:

```bash
freebeat assets upload-audio --file song.mp3
freebeat assets upload-audio --url https://soundcloud.com/artist/track
freebeat assets upload-image ref.jpg
```

Use the returned `musicId` and first `imageUrl` with `effect create`.

Keep raw API keys out of logs and serialized asset metadata.
