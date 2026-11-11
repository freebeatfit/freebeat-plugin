# Media Inputs

## Where images come from

The bundled Freebeat server runs remotely. It cannot read the user's disk, and it only accepts HTTPS image URLs.

- ChatGPT attachment -> `upload_image {files: [...]}` (one or two per call) -> returned `image_urls`.
- Public HTTPS image link -> `upload_image {urls: [...]}` to copy it into Freebeat, or pass it directly when it is already a stable HTTPS image URL.
- Local file in Codex -> `freebeat --json assets upload-image <path> [<path>]` with the Freebeat CLI, signed in to the same Freebeat account; it returns `imageUrls`. Attachments reach Freebeat only in ChatGPT. Without the CLI, do not install it or ask for an attachment: ask the user to share an HTTPS link, or to install the CLI from https://freebeat.ai/cli and run `freebeat auth set-api-key` in their own terminal.
- `file_paths` works only in a local stdio install of the server, never on the hosted plugin.

`upload_image` accepts PNG or JPG, up to 10 MB each.

## Image count by mode

| Tool and mode | Images |
|---|---|
| `edit_image` | exactly one |
| `generate_video` `text` | none |
| `generate_video` `image` | exactly one |
| `generate_video` `transition` | exactly two, first frame then last frame |
| `generate_video` `reference` | one or more; use the fewest the request needs |

Freebeat enforces the per-model limit; a rejection message says when a model takes fewer images.
