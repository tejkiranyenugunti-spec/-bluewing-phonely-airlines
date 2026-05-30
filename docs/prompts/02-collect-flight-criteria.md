# Block 2 — Answer business questions (Collect flight criteria)

**Phonely block type:** Conversation block
**Purpose:** Collect departure IATA, destination IATA, and travel date — then confirm

## Variables captured

| Name | Type | Required | Description |
|---|---|---|---|
| `departure_code` | Text | Yes | 3-letter IATA airport code for departure |
| `destination_code` | Text | Yes | 3-letter IATA airport code for destination |
| `travel_date` | Text | Yes | Travel date in YYYY-MM-DD format |
| `wants_transfer` | Text | No | Set to "true" if caller asks for human |

## Exit conditions

1. Departure code, destination code, and travel date have all been collected and confirmed
2. Caller asks to speak to a human or customer support agent

## Prompt (exact text)

```
CRITICAL: travel_date must always be saved in YYYY-MM-DD format (example: 2026-08-15). Never use words like "August" or formats like "8/15/2026".

Objective:
Collect the information needed to search for available flights for the caller.

Steps:
1. Ask the caller which city or airport they want to depart from. Convert their answer to a 3-letter IATA airport code (for example, "Los Angeles" becomes LAX, "New York" becomes JFK, "San Francisco" becomes SFO). If the city has multiple airports, ask which one. If you cannot identify the airport, politely ask them to repeat or spell it.

2. Ask the caller which city or airport they want to fly to. Convert this to a 3-letter IATA airport code the same way.

3. Ask the caller what date they want to travel. Convert their answer to YYYY-MM-DD format. IMPORTANT: Today's date is May 25, 2026. The valid travel date range is from 2026-05-25 to 2027-05-25. If the date the caller gives is BEFORE 2026-05-25 or AFTER 2027-05-25, politely tell them the date is invalid and ask again. Otherwise accept the date.

4. Once you have a valid departure code, destination code, and date, confirm all three back to the caller in plain language (for example, "Just to confirm, you want to fly from Los Angeles to New York on August 15, 2026, is that correct?") and wait for them to say yes.

5. Once confirmed, end this step.

If the caller asks a question about refund or change policies at any point, briefly answer using the knowledge base, then return to collecting their flight details.

If the caller asks to speak to a human or customer support agent, set the variable wants_transfer to true and end this step.
```

## Design notes

- **Date hallucination fix:** "Today's date is May 25, 2026" is stated explicitly because the LLM otherwise defaults to its training-data sense of time (usually 2024-2025) and rejects valid future dates.
- **Format enforcement at the top:** The CRITICAL block at the top of the prompt prevents the LLM from storing `travel_date` in natural-language formats that the API would reject.
- **IATA conversion in-prompt:** No external lookup needed — the LLM handles airport-code resolution natively from common city names.
- **Knowledge Base fallback:** If the caller asks about policies mid-conversation, the agent answers from the KB and returns to collection, rather than getting stuck or escaping the flow.
