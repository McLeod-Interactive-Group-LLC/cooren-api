# Authentication

Cooren uses a custom high-security API key system modeled after industry standards like Stripe and GitHub.

---

## API Key Format

Every Cooren API key follows this format:

```
cr_live_[32 hex characters]
```

Example:
```
cr_live_4b70840b5682c653e72aa951345a082c
```

Keys are issued per account and tied to a Stripe customer for billing purposes.

---

## Making Authenticated Requests

Pass your API key as a Bearer token in the `Authorization` header on every request:

```bash
curl -X POST https://cooren.dev/api/create_session.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{"title": "My First Session"}'
```

---

## Key Security Model

| Component | Detail |
|-----------|--------|
| **Format** | `cr_live_` prefix + 32 hex characters |
| **Prefix lookup** | First 16 hex chars stored in plaintext for O(1) database lookup |
| **Storage** | Full key is never stored — only a bcrypt hash |
| **Validation** | Prefix lookup → hash verification on every request |

Your raw API key is shown **exactly once** at generation. Copy it immediately and store it securely. It cannot be recovered.

---

## Participant Tokens

Participants in a session are issued single-use tokens when added via `add_participants.php`. These tokens:

- Are valid for 24 hours by default
- Authorize exactly one signal submission per session
- Cannot be reused after a signal is submitted
- Are separate from your API key — participants never see your key

---

## Authority Tokens

When you create a session, an `authority_token` is returned. This token:

- Grants control over the session
- Is required to call `record_decision.php`
- Should be stored server-side and never exposed to participants
- Cannot be regenerated — store it when the session is created

---

## Key Management

### Generate a new key
```bash
curl -X POST https://cooren.dev/api/keys.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{"name": "Production Key", "environment": "production"}'
```

### List your keys
```bash
curl https://cooren.dev/api/keys.php \
  -H "Authorization: Bearer cr_live_<your_key>"
```

### Revoke a key
```bash
curl -X DELETE https://cooren.dev/api/keys.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{"key_id": 3}'
```

---

## Error Responses

| HTTP Code | Meaning |
|-----------|---------|
| `401` | Missing or malformed Authorization header |
| `401` | Invalid API key |
| `401` | API key is inactive — check subscription status |
| `403` | Valid key but unauthorized for this resource |

All auth errors return JSON:
```json
{
  "error": "Invalid API key"
}
```

---

## Best Practices

- **Never expose your API key** in client-side code, mobile apps, or public repositories
- **Rotate keys** if you suspect compromise — revoke the old key immediately after generating a new one
- **Use separate keys** for development and production environments
- **Store authority tokens** server-side only — treat them like passwords
- **Set short token expiries** for participant tokens in high-security contexts

---

© 2026 McLeod Interactive Group LLC
