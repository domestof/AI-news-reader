# AI News Reader

A daily tech news digest, automated end-to-end with a [Claude Code Routine](https://code.claude.com/docs/en/routines) — no application code, no server to maintain.

## What it does

Every morning, an unattended cloud agent:

1. Searches Gmail for newsletters from LinkedIn, TLDR, and Substack received in the last 48 hours.
2. Extracts individual news items and filters them down to five topics: **AI**, **tech law & regulation**, **cybersecurity**, **infrastructure**, and **Linux**.
3. Merges duplicate coverage of the same story across sources (more sources covering a story boosts its importance score).
4. Checks a running history in Google Drive to skip stories already surfaced in the last 30 days, unless there's a genuine update.
5. Scores each remaining item 1–5 🔥 based on source count, specificity/impact, and actionability.
6. Composes a digest capped at 5 items per category, sorted by score.
7. Delivers the digest as a **Telegram message**.
8. Labels processed emails (`ResumIA-Processat`) so they aren't re-read tomorrow.
9. Updates the Drive history file for next time.

## Why

Manually triaging a flooded inbox of AI/tech newsletters doesn't scale. This routine does the reading, filtering, deduplication, and prioritization automatically, and drops the result straight into a Telegram chat — fully hands-off.

## How it works

There's no code here — the entire pipeline is a prompt stored in a Claude Code Routine's configuration, running on Anthropic's cloud infrastructure on a daily schedule. It reaches Gmail and Google Drive through MCP connectors, and Telegram through a direct HTTPS call.

See **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** for the full architecture diagram, pipeline details, and required configuration (connectors, tool permissions, network access, scheduling).

## Status

**Working.** Runs daily at 5:00 UTC, delivering to Telegram only.

## Configuration

This project is 100% configuration, not code:

- A Claude Code Routine (prompt + schedule + connectors), managed at [claude.ai/code/routines](https://claude.ai/code/routines) or via the `/schedule` CLI command.
- Gmail and Google Drive connectors, authorized under the account running the routine (Gmail is read/label only — it's not the delivery channel).
- A dedicated Telegram bot and private group — the sole delivery channel.
- A cloud environment with **Custom** network access (to reach `api.telegram.org`, which isn't in the default allowlist).

None of these values are stored in this repository — see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#required-configuration) for what's needed and where it lives.

## Billing

This runs on the Claude Pro/Max subscription's included usage, not pay-per-token API credits — see the [Billing section](docs/ARCHITECTURE.md#billing) in the architecture doc for the official source.

## Maintenance

- View or edit the routine: [claude.ai/code/routines](https://claude.ai/code/routines)
- Run it on demand, change the schedule, sources, or scoring rules: see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#maintenance)

## Known limitations

- No way to update an existing Drive file — each run uploads a new version of the history file.
- Push notifications only work if the routine's cloud environment allows outbound network access to the relevant host (Telegram works after enabling Custom network access; a plain notification service like ntfy.sh was tried and dropped for UX reasons, not a technical failure).

Full details in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#known-limitations).
