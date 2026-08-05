# Daily AI/Tech News Digest — Architecture

## Goal

Automatically generate, every day, a digest of the most relevant news on
Artificial Intelligence, tech law/regulation, cybersecurity, infrastructure,
and Linux, sourced from newsletter emails arriving in Gmail (LinkedIn, TLDR,
Substack), with deduplication and importance scoring, delivered as a
Telegram message.

## Architecture

**There is no code or server of our own.** The entire system is a **cloud
scheduled routine** ([Claude Code Routines](https://code.claude.com/docs/en/routines)), managed with the `schedule` skill and
the `RemoteTrigger` tool. It doesn't run in this repository or on any of the
user's machines — it's an isolated Claude agent that spins up on its own every
day, runs a fixed prompt, and stops.

```
┌─────────────────────────────────────────────────────────────┐
│  Cloud routine (Claude Code Routine)                          │
│  Cron: 0 5 * * * UTC  ·  Environment: Default Cloud Environment │
│  Model: claude-sonnet-5                                      │
└───────────────┬───────────────────────────┬──────────────────┘
                │                           │
        ┌───────▼────────┐          ┌───────▼────────┐          ┌──────────────┐
        │  Gmail          │          │  Google Drive   │          │  Telegram     │
        │  connector (MCP)│          │  connector (MCP)│          │  Bot API      │
        │  - search/read  │          │  - JSON registry│          │  (curl/Bash)  │
        │  - label        │          │    of past news │          │  - sendMessage│
        └────────────────┘          └────────────────┘          └──────────────┘
```

Components:
- **Routine**: the full pipeline prompt, stored in the routine's `job_config`
  — not in files in this repo. Queried/edited with `RemoteTrigger`
  (`action: get|update|run`).
- **Gmail connector**: data source only (searches and reads emails, manages
  the control label). Does not deliver the digest.
- **Google Drive connector**: persistent memory across runs (registry of
  already-shown news items, to avoid repeats).
- **Telegram**: the sole delivery channel, reached with a direct `curl` call
  from Bash (no MCP connector involved).

## Features (9-step pipeline)

1. **Email search** — Gmail, last 48h, by sender (not by label or category,
   since these newsletters land directly in the INBOX unlabeled):
   - LinkedIn: `from:newsletters-noreply@linkedin.com`
   - TLDR (tech/AI/infosec/devops...): `from:tldr`
   - Substack: `from:substack.com`
2. **Candidate news extraction** — title, source, original link, short
   summary. Discards off-topic or purely promotional content.
3. **Same-day duplicate merging** — if several sources cover the same story,
   they're merged into one entry; more sources = higher importance.
4. **Cross-day deduplication** — checks the registry on Drive
   (`resum_ia_estat.json`) and skips news items already shown in the last 30
   days (unless there's a relevant update, explicitly flagged).
5. **Importance scoring** (🔥 1-5): +2 multi-source, +2 impact/specificity,
   +1 actionable content.
6. **Content composition** — grouped by category (🤖 AI, ⚖️ Law &
   Regulation, 🔒 Cybersecurity, 🖥 Infrastructure, 🐧 Linux), capped at
   **5 items per category** (highest-scoring first), with a link to the
   original source.
7. **Telegram message** — `curl` from Bash to the Telegram API
   (`sendMessage`), using a dedicated bot's token and the `chat_id` of a
   private Telegram group. If it fails, it doesn't block the rest of the
   routine. This is the **only delivery channel** — no Gmail draft is
   created.
8. **Marking processed emails** — `ResumIA-Processat` label, so future runs
   don't re-process the same emails.
9. **Registry update** — uploads a new version of `resum_ia_estat.json` to
   Drive with today's included news items, dropping entries older than 30
   days.

## Required configuration

### Connectors (claude.ai → Connectors)
- **Gmail**: connected and authorized. Needed for `create_label` and
  `label_message` to work (not just reading). No "send"/"draft" permissions
  are needed since Gmail is no longer a delivery channel.
- **Google Drive**: connected and authorized.

### Routine permissions (`job_config.ccr`)
- `session_context.allowed_tools`: `["Bash", "mcp__Gmail__*", "mcp__Google-Drive__*"]`
- `mcp_connections[].permitted_tools` — **must be declared explicitly per
  connector**, the `allowed_tools` wildcard alone isn't enough:
  - Gmail: `search_threads`, `get_message`, `get_thread`, `list_labels`,
    `create_label`, `label_message`, `unlabel_message`
  - Google Drive: `search_files`, `download_file_content`, `read_file_content`,
    `get_file_metadata`, `create_file`, `list_recent_files`

### Execution environment
- The user's own cloud environment ("Default Cloud Environment", always on,
  doesn't depend on any of the user's computers).
- **Network access: Custom** (manually configured by the user at
  claude.ai/code → environment selector → settings gear). By default the
  environment uses **"Trusted"** access, which only allows a
  [default allowlist](https://code.claude.com/docs/en/cloud-environments#default-allowed-domains)
  (package registries, cloud provider APIs, common developer domains) — this
  is why the first attempts at `api.telegram.org` and `ntfy.sh` failed (403).
  MCP connectors (Gmail/Drive) bypass this network filter, routed through
  Anthropic's servers, which is why they always worked.
  **Domains added to the Custom allowlist**: `api.telegram.org`, `ntfy.sh`
  (the latter is no longer used, but the domain remains allowed).
  This configuration **cannot be done via API/tools** — only from the web UI
  or the Desktop app (claude.ai/code). See
  [Configure cloud environments](https://code.claude.com/docs/en/cloud-environments#network-access)
  if it ever needs to be changed.

### Telegram
- Dedicated bot, shared with another of the user's agents (same bot,
  different chat) — verify the token directly with BotFather if it ever
  needs to be reconfigured.
- Private Telegram group dedicated to this digest (a basic group, not a
  supergroup).
- ⚠️ **Known pitfall**: if it ever fails again with `400 chat not found`
  despite a correct group ID, the most likely cause is that the stored token
  does NOT correspond to the bot that's actually a member of the group
  (multiple bots can coexist in the same group). Always check the group's
  member list on Telegram before suspecting the ID.

### Schedule
- Cron: `0 5 * * *` (UTC) = 7:00 Madrid time during daylight saving (worth
  revisiting once winter time kicks in, if keeping the exact local time
  matters).

## Billing

Claude Code Routines **draw from the subscription plan (Pro/Max)**, not
pay-per-token API credits — it's the same kind of usage as regular
interactive use. Official source
([code.claude.com/docs/en/routines](https://code.claude.com/docs/en/routines),
"Usage and limits" section):

> "Routines draw down subscription usage the same way interactive sessions do. [...]
> Without usage credits, additional runs are rejected until the window resets."

On top of the shared subscription limit, there's a **daily cap on the number
of routine runs** per account (viewable at claude.ai/code/routines). It would
only be billed at API rates if the user explicitly enables "usage credits"
(overage) on their account — by default, hitting the limit simply blocks new
runs until the window resets, with no charge.

This is a different product from the "build agents" section of the API
console (console.anthropic.com / Claude Developer Platform), which does bill
per token.

## Known limitations

- **No "update" on Drive**: the connector only has `create_file`, no way to
  edit an existing file. Every day a new version of `resum_ia_estat.json` is
  uploaded; the routine looks up the most recent one by `modifiedTime`. Old
  files stay on Drive (not auto-deleted).
- **Gmail's OAuth token can expire**: if labeling starts failing again while
  reads keep working, the most likely cause is an expired token
  — disconnect and reconnect the Gmail connector at
  claude.ai/customize/connectors (known bug: Claude doesn't refresh it on its
  own).
- **No Telegram message history**: not applicable — Telegram delivery works
  as documented above.

## Maintenance

- **View/edit the routine**: https://claude.ai/code/routines (search by name)
- **Run it on demand**: `RemoteTrigger({action: "run", trigger_id: "<ROUTINE_ID>"})`
- **Change the prompt** (add sources, change the item cap, the schedule,
  etc.): `RemoteTrigger({action: "update", trigger_id: "...", body: {...}})`
  — the entire `job_config` must be resent with the change applied (it's a
  partial update at the field level, not of text inside the prompt).
- **Verify it's working**: check the Telegram group for today's message,
  `list_labels` (`ResumIA-Processat` label with messages > 0), and the
  `resum_ia_estat.json` file on Drive with today's `modifiedTime`.
