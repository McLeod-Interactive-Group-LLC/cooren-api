# Cooren — Coordination Engine

**Decision-as-a-Service API by McLeod Interactive Group LLC**

[![Status](https://img.shields.io/badge/status-private%20beta-amber)](https://cooren.dev)
[![License](https://img.shields.io/badge/license-proprietary-red)](./LICENSE)
[![API Version](https://img.shields.io/badge/API-v1.0-blue)](https://cooren.dev)

---

## What is Cooren?

Cooren is a backend Coordination Engine API that automates multi-party decision-making across any domain.

Whether a family choosing dinner or a logistics hub tendering a load — the bottleneck is always the same: **the gap between request and decision.**

Cooren closes that gap. Six endpoints. One complete decision loop. No UI required.

---

## The Problem Cooren Solves

Every coordinated decision shares the same structure:

1. Someone defines options
2. Stakeholders signal preferences
3. An authority closes the loop

The infrastructure to do this reliably — auth, state management, tally logic, billing — gets rebuilt from scratch in every application that needs it. Cooren abstracts that infrastructure into a clean API so developers never build it again.

---

## Core Architecture

### Zero-Trust Auth
Single-use cryptographic participant tokens. Authority tokens gate session control. No passwords, no sessions.

### Stateless Logic
Listen-Act-Close architecture. Each request is self-contained. Scales horizontally with zero session state.

### Credit-as-a-Service
Stripe-native billing. Free tier included. Upgrade via API. Usage tracked per key, per operation.

---

## The Decision Loop

```http
# 1. Create a coordination session
POST /api/create_session.php
Authorization: Bearer cr_live_<key>
Body: { "title": "Where should we eat?" }

# 2. Load the options
POST /api/add_options.php
Body: { "session_id": "...", "options": ["Pizza", "Sushi", "Tacos"] }

# 3. Invite participants
POST /api/add_participants.php
Body: { "session_id": "...", "participants": ["alice@example.com"] }

# 4. Participants vote
POST /api/submit_signal.php
Body: { "token": "cr_pt_<participant_token>", "option_id": "..." }

# 5. Check the tally
GET /api/get_tally.php?session_id=...

# 6. Record the decision
POST /api/record_decision.php
Body: { "authority_token": "cr_at_<token>", "option_id": "..." }
```

---

## Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/api/create_session.php` | Initialize coordination event |
| `POST` | `/api/add_options.php` | Load decision options |
| `POST` | `/api/add_participants.php` | Invite participants |
| `POST` | `/api/submit_signal.php` | Cast a vote / signal |
| `GET` | `/api/get_tally.php` | Retrieve live results |
| `POST` | `/api/record_decision.php` | Close session, stamp winner |
| `GET` | `/api/keys.php` | List API keys |
| `POST` | `/api/keys.php` | Generate new key |
| `DELETE` | `/api/keys.php` | Revoke key |

---

## Use Cases

**Consumer**
Family or group decision apps. Dinner, movies, travel. Any multi-party choice with a human authority.

**Logistics**
Load tendering with verified carrier acceptance. Instantaneous M2M consensus across dispatch nodes.

**Enterprise**
Async decision workflows across time zones. No meetings. Stakeholders signal, authority closes.

**Manufacturing**
Decentralized task allocation across machine nodes. Signal-driven orchestration without a central scheduler.
- **Embedded / Physical** — [CoorenIO Wokwi simulation](examples/coorenio-wokwi/README.md) (Arduino Mega + DHT22 sensors + damper control)

---

## Proof of Concept

Cooren's coordination loop was extracted from [Dinner Decider](https://thedinnerdecider.net) — a deployed consumer app available on Google Play and the Apple App Store. The decision loop powering family meal voting is the same primitive that powers any coordinated decision at any scale.

The loop is proven in production. Cooren exposes it as infrastructure.

---

## Pricing

| Tier | Price | Signals/month |
|------|-------|---------------|
| Free | $0 | 100 |
| Starter | $49/mo | 10,000 |
| Growth | $149/mo | 100,000 |
| Scale | $499/mo | 1,000,000 |
| Enterprise | $1,499/mo | Unlimited + SLA |

---

## Status

Cooren is currently in **private beta** for select partners.

- ✅ API endpoints live
- ✅ Stripe billing connected
- ✅ Free tier available — no credit card required to start
- 🔄 Documentation expanding
- 🔄 SDK in development

---

## Request Access

**[cooren.dev](https://cooren.dev)**

Contact: [contact@cooren.dev](mailto:contact@cooren.dev)

---

## About

Cooren is built and maintained by **McLeod Interactive Group LLC**, Jacksonville, Florida.

> *"The bottleneck is always the gap between request and decision. Cooren closes that gap."*

---

© 2026 McLeod Interactive Group LLC. All rights reserved.
Proprietary software. Commercial use requires a license.
See [LICENSE](./LICENSE) for details.
