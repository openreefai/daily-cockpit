# daily-cockpit

Personal command center that monitors WhatsApp, Slack, iMessage, Gmail, and Google Calendar to deliver organized briefings, priority alerts, and interactive Q&A.

## Formation Overview

| | |
|---|---|
| **Type** | Shoal (2 agents) |
| **Namespace** | `daily-cockpit` |
| **Operating Hours** | 08:00–19:00 São Paulo (America/Sao_Paulo) |

### Agents

| Agent | Role | Model | Description |
|-------|------|-------|-------------|
| **monitor** | Collector + Sentinel | Claude Sonnet | Polls all 5 sources on schedule. Detects urgency. |
| **synthesizer** | Coordinator + User-facing | Claude Opus | Produces briefings, digests, alerts. Handles Q&A via Discord. |

### Data Flow

```
Gmail ──┐
GCal ──┤
Slack ──┤──→ Monitor ──→ Synthesizer ──→ Discord (user)
iMsg ──┤                      ↑
WhatsApp┘               User Q&A (Discord)
```

## Variables

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `SLACK_USER_TOKEN` | string | Yes | — | Slack user OAuth token (`xoxp-`). Used for read-only access to Slack messages. Sensitive. |
| `GOG_ACCOUNTS` | string | Yes | — | Comma-separated Google accounts with scope. Format: `email:scope` where scope is `email`, `cal`, or `both`. Example: `user1@gmail.com:both,user2@gmail.com:email` |
| `DISCORD_CHANNEL` | string | Yes | — | Private Discord channel ID for briefings, digests, alerts, and Q&A. |

## Setup

### Prerequisites

- OpenClaw >= 0.5.0
- Skills installed: `gog`, `imsg`, `wacli`, `slack`, `summarize`
- Google OAuth configured for each Gmail account listed in `GOG_ACCOUNTS` (read-only scopes only)
- Slack user token (xoxp-) with read access
- macOS host (required for iMessage access)
- WhatsApp database pre-synced via wacli

### Installation

1. **Copy environment file:**
   ```bash
   cp .env.example .env
   ```

2. **Fill in your values in `.env`:**
   - `SLACK_USER_TOKEN` — Your Slack user OAuth token
   - `GOG_ACCOUNTS` — Google accounts with scopes (e.g. `dash@gmail.com:both,work@gmail.com:email`)
   - `DISCORD_CHANNEL` — Discord channel ID for output

3. **Install the formation:**
   ```bash
   reef install .
   ```

4. **Verify:**
   ```bash
   reef validate .
   ```

## Schedule

| Job | Agent | Schedule | Description |
|-----|-------|----------|-------------|
| Light Scan | monitor | Every 5 min (08–19h) | Quick urgency check across all sources |
| Full Harvest | monitor | Every 30 min (08–19h) | Complete data collection from all sources |
| Morning Briefing | synthesizer | 08:00 daily | Calendar overview + overnight summary |

## Usage

### Automatic Output

- **08:00** — Morning briefing with calendar, overnight messages, action items
- **Every 30 min** — Digest of new items since last update
- **Immediate** — Urgent alerts whenever detected (not gated by schedule)

### Interactive Q&A

Send messages in the configured Discord channel:

- "Did anyone email me about the proposal?"
- "What's on my calendar tomorrow?"
- "Any Slack messages from @john?"
- "Summarize what I missed in #engineering"

## Security Notes

### Read-Only Enforcement

This formation **never sends, replies, modifies, or deletes** anything in any source. Read-only access is enforced at three layers:

1. **OAuth / Token Scopes** (hardest to bypass)
   - **Google OAuth** must use `gmail.readonly` and `calendar.readonly` only. Do NOT grant `gmail.send`, `gmail.modify`, `calendar.events`, or any write scope.
   - **Slack user token** (`xoxp-`) must use read-only scopes only: `channels:history`, `channels:read`, `groups:history`, `groups:read`, `im:history`, `im:read`, `mpim:history`, `users:read`. Do NOT grant `chat:write`, `channels:write`, `reactions:write`, or any write scope.
   - **iMessage** — inherently read-only (queries local SQLite database owned by Messages.app)
   - **WhatsApp** — inherently read-only (queries pre-synced database export via wacli)

2. **Skill Command Restrictions** — Each skill's permitted and forbidden operations are documented in `agents/monitor/knowledge/static/source-polling-reference.md`. Only read/list commands are permitted; all send/reply/create/modify/delete commands are explicitly forbidden.

3. **SOUL Behavioral Rules** — Agent SOULs contain explicit prohibitions against calling write operations, reinforcing the policy even if a skill technically exposes them.

### Other Security Notes

- Slack uses a user token — no bot presence in workspace
- Sensitive tokens are marked `sensitive: true` in the manifest and never stored in plaintext

## Teardown / Uninstall

To remove the daily-cockpit formation:

1. **Uninstall the formation** (removes cron jobs and agent sessions):
   ```bash
   reef uninstall daily-cockpit
   ```

2. **Verify agents are stopped:**
   ```bash
   reef status
   ```

3. **Remove the formation directory** (optional — deletes all configuration):
   ```bash
   rm -rf daily-cockpit/
   ```

4. **Clean up environment variables** (optional):
   - Remove or revoke your `SLACK_USER_TOKEN` from Slack app settings
   - Revoke Google OAuth tokens if no longer needed

> **Note:** Uninstalling does **not** affect your data sources. No messages, emails, or calendar events are modified or deleted — the formation is strictly read-only.

## Directory Structure

```
daily-cockpit/
├── reef.json
├── reef.lock.json
├── .env.example
├── .gitignore
├── README.md
└── agents/
    ├── monitor/
    │   ├── SOUL.md
    │   ├── IDENTITY.md
    │   └── knowledge/
    │       ├── static/
    │       │   └── source-polling-reference.md
    │       └── dynamic/
    └── synthesizer/
        ├── SOUL.md
        ├── IDENTITY.md
        └── knowledge/
            ├── static/
            │   └── briefing-templates.md
            └── dynamic/
```
