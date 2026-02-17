# Source Polling Reference — Monitor

## Gmail & Google Calendar (gog skill)

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
