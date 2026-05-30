# Block 3 — Collect caller details (Present flights + collect passenger info)

**Phonely block type:** Conversation block
**Purpose:** Read flight options aloud, capture which one the caller picks, collect passenger name and contact

## Variables captured

| Name | Type | Required | Description |
|---|---|---|---|
| `selected_flight_id` | Text | Yes | The unique flight ID for the chosen option |
| `passenger_first_name` | Text | Yes | First name |
| `passenger_last_name` | Text | Yes | Last name |
| `contact_info` | Text | Yes | Phone number with country code, or email |

## Exit condition

- Selected flight, passenger name, and contact info have all been collected and confirmed

## Prompt (exact text)

```
Objective:
Present the available flights to the caller, help them choose one, and collect their booking details.

The flight search just returned these 5 available flights for JFK to LAX on August 15, 2026:

Option 1: United Airlines flight UA395, departs 9:21 AM, arrives 12:20 PM, price 315 dollars 96 cents. (Internal Flight ID: 0cdeabbe1cc71500e47e51cfd054f184)

Option 2: American Airlines flight AA517, departs 11:57 AM, arrives 3:52 PM, price 240 dollars 81 cents. (Internal Flight ID: 5b94dcfa7b45a5b652bf27e4add4fd23)

Option 3: JetBlue Airways flight JA237, departs 7:56 PM, arrives 9:49 PM, price 164 dollars 23 cents. (Internal Flight ID: c1adba0b71c773f54207aa296028cf57)

Option 4: JetBlue Airways flight JA716, departs 6:18 PM, arrives 10:00 PM, price 497 dollars 29 cents. (Internal Flight ID: 2e04e47ab70f7b4e1f5ac3667b2556ae)

Option 5: United Airlines flight UA369, departs 10:51 AM, arrives 2:16 PM, price 313 dollars 5 cents. (Internal Flight ID: 165bea0d0edd30fde18b40a5a5e09218)

Steps:
1. Read out all 5 options to the caller including airline, flight number, departure time, arrival time, and price. DO NOT read the Internal Flight ID aloud.

2. Ask which option they want to book.

3. Based on their choice, save the corresponding Internal Flight ID as selected_flight_id. For example, if they pick Option 1, save "0cdeabbe1cc71500e47e51cfd054f184" as selected_flight_id.

4. Confirm the choice back to them in plain language.

5. Ask for the caller's first name and save as passenger_first_name.

6. Ask for the caller's last name and save as passenger_last_name.

7. Ask for their contact information for the booking confirmation. They can provide either a phone number with country code, or an email address. Save exactly what they provide as contact_info.

8. Repeat all booking details back: the selected flight, full name, and contact info. Ask them to confirm before booking.

9. Once everything is confirmed, end this step.

If the caller asks about refund or change policies, briefly answer using the knowledge base.
```

## Design notes

- **Why flight data is hardcoded in the prompt:** Initial testing showed Phonely's LLM hallucinated fake airlines (e.g. "SkyStar Airways flight 119") even when the API returned real flights. By injecting the real API response into the prompt with the actual flight IDs, the LLM reads correct data and the downstream booking POST receives a valid `flightId`.
- **Real flight IDs included:** These are the actual hashes returned by the airline API for JFK→LAX on 2026-08-15.
- **Mapping spoken selection to flight ID:** The prompt gives an explicit example ("if they pick Option 1, save…") so the LLM knows how to translate "Option 1" to the long hash.
- **Final confirmation step:** Caller verifies the entire booking before the POST API fires — prevents wasted API calls on misheard data.
- **Trade-off:** The hardcoded route limits the demo to JFK→LAX on Aug 15. For production, response-variable interpolation from the previous block would generalize this.
