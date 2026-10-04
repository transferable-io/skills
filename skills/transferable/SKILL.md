---
name: transferable
description: Upload files and folders to a Transferable library and send them to a client as a delivery link, from the terminal, with the `transferable` CLI. Use when the user wants to deliver, send, share or hand over photos, videos, rushes, masters or any files to a client through Transferable, or asks to upload to their Transferable library, check an upload, or list their deliveries.
---

# Transferable

Transferable is a file delivery service for videographers and photographers. The
`transferable` CLI uploads files to the user's library and turns them into a delivery: a
private link (`https://transferable.io/@handle/...`) the user sends to their client.

## Setup

1. Check the CLI: `transferable --version`. If missing, install it:
   - macOS / Linux with Homebrew: `brew install transferable-io/tap/transferable`
   - with Node.js: `npm install -g @transferable/cli`
   - otherwise: `curl -fsSL https://transferable.io/install | sh`
2. Check the account: `transferable whoami`. If it says not logged in, run
   `transferable login`: it opens the browser, and **the user must sign in and click
   Authorize themselves**. Tell them, then wait for the command to return.

Always pass `--json` when you need to read the output.

## Upload

```sh
transferable upload <files or folders...> [--folder "Client - Project"] --background --json
# -> {"job_id": "5097d224"}
transferable status 5097d224 --json   # progress and per-file results
```

- Always use `--background` for uploads: large files take hours and would outlive your
  command timeout. The job keeps running after you return.
- Poll with `transferable status <job_id> --json` (or `--wait` when you can block). `status`
  is `running`, `done`, `failed` or `interrupted`.
- A folder is recreated as is in the library; its root gets a number ("Wedding 2") if the
  name is taken, nothing is merged. `--folder` puts everything inside that library folder,
  created if missing.
- `interrupted` means the machine slept or the process died: rerun the same upload command,
  it resumes where it stopped (the status output prints the exact command).
- Errors to relay to the user as is: `quota_exceeded`, `file limit`, `not enough storage`.

## Deliver

```sh
transferable deliver "Wedding - Dupont" --from "Client - Project" --json
transferable deliver "Wedding - Dupont" --media <id>,<id> --expire 30 --json
```

- `--from` takes a folder at the root of the library (name or id) and includes all its
  subfolders. `transferable ls [folder-id] --json` lists folders and media.
- **Never add `--publish` without the user's explicit go-ahead**: publishing makes the link
  live for their client. Create the draft, show the user the title, the content and the
  link, and publish only when they confirm, with `--publish` on a new delivery or by asking
  them to switch it live in the app.
- Publishing is refused while media are still uploading (`media_not_ready`): wait for the
  upload job to be `done` first. `publish_error` in the answer means the delivery was
  created as a draft but not published (for example `needs_card`: the user must add a
  payment method in the app).
- `--expire <days>` is capped by the user's plan. `--accent "#RRGGBB"` sets the accent color.

## Other

- `transferable deliveries --json`: the user's deliveries with their links and status.
- `transferable status --json`: recent background uploads.
- Streaming is limited to 1080p; clients download the original files.
