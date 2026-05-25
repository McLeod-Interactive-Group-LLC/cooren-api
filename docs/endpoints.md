# Endpoint Reference

Base URL: `https://cooren.dev`

All endpoints require `Authorization: Bearer cr_live_<key>` unless noted.
All request bodies are JSON. All responses are JSON.

---

## POST /api/create_session.php

Initialize a new coordination session.

### Request
```json
{
  "title": "Where should we eat tonight?"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | No | Session label. Defaults to "New Coordination Session". Max 255 chars. |

### Response `200`
```json
{
  "success": true,
  "data": {
    "session_id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
    "title": "Where should we eat tonight?",
    "authority_token": "5ad40402b7b98f002b66816bd5852359",
    "share_url": "https://cooren.dev/join/5e3e491fa6612f40621ac8f8c0e64f55362e"
  }
}
```

⚠️ Store `authority_token` immediately. It is not recoverable.

---

## POST /api/add_options.php

Add options to an existing open session.

### Request
```json
{
  "session_id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
  "options": [
    { "label": "Pizza", "sort_order": 1 },
    { "label": "Sushi", "sort_order": 2 },
    { "label": "Tacos", "sort_order": 3 }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `session_id` | string | Yes | Session to add options to |
| `options` | array | Yes | Minimum 2 items |
| `options[].label` | string | Yes | Display name. Max 255 chars. |
| `options[].description` | string | No | Optional detail |
| `options[].metadata` | object | No | Arbitrary key/value pairs |
| `options[].sort_order` | int | No | Display order |

### Response `200`
```json
{
  "success": true,
  "message": "3 options added.",
  "options": [
    { "id": "95a8e5a69471b661", "label": "Pizza" },
    { "id": "a85b449f29e56b71", "label": "Sushi" },
    { "id": "6c2a71a6da2fbeca", "label": "Tacos" }
  ]
}
```

---

## POST /api/add_participants.php

Invite participants to a session. Each participant receives a unique token for voting.

### Request
```json
{
  "session_id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
  "participants": [
    { "identifier": "alice@example.com", "display_name": "Alice" },
    { "identifier": "bob@example.com", "display_name": "Bob" }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `session_id` | string | Yes | Target session |
| `participants` | array | Yes | Minimum 1 item |
| `participants[].identifier` | string | No | Email or user ID for your reference |
| `participants[].display_name` | string | No | Display name |

### Response `200`
```json
{
  "success": true,
  "message": "2 participant(s) added.",
  "participants": [
    {
      "id": "0ca279d4043daebe943a02f8cb4d7c8c4ad2",
      "identifier": "alice@example.com",
      "display_name": "Alice",
      "token": "670ade11c4ef204bcbb2fc4445717ca3",
      "token_expires": "2026-05-24 17:16:41"
    }
  ]
}
```

Distribute each participant's `token` and `id` to them via your own notification system. Tokens expire after 24 hours.

---

## POST /api/submit_signal.php

Submit a participant's vote for an option.

### Request
```json
{
  "session_id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
  "participant_id": "0ca279d4043daebe943a02f8cb4d7c8c4ad2",
  "option_id": "95a8e5a69471b661",
  "token": "670ade11c4ef204bcbb2fc4445717ca3"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `session_id` | string | Yes | Active session |
| `participant_id` | string | Yes | Participant's ID from add_participants |
| `option_id` | string | Yes | Option ID from add_options |
| `token` | string | Yes | Participant's token |

### Response `200`
```json
{
  "success": true,
  "message": "Signal submitted.",
  "signal_id": "665403effa027ce1fb91f5ed28f2148f9763"
}
```

### Error `409`
```json
{
  "success": false,
  "error": "Participant has already submitted a signal"
}
```

Each participant may submit exactly one signal per session.

---

## GET /api/get_tally.php

Retrieve live vote counts for a session.

### Request
```
GET /api/get_tally.php?session_id=5e3e491fa6612f40621ac8f8c0e64f55362e
```

### Response `200`
```json
{
  "success": true,
  "session": {
    "id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
    "title": "Where should we eat tonight?",
    "status": "open"
  },
  "participation": {
    "total_participants": 2,
    "total_signals": 2,
    "pending": 0
  },
  "tally": [
    { "option_id": "95a8e5a69471b661", "label": "Pizza", "vote_count": 1 },
    { "option_id": "a85b449f29e56b71", "label": "Sushi", "vote_count": 1 },
    { "option_id": "6c2a71a6da2fbeca", "label": "Tacos", "vote_count": 0 }
  ]
}
```

Results are ordered by `vote_count` descending. Call this endpoint at any time — sessions remain open until `record_decision` is called.

---

## POST /api/record_decision.php

Close the session and stamp the winning option. Requires the authority token.

### Request
```json
{
  "session_id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
  "option_id": "95a8e5a69471b661",
  "authority_token": "5ad40402b7b98f002b66816bd5852359",
  "note": "Tie broken by authority. Pizza it is."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `session_id` | string | Yes | Session to close |
| `option_id` | string | Yes | The winning option |
| `authority_token` | string | Yes | Token from create_session |
| `note` | string | No | Optional decision rationale |

### Response `200`
```json
{
  "success": true,
  "message": "Decision recorded. Session closed.",
  "decision": {
    "id": "d9c7d96f350028c5c1787a1af891522ce8da",
    "session_id": "5e3e491fa6612f40621ac8f8c0e64f55362e",
    "session": "Where should we eat tonight?",
    "winner": "Pizza",
    "option_id": "95a8e5a69471b661",
    "note": "Tie broken by authority. Pizza it is.",
    "decided_at": "2026-05-23T19:24:26Z"
  }
}
```

Once closed, a session cannot be reopened. The status changes to `closed` and no further signals are accepted.

---

## Key Management

### GET /api/keys.php — List keys
```bash
curl https://cooren.dev/api/keys.php \
  -H "Authorization: Bearer cr_live_<key>"
```

### POST /api/keys.php — Generate key
```json
{
  "name": "Production Key",
  "environment": "production"
}
```

### DELETE /api/keys.php — Revoke key
```json
{
  "key_id": 3
}
```

---

© 2026 McLeod Interactive Group LLC
