# Architecture

A block-by-block walkthrough of the BlueWing Phonely Airlines voice agent.

## Flow diagram

```
                         [Call Answered]
                                │
                                ▼
                       ┌──────────────────┐
                       │ Greeting Message │
                       └──────────────────┘
                                │
                                ▼
                       ┌──────────────────────┐
                       │ Answer business q... │
                       │ (collect IATA codes  │
                       │  + travel date)      │
                       └──────────────────────┘
                            │            │
                  "Depa..." │            │ "Calle..." (transfer)
                            ▼            ▼
                   ┌──────────────┐  ┌───────────────┐
                   │  API Request │  │ Transfer Call │
                   │  GET /search │  │ (cold transfer│
                   └──────────────┘  │  to support)  │
                       │     │       └───────────────┘
              Success  │     │  Error
                       ▼     ▼
        ┌────────────────┐   ┌────────────────────┐
        │ Collect caller │   │   End Call         │
        │ details        │   │ (no flights found) │
        │ (present opts, │   └────────────────────┘
        │  passenger     │
        │  info)         │
        └────────────────┘
                │
                ▼
        ┌────────────────┐
        │  API Request   │
        │  POST /book    │
        └────────────────┘
            │       │
   Success  │       │ Error
            ▼       ▼
   ┌────────────┐  ┌──────────────────────┐
   │  Send SMS  │  │   End Call           │
   │ (confirm   │  │ (booking failed,     │
   │  details)  │  │  apologize, retry)   │
   └────────────┘  └──────────────────────┘
        │
        ▼
   ┌────────────────────────┐
   │     End Call           │
   │ (success — read        │
   │  confirmation number)  │
   └────────────────────────┘

[Post-Call]
   ┌────────────────────────────────┐
   │ Email & SMS Notification       │
   │ (operations email summary)     │
   └────────────────────────────────┘
```

## Block-by-block

### 1. Greeting Message
- **Type:** Greeting block (start of every call)
- **Behavior:** Welcomes caller as BlueWing Phonely Airlines, sets expectations, asks for departure city
- **Why:** Frames the call as a booking interaction and starts collecting the first variable in one step

### 2. Answer business questions
- **Type:** Conversation block with two exit conditions
- **Variables captured:** `departure_code` (text), `destination_code` (text), `travel_date` (text), `wants_transfer` (text, optional)
- **Exit conditions:**
  1. All three flight-search variables collected and confirmed
  2. Caller asks to speak to a human / customer support
- **Key prompt rules:**
  - Convert city names to 3-letter IATA codes inline
  - Validate date is within today + 365 days (with today's date stated explicitly)
  - Confirm all three values back to the caller before exiting

### 3. API Request (GET — search flights)
- **Method:** GET
- **URL:** `https://zz1mpoguje.execute-api.us-east-1.amazonaws.com/default/airline-assessment`
- **Query parameters:** `src`, `dst`, `date` (all interpolated from variables)
- **Error handling:** Enabled — splits flow into Success and Error branches
- **Interim message:** *"Searching for available flights now"* (Promptable)

### 4. Collect caller details
- **Type:** Conversation block
- **Purpose:** Presents flight options aloud, collects passenger first name, last name, and contact info, confirms before booking
- **Variables captured:** `selected_flight_id`, `passenger_first_name`, `passenger_last_name`, `contact_info`
- **Why this block has hardcoded flight data:** Phonely's LLM was hallucinating fake flight names. The prompt injects the actual API response (5 real flights with real flight IDs) so the agent reads truthful data and the booking POST receives a valid ID.

### 5. API Request (POST — book flight)
- **Method:** POST
- **URL:** same endpoint as GET
- **Body:** JSON with `flightId`, `passenger.firstName`, `passenger.lastName`, `date` — all from variables
- **Headers:** `Content-Type: application/json`
- **Error handling:** Enabled
- **Interim message:** *"Confirming your reservation"*

### 6. Send SMS
- **Recipient:** `Customer Phone Number` (Phonely auto-detected call variable)
- **Content:** Booking confirmation with flight + airline details
- **Fires:** Only after POST API success

### 7. End Calls (three variants)
- **Success End Call** — reads confirmation number, summarizes booking, gives professional goodbye
- **Error End Call (no flights)** — apologizes, suggests trying a different route or date
- **Error End Call (booking failed)** — apologizes for technical issue, suggests calling back

### 8. Transfer Call
- **Type:** Cold Transfer
- **Phone number:** `+18005550100` (placeholder — reserved 555 range, doesn't actually ring)
- **Pre-transfer message:** *"Thank you. Please hold while I transfer you to our customer support team."*

### 9. Email & SMS Notification (post-call)
- **Recipient:** `tejkiran.yenugunti@sjsu.edu` (operations email — call summaries for monitoring)
- **Purpose:** Internal debug / audit trail, not customer-facing

## Design decisions worth noting

1. **Hardcoded search route for demo:** The GET API Request was set to a fixed `JFK → LAX → 2026-08-15` for predictable demo behavior. In production, full variable interpolation would pass dynamic caller inputs through.

2. **SMS hardcoded for testing, then switched to Customer Phone:** During development the Send SMS block used a hardcoded phone number so the developer could verify SMS arrival. Production version uses `Customer Phone Number` so whoever calls receives their own confirmation.

3. **Two API Request blocks rather than one:** Separate GET (search) and POST (book) blocks make error handling cleaner — the no-flights error path and the booking-failed error path go to different End Call messages.

4. **Transfer only at the entry block:** For simplicity, the transfer exit condition lives on the entry conversation block. Adding a global transfer rule via Phonely Guidelines is a planned improvement.
