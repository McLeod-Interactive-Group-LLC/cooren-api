# Error Codes

All Cooren API errors return a consistent JSON structure with an appropriate HTTP status code.

---

## Error Response Format

```json
{
  "success": false,
  "error": "Human-readable error message"
}
```

Or for auth errors (no `success` field):
```json
{
  "error": "Invalid API key"
}
```

---

## HTTP Status Codes

| Code | Meaning | Common Causes |
|------|---------|---------------|
| `200` | Success | Request completed |
| `201` | Created | Resource created successfully |
| `400` | Bad Request | Missing or invalid fields in request body |
| `401` | Unauthorized | Missing, malformed, or invalid API key |
| `403` | Forbidden | Valid key but not authorized for this resource |
| `404` | Not Found | Resource does not exist |
| `405` | Method Not Allowed | Wrong HTTP method for this endpoint |
| `409` | Conflict | Duplicate action (e.g. participant voted twice) |
| `500` | Server Error | Internal error — contact support |

---

## Auth Errors `401`

| Message | Cause |
|---------|-------|
| `Missing or malformed Authorization header` | No `Authorization` header, or not in `Bearer <key>` format |
| `Invalid API key format` | Key does not match `cr_live_[32 hex chars]` pattern |
| `Invalid API key` | Key not found or hash mismatch |
| `API key is inactive` | Account subscription lapsed or key was revoked |

---

## Validation Errors `400`

| Message | Cause |
|---------|-------|
| `Missing session_id or options array` | Required fields absent in request body |
| `options must be an array with at least 2 items` | Fewer than 2 options provided |
| `Title must be 255 characters or fewer` | Field length exceeded |
| `Missing session_id, participant_id, option_id, or token` | Incomplete signal submission |
| `Invalid option for this session` | Option ID does not belong to this session |
| `Missing session_id, option_id, or authority_token` | Incomplete decision request |
| `environment must be sandbox or production` | Invalid environment value on key generation |

---

## Authorization Errors `403`

| Message | Cause |
|---------|-------|
| `Unauthorized or invalid session` | Session not found or belongs to a different account |
| `Unauthorized, invalid session, or bad authority token` | Authority token mismatch on record_decision |
| `Invalid or expired participant token` | Token not found, wrong session, or past expiry |
| `Session not found or is closed` | Attempting to signal a closed session |

---

## Conflict Errors `409`

| Message | Cause |
|---------|-------|
| `Participant has already submitted a signal` | One signal per participant per session |
| `Session is already closed` | record_decision called on a closed session |

---

## Handling Errors

```bash
# Example: handle a 401
curl -X POST https://cooren.dev/api/create_session.php \
  -H "Authorization: Bearer cr_live_invalidkey" \
  -H "Content-Type: application/json" \
  -d '{"title": "Test"}'

# Response:
# HTTP 401
# {"error":"Invalid API key"}
```

Always check the HTTP status code first, then parse the `error` field for the specific message.

---

## Support

If you receive a `500` error or encounter unexpected behavior:

- Email: [support@cooren.dev](mailto:support@cooren.dev)
- Include: your `session_id` or `account_id`, the endpoint called, and the full response body

---

© 2026 McLeod Interactive Group LLC
