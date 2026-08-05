# AI-news-reader

This repository **contains no application code**. The project is a cloud
scheduled routine (Claude Code Routine) that generates a daily digest of
AI/tech news from the user's Gmail newsletters (LinkedIn, TLDR, Substack),
and delivers it as a Telegram message. Gmail is read/label only, not a
delivery channel.

- **Full architecture, pipeline, and required configuration**:
  [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)

## Quick reference

- Routine: "Resum diari IA/Tech (LinkedIn + newsletters)" —
  see it at https://claude.ai/code/routines
- Schedule: `0 5 * * *` UTC (7:00 Madrid during daylight saving)
- Management: `RemoteTrigger` tool (`action: get|update|run`) or the
  `schedule` skill
- Status: **working** — delivers to Telegram daily, no user action needed

Before touching the routine (adding sources, changing the schedule, item
caps, etc.), read `docs/ARCHITECTURE.md` to understand the connectors'
limitations (Gmail is read-only, Drive only creates new files, no "update")
before proposing changes that depend on them.
