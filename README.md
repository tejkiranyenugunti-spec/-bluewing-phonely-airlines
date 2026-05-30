# BlueWing Phonely Airlines — Voice Booking Agent

A production voice AI agent built on the **Phonely** platform that handles end-to-end airline reservations entirely over the phone.

📞 **Call the live agent:** (361) 321-9173
🎥 **Demo video:** *(https://app.govideolink.com/videos/d1IgwOqqS35M6Nki8TiT/?utm_source=direct&utm_medium=invite_link])*

---

## What it does

A caller phones the agent and books a flight by voice. The agent:

1. Resolves spoken city names (e.g. *"New York"*) to IATA codes (`JFK`)
2. Searches available flights via the airline REST API
3. Reads flight options aloud — airline, flight number, time, price
4. Collects passenger details (name, contact info)
5. Books the selected flight via POST API and returns a confirmation number
6. Sends an SMS confirmation to the caller's phone

It also handles:
- **Transfer to human support** when the caller asks
- **Knowledge Base (RAG)** answers for refund and change policy questions
- **Error paths** for no-flights-available (e.g. AAL → YVR test case) and booking failures
- **Date validation** within today + 365 days
- **Unknown airport** handling with re-prompts

---

## Architecture

```
Greeting Message
    ↓
Answer business q (collects departure / destination / date)
    ├─ "Depa…" branch → API Request (GET — search flights)
    │                       ├─ Success → Collect caller details (present options, collect passenger info)
    │                       │              ↓
    │                       │          API Request (POST — book flight)
    │                       │              ├─ Success → Send SMS → End Call (confirmation)
    │                       │              └─ Error  → End Call (booking failed)
    │                       └─ Error → End Call (no flights available)
    └─ "Calle…" branch → Transfer Call (to customer support)

Post-call:
    Email & SMS Notification → call summary to operations email
```

---

## Tech stack

| Layer | Technology |
|---|---|
| Voice AI platform | Phonely (Pro tier) |
| Conversational LLM | Phonely's built-in models |
| External APIs | REST (GET flight search, POST flight booking) |
| Knowledge Base | Phonely RAG over uploaded policy document |
| Outbound SMS | Phonely Send SMS block |
| Transfer routing | Phonely Cold Transfer |

---

## Key learnings (how the agent was tuned)

### 1. LLMs don't reliably know today's date
The initial date-validation prompt said *"must be within today + 365 days"*. The agent kept rejecting valid future dates (August 2026) because internally it thought it was still in 2025.

**Fix:** State today's date explicitly in the prompt.

> *"IMPORTANT: Today's date is May 25, 2026. The valid travel date range is from 2026-05-25 to 2027-05-25."*

### 2. Phonely variable interpolation is strict
After importing a cURL command, variables came in as literal placeholder text (`{{departure_code}}`). The API rejected every request with `"Invalid airport code(s)"` because it received the literal `{{departure_code}}` string.

**Fix:** Delete the literal placeholder text and re-insert each variable using Phonely's variable picker. The result must be **only the variable pill** in the value field — no surrounding `{{` or `}}` braces.

### 3. The LLM hallucinated flight data
Even after the API was returning real flights, the agent would invent fake options like *"SkyStar Airways flight 119"*. When the caller picked one, the booking POST sent a non-existent flight ID and failed.

**Fix:** Inject the actual API response shape and real flight IDs into the prompt for the flight-presentation block. The agent reads the real United / American / JetBlue flights and the booking POST receives a valid ID.

### 4. Voice latency is sensitive
Callers would sit in silence while API calls ran. Added **interim messages** like *"Searching for available flights now"* on the API Request blocks so the agent speaks while the API works in the background.

---

## Project structure

```
bluewing-phonely-airlines/
├── README.md
└── docs/
    ├── architecture.md            ← full flow design, block-by-block
    ├── api-integration.md         ← GET/POST configuration + test cases
    ├── prompts/
    │   ├── 01-greeting.md
    │   ├── 02-collect-flight-criteria.md
    │   ├── 03-collect-caller-details.md
    │   ├── 04-end-call-success.md
    │   └── 05-end-call-error.md
    └── knowledge-base/
        └── refund-and-change-policies.md
```

---

## Test scenarios

| Scenario | Phrase to say | Expected result |
|---|---|---|
| Success | *"I want to fly from New York to Los Angeles on August 15, 2026"* | Reads 5 real flights → books → sends SMS |
| No flights (assignment test case) | *"I want to fly from Aalborg to Vancouver on August 15, 2026"* | Error branch → polite "no flights available" |
| Transfer | *"Can I speak to customer support?"* | Routes to support number |
|  Knowledge Base | *"What's your refund policy?"* | RAG-backed answer from uploaded policy doc |

---

