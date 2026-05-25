# Example — Logistics Load Tendering

This example demonstrates Cooren applied to freight load tendering — a multi-party coordination problem where a shipper needs carriers to accept or decline a load, and the first qualified acceptance wins.

---

## The Problem

A logistics hub needs to tender a load to carriers. The traditional process involves phone calls, emails, and load boards — high friction, slow, and error-prone. Every minute a load sits untendered is money left on the table.

Cooren replaces that process with a structured coordination loop.

---

## The Flow

```
Dispatcher (Authority)     Cooren API              Carriers
      |                        |                       |
      |-- create_session ----→ |                       |
      |   "Load #TX-4892"      |                       |
      |← session_id + auth_token                       |
      |                        |                       |
      |-- add_options --------→|                       |
      |   [Accept, Decline,    |                       |
      |    Counter-Offer]      |                       |
      |                        |                       |
      |-- add_participants ---→|                       |
      |   [Carrier A, B, C]    |-- token → ----------→ |
      |                        |                       |
      |                        |← submit_signal -----  |
      |                        |  Carrier A: Accept    |
      |-- get_tally ----------→|                       |
      |← Accept: 1             |                       |
      |                        |                       |
      |-- record_decision ----→|                       |
      |   Carrier A awarded    |                       |
```

---

## Implementation

### 1. Dispatcher creates the load session

```bash
curl -X POST https://cooren.dev/api/create_session.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{"title": "Load #TX-4892 — Dallas to Memphis, 42000 lbs dry van, pickup 0600"}'
```

### 2. Define the response options

```bash
curl -X POST https://cooren.dev/api/add_options.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "...",
    "options": [
      {
        "label": "Accept — $2,400",
        "description": "Full rate acceptance",
        "metadata": {"rate": 2400, "action": "accept"},
        "sort_order": 1
      },
      {
        "label": "Counter — $2,600",
        "description": "Counter-offer at higher rate",
        "metadata": {"rate": 2600, "action": "counter"},
        "sort_order": 2
      },
      {
        "label": "Decline",
        "description": "Not available for this load",
        "metadata": {"action": "decline"},
        "sort_order": 3
      }
    ]
  }'
```

### 3. Tender to qualified carriers

```bash
curl -X POST https://cooren.dev/api/add_participants.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "...",
    "participants": [
      {"identifier": "carrier-001", "display_name": "Swift Transport"},
      {"identifier": "carrier-002", "display_name": "Heartland Freight"},
      {"identifier": "carrier-003", "display_name": "Lone Star Carriers"}
    ]
  }'
```

Your system sends each carrier their unique response link via TMS, SMS, or email.

### 4. Carrier responds

```bash
curl -X POST https://cooren.dev/api/submit_signal.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "...",
    "participant_id": "swift_id...",
    "option_id": "accept_id...",
    "token": "swift_token..."
  }'
```

### 5. Dispatcher monitors in real time

```bash
curl "https://cooren.dev/api/get_tally.php?session_id=..." \
  -H "Authorization: Bearer cr_live_<key>"
```

Your TMS polls this endpoint and surfaces the first acceptance to the dispatcher immediately.

### 6. Award the load

```bash
curl -X POST https://cooren.dev/api/record_decision.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "...",
    "option_id": "accept_id...",
    "authority_token": "...",
    "note": "Awarded to Swift Transport — first acceptance at $2,400"
  }'
```

---

## What This Replaces

| Old Process | Cooren Process |
|-------------|----------------|
| Phone calls to each carrier | Single API call creates session |
| Email chains with no audit trail | Every signal timestamped and stored |
| Manual tracking in spreadsheets | `get_tally` returns live state |
| Dispatcher memory for who accepted | `record_decision` stamps the winner permanently |
| 20-45 minutes per load | Under 5 minutes from tender to award |

---

## Extensions

- **Auto-award**: Poll `get_tally` and call `record_decision` automatically when the first acceptance arrives
- **Rate negotiation**: Use counter-offer options and multiple rounds
- **Carrier scoring**: Pass carrier performance metadata in the options `metadata` field
- **Audit trail**: Every session, signal, and decision is stored — full compliance history

---

© 2026 McLeod Interactive Group LLC
