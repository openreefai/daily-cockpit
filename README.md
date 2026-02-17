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
| `GOG_ACCOUNT_1` | string | Yes | — | Primary Gmail address for Gmail and Google Calendar polling. |
| `GOG_ACCOUNT_2` | string | Yes | — | Secondary Gmail address for Gmail and Google Calendar polling. |
| `DISCORD_CHANNEL` | string | Yes | — | Private Discord channel ID for briefings, digests, alerts, and Q&A. |

## Setup

### Prerequisites

- OpenClaw >= 0.5.0
- Skills installed: `gog`, `imsg`, `wacli`, `slack`, `summarize`
- Google OAuth configured for both Gmail accounts (read-only)
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
   - `GOG_ACCOUNT_1` — Primary Gmail address
   - `GOG_ACCOUNT_2` — Secondary Gmail address
   - `DISCORD_CHANNEL` — Discord channel ID for output

3. **Deploy the formation:**
   ```bash
   reef deploy
   ```

4. **Verify:**
   ```bash
   reef validate
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

- All source access is **strictly read-only**
- Slack uses a user token — no bot presence in workspace
- iMessage reads from local database — coexists with existing bot
- WhatsApp reads from pre-synced database only
- Sensitive tokens are never stored in plaintext (marked `sensitive: true`)

## Teardown / Uninstall

To remove the daily-cockpit formation:

1. **Stop the formation** (removes cron jobs and agent sessions):
   ```bash
   reef undeploy daily-cockpit
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

> **Note:** Undeploying does **not** affect your data sources. No messages, emails, or calendar events are modified or deleted — the formation is strictly read-only.

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
