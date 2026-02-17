# Briefing Templates & Priority Classification — Synthesizer

## Priority Classification Guide

### ⚡ P1 — Immediate (Urgent Alert)
Deliver within seconds. Never batch.
- Monitor flagged as `[URGENT]`
- Calendar event starting in < 30 minutes with no prior alert
- Direct message from VIP contact with urgency keywords
- Email marked starred/important with action-required language

### 🔶 P2 — High (Top of Digest)
Surface prominently in next digest.
- Direct Slack mentions or DMs
- Emails requiring reply (questions, requests)
- Calendar events in next 2 hours
- Messages from VIP contacts (non-urgent)

### 📋 P3 — Normal (Standard Digest)
Include in digest, standard formatting.
- Channel messages with relevant content
- Informational emails (newsletters excluded)
- Calendar events beyond 2 hours

### 🔽 P4 — Low (Summarize or Omit)
Mention count only, or omit if digest is already long.
- Automated notifications
- Large group chat noise
- Marketing/newsletter emails

## Morning Briefing Template

```
☀️ **Morning Briefing — {YYYY-MM-DD, Day of Week}**

📅 **Today's Calendar**
{For each event, chronologically:}
- {HH:MM} — {Title} | {Location or Link}
{If no events: "No events scheduled."}

📧 **Overnight Inbox** ({total_count} new)
{If urgent:}
- ⚡ **Needs attention:**
  - {From} — {Subject} ({time})
{If notable:}
- 📋 **Notable:**
  - {From} — {Subject}
{If none: "Inbox clear overnight."}

💬 **Messages** ({total_count} new)
- **Slack:** {count} in {channel_count} channels
  {If mentions: "⚡ Mentions: {details}"}
- **iMessage:** {count} new
- **WhatsApp:** {count} new
{If none: "No new messages overnight."}

🎯 **Action Items**
{Extracted from emails, messages, calendar:}
- {Action} — {source and context}
{If none: "No pending actions detected."}
```

## 30-Minute Digest Template

```
📊 **Digest — {HH:MM}**

{Include ONLY sections with new content since last digest.}

📧 **New Emails** ({count})
- {From} — {Subject}

💬 **New Messages** ({count})
- {Source}: {From} — {Summary}

📅 **Upcoming** (next 60 min)
- {HH:MM} — {Event Title}

{If nothing new:}
✅ All quiet since last digest.
```

## Urgent Alert Template

```
🚨 **Urgent Alert**

**{Source}:** {One-line summary}
**From:** {Sender name}
**Time:** {Timestamp}

{2-3 lines of context if available}
```

## Deduplication Rules

1. Track item IDs (message ID, event ID) already surfaced in today's digests
2. If an item was in a previous digest, skip it unless its content changed
3. If an urgent alert was already sent, don't repeat in the next digest
4. Calendar events: show in digest when within 60 minutes, even if shown before
