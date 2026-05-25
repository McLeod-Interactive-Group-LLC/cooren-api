# Example — Dinner Decider

This is the original use case that Cooren was extracted from. The Dinner Decider app on Google Play and the Apple App Store uses this exact coordination loop to help families decide where to eat.

---

## The Problem

A family of four wants to decide where to eat. Everyone has opinions. Nobody agrees. The usual result is 20 minutes of "I don't care, where do you want to go?" followed by someone making a unilateral decision that satisfies nobody.

Cooren solves this in six API calls.

---

## The Flow

```
Parent (Authority)         Cooren API              Family Members
      |                        |                          |
      |-- create_session ----→ |                          |
      |← session_id + auth_token                          |
      |                        |                          |
      |-- add_options --------→|                          |
      |   ["Pizza","Sushi",    |                          |
      |    "Tacos","Burgers"]  |                          |
      |                        |                          |
      |-- add_participants ---→|                          |
      |                        |-- token → -----------→  |
      |                        |-- token → -----------→  |
      |                        |-- token → -----------→  |
      |                        |                          |
      |                        |← submit_signal ----------|
      |                        |← submit_signal ----------|
      |                        |← submit_signal ----------|
      |                        |                          |
      |-- get_tally ----------→|                          |
      |← Pizza: 2, Sushi: 1    |                          |
      |                        |                          |
      |-- record_decision ----→|                          |
      |   winner: Pizza        |                          |
      |← decision stamped      |                          |
```

---

## Implementation

### 1. Parent creates the session

```bash
curl -X POST https://cooren.dev/api/create_session.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{"title": "Where are we eating tonight?"}'
```

```json
{
  "success": true,
  "data": {
    "session_id": "abc123...",
    "authority_token": "def456...",
    "share_url": "https://cooren.dev/join/abc123..."
  }
}
```

### 2. Load tonight's options

```bash
curl -X POST https://cooren.dev/api/add_options.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "abc123...",
    "options": [
      {"label": "Pizza Palace", "sort_order": 1},
      {"label": "Sakura Sushi", "sort_order": 2},
      {"label": "Taco Town", "sort_order": 3},
      {"label": "Burger Barn", "sort_order": 4}
    ]
  }'
```

### 3. Invite the family

```bash
curl -X POST https://cooren.dev/api/add_participants.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "abc123...",
    "participants": [
      {"display_name": "Mom"},
      {"display_name": "Kid 1"},
      {"display_name": "Kid 2"}
    ]
  }'
```

Your app sends each participant their unique voting link containing their `participant_id` and `token`.

### 4. Family members vote

Each family member taps their link, sees the options, and submits their choice:

```bash
curl -X POST https://cooren.dev/api/submit_signal.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "abc123...",
    "participant_id": "mom_id...",
    "option_id": "pizza_id...",
    "token": "mom_token..."
  }'
```

### 5. Parent checks the tally

```bash
curl "https://cooren.dev/api/get_tally.php?session_id=abc123..." \
  -H "Authorization: Bearer cr_live_<key>"
```

```json
{
  "participation": { "total_participants": 3, "total_signals": 3, "pending": 0 },
  "tally": [
    { "label": "Pizza Palace", "vote_count": 2 },
    { "label": "Sakura Sushi", "vote_count": 1 },
    { "label": "Taco Town", "vote_count": 0 },
    { "label": "Burger Barn", "vote_count": 0 }
  ]
}
```

### 6. Parent closes the loop

```bash
curl -X POST https://cooren.dev/api/record_decision.php \
  -H "Authorization: Bearer cr_live_<key>" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "abc123...",
    "option_id": "pizza_id...",
    "authority_token": "def456...",
    "note": "Pizza wins 2-1!"
  }'
```

---

## The Real Product

This example runs live inside [Dinner Decider](https://thedinnerdecider.net), available on Google Play and the Apple App Store.

Every family vote is a Cooren session. The app provides the UI — swipe cards, meal history, family profiles — but the coordination primitive underneath is exactly what you see here.

That same primitive works at any scale, for any decision, in any domain.

---

© 2026 McLeod Interactive Group LLC
