# Block 4 — End Call (Success / Booking Confirmation)

**Phonely block type:** End Call
**Fires:** After POST API Request → Success branch (and after the Send SMS block)
**Field:** "End Call Message Prompt" (Promptable)

## Prompt (exact text)

```
The flight has been successfully booked. The booking API response includes a confirmation number. Read the confirmation number clearly to the caller, then summarize the booking: airline, flight number, departure city, destination city, and travel date. Tell them they will receive a confirmation by SMS or email shortly. End with "Thank you for booking with BlueWing Phonely Airlines, have a great flight!"
```

## Design notes

- **Confirmation number is spoken** so the caller has it immediately even if the SMS is delayed.
- **Full booking summary repeated** acts as a final receipt — caller can catch any error before hanging up.
- **SMS mention sets expectations** — caller knows to look for a follow-up message.
- **Branded sign-off** reinforces airline identity at the end of the call.
