# Source Polling Reference — Monitor

## Read-Only Access Policy

Every skill in this formation is used for **reading only**. You must never call any skill command that sends, creates, modifies, deletes, or marks-as-read. The permitted and forbidden operations for each skill are listed below.

If you encounter a skill command not listed under "Permitted Operations," do not use it.

---

## Gmail & Google Calendar (gog skill)

### Account Configuration

Parse `{{GOG_ACCOUNTS}}` on startup. It is a comma-separated list of `email:scope` entries:
- `user@gmail.com:email` — poll inbox only
- `user@gmail.com:cal` — poll calendar only
- `user@gmail.com:both` — poll inbox and calendar

For each account, only run the commands matching its declared scope.

### Required OAuth Scopes (read-only only)
- `https://www.googleapis.com/auth/gmail.readonly` — Read email, never send
- `https://www.googleapis.com/auth/calendar.readonly` — Read calendar, never create/edit events

### Permitted Operations
- `gog mail list` — List emails with filters
- `gog mail read` — Read a specific email by ID
- `gog cal list` — List calendar events in a time range

### Forbidden Operations — NEVER USE
- `gog mail send` — Sending email
- `gog mail reply` — Replying to email
- `gog mail forward` — Forwarding email
- `gog mail modify` — Modifying labels, marking read/unread
- `gog mail delete` — Deleting email
- `gog mail trash` — Trashing email
- `gog cal create` — Creating calendar events
- `gog cal update` — Modifying calendar events
- `gog cal delete` — Deleting calendar events
- Any `gog` command with `--send`, `--reply`, `--create`, `--update`, `--delete`, or `--modify` flags

### Commands
```bash
# List unread emails
gog mail list --account {{account}} --filter unread --limit 50

# Read specific email
gog mail read --account {{account}} --id {message_id}

# List calendar events
gog cal list --account {{account}} --from now --to +12h

# List upcoming events (light scan)
gog cal list --account {{account}} --from now --to +30m
```

### Expected Output Format
```json
{
  "messages": [
    {
      "id": "msg_abc123",
      "from": "sender@example.com",
      "subject": "Meeting follow-up",
      "snippet": "First 100 chars of body...",
      "date": "2026-02-17T10:30:00Z",
      "labels": ["INBOX", "UNREAD", "STARRED"],
      "threadId": "thread_xyz"
    }
  ]
}
```

## Slack (slack skill)

### Required Token Scopes (read-only only)
The `xoxp-` user token must be provisioned with **read-only scopes only**:
- `channels:history` — Read channel messages
- `channels:read` — List channels
- `groups:history` — Read private channel messages
- `groups:read` — List private channels
- `im:history` — Read DMs
- `im:read` — List DM conversations
- `mpim:history` — Read group DMs
- `users:read` — Resolve user names

**Do NOT grant:** `chat:write`, `channels:write`, `reactions:write`, or any write scope.

### Permitted Operations
- `slack messages` — List channel messages
- `slack dms` — List direct messages
- `slack mentions` — List mentions of the user

### Forbidden Operations — NEVER USE
- `slack send` — Sending messages
- `slack post` — Posting to channels
- `slack react` — Adding reactions
- `slack upload` — Uploading files
- Any `slack` command with `--send`, `--post`, `--reply`, or `--react` flags

### Commands
```bash
# List recent messages in all channels
slack messages --token {{SLACK_USER_TOKEN}} --since {last_harvest_ts} --limit 100

# List DMs only
slack dms --token {{SLACK_USER_TOKEN}} --since {last_harvest_ts}

# List mentions
slack mentions --token {{SLACK_USER_TOKEN}} --since {last_harvest_ts}
```

### Expected Output Format
```json
{
  "messages": [
    {
      "channel": "#engineering",
      "user": "john.doe",
      "text": "Message content here",
      "ts": "1739784600.000100",
      "thread_ts": null,
      "is_mention": false
    }
  ]
}
```

## iMessage (imsg skill)

### Access Model
Reads directly from the local iMessage SQLite database. Inherently read-only — the database is owned by the Messages app and the skill only queries it.

### Permitted Operations
- `imsg list` — List messages with time/contact filters

### Forbidden Operations — NEVER USE
- `imsg send` — Sending iMessages
- `imsg reply` — Replying to messages
- Any `imsg` command with `--send` or `--reply` flags

### Commands
```bash
# List recent messages
imsg list --since {last_harvest_ts} --limit 100

# List messages from specific contact
imsg list --from "+5511999999999" --since {last_harvest_ts}
```

### Expected Output Format
```json
{
  "messages": [
    {
      "id": "imsg_001",
      "from": "+5511999999999",
      "fromName": "Contact Name",
      "text": "Message text",
      "date": "2026-02-17T10:30:00Z",
      "group": null,
      "isFromMe": false
    }
  ]
}
```

## WhatsApp (wacli skill)

### Access Model
Reads from a pre-synced WhatsApp database. The database is a read-only export — the skill queries it but cannot interact with the live WhatsApp service.

### Permitted Operations
- `wacli messages` — List messages with time/group/contact filters

### Forbidden Operations — NEVER USE
- `wacli send` — Sending WhatsApp messages
- `wacli reply` — Replying to messages
- Any `wacli` command with `--send` or `--reply` flags

### Commands
```bash
# List recent messages
wacli messages --since {last_harvest_ts} --limit 100

# List messages from specific group
wacli messages --group "Group Name" --since {last_harvest_ts}
```

### Expected Output Format
```json
{
  "messages": [
    {
      "id": "wa_001",
      "from": "+5511888888888",
      "fromName": "Contact Name",
      "text": "Message text",
      "date": "2026-02-17T10:30:00Z",
      "group": "Family Group",
      "isFromMe": false
    }
  ]
}
```

## Urgency Detection Keywords

Flag items as urgent if any of these conditions match:
- **Gmail:** starred, labels include IMPORTANT, subject contains: `urgent`, `asap`, `deadline`, `action required`, `time-sensitive`
- **Calendar:** event starts within 30 minutes
- **Slack:** direct mention (@user), DM
- **iMessage/WhatsApp:** from VIP contact list, contains urgency keywords
