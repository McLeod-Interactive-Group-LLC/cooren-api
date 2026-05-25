# Cooren SDKs

Official client libraries for the Cooren Coordination Engine API.

---

## Status

| SDK | Status | Install |
|-----|--------|---------|
| PHP | 🔄 In Development | — |
| Node.js | 🔄 In Development | — |
| Python | 📋 Planned | — |
| Go | 📋 Planned | — |

---

## Until the SDKs Ship

The Cooren API is a clean REST API over HTTPS. Any HTTP client works today:

**PHP**
```php
$ch = curl_init('https://cooren.dev/api/create_session.php');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(['title' => 'My Session']));
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: Bearer cr_live_<your_key>',
    'Content-Type: application/json',
]);
$response = json_decode(curl_exec($ch), true);
```

**Node.js**
```javascript
const response = await fetch('https://cooren.dev/api/create_session.php', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer cr_live_<your_key>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ title: 'My Session' }),
});
const data = await response.json();
```

**Python**
```python
import requests

response = requests.post(
    'https://cooren.dev/api/create_session.php',
    headers={'Authorization': 'Bearer cr_live_<your_key>'},
    json={'title': 'My Session'}
)
data = response.json()
```

**cURL**
```bash
curl -X POST https://cooren.dev/api/create_session.php \
  -H "Authorization: Bearer cr_live_<your_key>" \
  -H "Content-Type: application/json" \
  -d '{"title": "My Session"}'
```

---

## SDK Notifications

Want to be notified when a specific SDK ships?

Contact: [contact@cooren.dev](mailto:contact@cooren.dev)

---

© 2026 McLeod Interactive Group LLC
