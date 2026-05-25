# Quick Start — Your First Decision in 5 Minutes

This guide walks you through a complete Cooren decision loop from start to finish.

**Prerequisites:** A Cooren API key. Request one at [cooren.dev](https://cooren.dev).

---

## Step 1 — Create a Session

```bash
curl -X POST https://cooren.dev/api/create_session.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{"title": "Quick Start Test"}'
```

**Save from the response:**
- `session_id`
- `authority_token` ← store this securely, it controls the session

---

## Step 2 — Add Options

```bash
curl -X POST https://cooren.dev/api/add_options.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "<session_id>",
    "options": [
      { "label": "Option A", "sort_order": 1 },
      { "label": "Option B", "sort_order": 2 },
      { "label": "Option C", "sort_order": 3 }
    ]
  }'
```

**Save from the response:**
- Each `option.id` — you'll need these for voting

---

## Step 3 — Add Participants

```bash
curl -X POST https://cooren.dev/api/add_participants.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "<session_id>",
    "participants": [
      { "identifier": "alice@example.com", "display_name": "Alice" },
      { "identifier": "bob@example.com", "display_name": "Bob" }
    ]
  }'
```

**Save from the response:**
- Each participant's `id` and `token`
- Distribute these to your participants via your own notification system

---

## Step 4 — Submit Signals

Each participant casts their vote using their token:

```bash
# Alice votes for Option A
curl -X POST https://cooren.dev/api/submit_signal.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "<session_id>",
    "participant_id": "<alice_participant_id>",
    "option_id": "<option_a_id>",
    "token": "<alice_token>"
  }'

# Bob votes for Option B
curl -X POST https://cooren.dev/api/submit_signal.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "<session_id>",
    "participant_id": "<bob_participant_id>",
    "option_id": "<option_b_id>",
    "token": "<bob_token>"
  }'
```

---

## Step 5 — Check the Tally

```bash
curl "https://cooren.dev/api/get_tally.php?session_id=<session_id>" \
  -H "Authorization: Bearer cr_live_<your_key>"
```

Response shows live vote counts per option, total participants, and how many are still pending.

---

## Step 6 — Record the Decision

The authority closes the session and stamps the winner:

```bash
curl -X POST https://cooren.dev/api/record_decision.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "<session_id>",
    "option_id": "<winning_option_id>",
    "authority_token": "<authority_token>",
    "note": "Optional rationale for the decision"
  }'
```

The session is now closed. The decision is stamped with a timestamp and stored permanently.

---

## What Just Happened

In 6 API calls you:

1. Created an isolated coordination context
2. Loaded it with options
3. Invited participants with cryptographic tokens
4. Collected signals from each participant
5. Retrieved a live tally
6. Closed the loop with an authoritative decision

That's the Cooren primitive. Everything else — the UI, the notifications, the business logic — is yours to build on top.

---

## Next Steps

- [Authentication Guide](../docs/authentication.md)
- [Full Endpoint Reference](../docs/endpoints.md)
- [Dinner Decider Integration Example](./dinner-decider.md)
- [Load Tendering Example](./load-tendering.md)

---

© 2026 McLeod Interactive Group LLC
