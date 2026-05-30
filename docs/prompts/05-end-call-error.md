# Block 5 — End Calls (Error Paths)

Two error End Call blocks handle different failure modes.

---

## 5a) No flights available

**Phonely block type:** End Call
**Fires:** After GET API Request → Error branch (e.g. the AAL → YVR test case)

### Prompt (exact text)

```
The flight search returned no available flights for the cities and date the caller provided. Politely tell them no flights are available for that route on that date, suggest they try a different route or date, and end the call with "Thank you for calling BlueWing Phonely Airlines, goodbye."
```

### Design notes

- **Acknowledges the specific failure** ("no flights for that route on that date") so the caller understands what went wrong.
- **Offers a constructive next step** ("try a different route or date") instead of just hanging up.
- **Branded sign-off** maintains professionalism even on the error path.

---

## 5b) Booking failed (technical issue)

**Phonely block type:** End Call
**Fires:** After POST API Request → Error branch (booking API failure after a valid flight was selected)

### Prompt (exact text)

```
The booking API failed when trying to book the flight. Apologize politely, tell the caller there was a technical issue and the booking could not be completed, suggest they try calling again later, and end with "Thank you for calling BlueWing Phonely Airlines, goodbye."
```

### Design notes

- **Apology is explicit** because the caller already provided all their info — failing at the last step is the worst place to fail.
- **Frames it as a technical issue** (not the caller's fault) — preserves trust.
- **"Try again later"** sets expectation that the issue is transient.
- **Same branded sign-off** for consistency across all End Call branches.
