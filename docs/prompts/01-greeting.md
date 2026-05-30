# Block 1 — Greeting Message

**Phonely block type:** Greeting Message
**Fires:** Immediately when a call is answered

## Prompt (exact text)

```
Thank you for calling BlueWing Phonely Airlines, your trusted partner for booking flights worldwide. I can help you search for flights, book a ticket, or answer questions about our policies. To get started, where would you like to fly from?
```

## Design notes

- **Identifies the airline by name** so the caller knows immediately who they reached.
- **Sets scope** — explicitly lists the three things the agent can help with (search, book, answer policy questions).
- **Starts collecting the first variable** in the same turn (`departure_code`) by asking "where would you like to fly from?" — avoids a wasted turn.
- **No recording disclosure** in this demo build (would be added for production: *"This call may be recorded for quality and training purposes."*).
