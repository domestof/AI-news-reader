# AI-news-reader

This repository **contains no application code**. The project is a cloud
scheduled routine (Claude Code Routine) that generates a daily digest of
AI/tech news from the user's Gmail newsletters (LinkedIn, TLDR, Substack),
and leaves it as a Gmail draft plus a Telegram message.

- **Full architecture, pipeline, and required configuration**:
  [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)
- **History of decisions and resolved issues** (MCP permissions, expired
  OAuth token, Telegram network block, etc.): [`docs/STATUS.md`](docs/STATUS.md)

## Quick reference

- Routine: "Resum diari IA/Tech (LinkedIn + newsletters)" —
  see it at https://claude.ai/code/routines
- Schedule: `0 5 * * *` UTC (7:00 Madrid during daylight saving)
- Management: `RemoteTrigger` tool (`action: get|update|run`) or the
  `schedule` skill
- Status: **working** (confirmed 2026-08-05) — the user has to manually click
  "Send" every day on the draft the routine generates

Before touching the routine (adding sources, changing the schedule, item
caps, etc.), read `docs/ARCHITECTURE.md` to understand the connectors'
limitations (Gmail only creates drafts, Drive only creates new files) before
proposing changes that depend on them.
