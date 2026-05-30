# API Integration

The agent integrates with a REST airline API for two operations: **flight search (GET)** and **flight booking (POST)**.

## Base URL

```
https://zz1mpoguje.execute-api.us-east-1.amazonaws.com/default/airline-assessment
```

No authentication required.

---

## 1. Flight Search (GET)

### Request

```http
GET /default/airline-assessment?src={IATA}&dst={IATA}&date={YYYY-MM-DD}
```

| Query param | Source | Example |
|---|---|---|
| `src` | `departure_code` variable | `JFK` |
| `dst` | `destination_code` variable | `LAX` |
| `date` | `travel_date` variable | `2026-08-15` |

### Success response (200)

```json
{
  "src": "JFK",
  "dst": "LAX",
  "date": "2026-08-15",
  "flights": [
    {
      "flightId": "0cdeabbe1cc71500e47e51cfd054f184",
      "airline": "United Airlines",
      "flightNumber": "UA395",
      "departureTime": "2026-08-15T09:21:00.000Z",
      "arrivalTime": "2026-08-15T11:20:00.000Z",
      "durationMinutes": 119,
      "stops": 0,
      "price": 315.96
    }
    // ... more flight objects
  ]
}
```

### Error responses

| HTTP | Body | Trigger |
|---|---|---|
| `400` | `{"error": "Invalid date. Must be today or within 1 year."}` | Date is in past or > 365 days ahead |
| `400` | `{"error": "Invalid airport code(s)."}` | IATA code not recognized |
| `200` | `{"src":"AAL","dst":"YVR","date":"...","flights":[]}` | Valid request but no flights — empty array |

### Phonely configuration

- **Error Handling:** Enabled (creates Success / Error branches)
- **Timeout:** 25 seconds
- **Interim Message:** *"Searching for available flights now"* (Promptable)

---

## 2. Flight Booking (POST)

### Request

```http
POST /default/airline-assessment
Content-Type: application/json
```

```json
{
  "flightId": "0cdeabbe1cc71500e47e51cfd054f184",
  "passenger": {
    "firstName": "Tej",
    "lastName": "Yenugunti"
  },
  "date": "2026-08-15"
}
```

| JSON field | Source variable |
|---|---|
| `flightId` | `selected_flight_id` |
| `passenger.firstName` | `passenger_first_name` |
| `passenger.lastName` | `passenger_last_name` |
| `date` | `travel_date` |

### Success response (200)

Returns a confirmation number which the agent reads back to the caller.

### Error responses

- `400 Bad Request` — invalid flight ID or malformed payload
- `404 Not Found` — flight no longer available

### Phonely configuration

- **Error Handling:** Enabled
- **Interim Message:** *"Confirming your reservation"*

---

## Variable interpolation — gotcha

Phonely's variable picker can produce two different states:

| What you see in the value field | What gets sent | Result |
|---|---|---|
| `{{departure_code}}` as **plain text** | Literal `{{departure_code}}` string | ❌ API rejects with `"Invalid airport code(s)"` |
| `{{` + pill + `}}` (typed braces around a pill) | `{{JFK}}` (extra braces wrap the value) | ❌ API rejects with `"Invalid airport code(s)"` |
| **Just the pill alone, no braces** | `JFK` | ✅ Works |

**Fix:** After inserting a variable via the picker, delete any surrounding `{{` and `}}` text. The field must contain only the pill chip.

---

## Test cases verified

| Inputs | Expected | Verified |
|---|---|---|
| `JFK → LAX → 2026-08-15` | 5 flights returned | ✅ |
| `AAL → YVR → 2026-08-15` | Empty flights array (no flights) | ✅ |
| `JFK → LAX → 2025-12-11` | 400 — date in past | ✅ |
| `XXX → LAX → 2026-08-15` | 400 — invalid airport code | ✅ |
