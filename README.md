# Cooren — Coordination Engine

**The authority gate for coordinated and agentic decisions.**
By McLeod Interactive Group LLC · [cooren.dev](https://cooren.dev)

---

## What is Cooren?

Cooren is a coordination primitive with one defining property:

> **Consensus is necessary, but never sufficient.**

Every coordinated decision has the same shape — options are defined, participants signal preferences, a tally aggregates them. Most systems stop there: agreement is reached, the action fires. Cooren inserts one thing between agreement and action — a **default-off authority gate**. The system can fully agree and still **hold**, taking no action, until a designated authority opens the gate.

Six operations run the complete loop. No UI required. The gate is the point.

---

## The gate, in one line

```
S(t+1) = G(t)·A(S(t), I(t)) + (1 − G(t))·S(t)
```

- `S` — system state · `I` — aggregated participant signals · `A` — arbitration function
- `G(t)` — the **authority gate**. Binary. **Default-off. Set from outside the system.**

When `G = 1`, the arbitration result becomes the new state. When `G = 0`, the state **holds** — no quantity of consensus, confidence, or agreement moves it. Authority is the final, irreducible gate. Nothing executes without it.

---

## Why this matters now

As AI agents move from *suggesting* to *acting*, the dangerous failure mode is **authority capture**: a system of agents that reach internal agreement and then execute, with no boundary between agreement and action. A swarm that convinces itself is still just a swarm.

Cooren is the boundary. The same six operations that collect and aggregate human votes collect and aggregate **agent signals** — and the gate decides whether that aggregate is permitted to become a state change. Agents propose. The gate, held by a human or a pre-committed policy, disposes.

This is approval-gated execution as infrastructure — the control surface that emerging agent-governance requirements (and a builder's own caution) increasingly demand before an autonomous action fires.

---

## How to reach Cooren

The engine is one system of record with two live doors:

**REST API** — direct programmatic integration at `cooren.dev` using a `cr_live_` key from your dashboard. Endpoints below.

**MCP server** — `mcp.cooren.dev`, live over remote HTTP with **OAuth 2.1** and Dynamic Client Registration. The same six operations, exposed as tools (`session_create`, `add_options`, `add_participants`, `submit_signal`, `get_tally`, `record_decision`) for any MCP-capable agent or IDE — Claude Desktop, Cursor, Windsurf. Per-account keys and billing are wired through the OAuth flow automatically. See the [MCP Quickstart Guide](docs/CoorenMCP_Quickstart.md).

MCP is a transport into the same engine, not a second engine — one source of truth, one billing meter, one audit trail regardless of the door you enter through.

---

## The decision loop

```bash
# 1. Create a coordination session
POST /api/create_session.php
Authorization: Bearer cr_live_<key>
Body: { "title": "Approve carrier reroute?" }

# 2. Load the options
POST /api/add_options.php
Body: { "session_id": "...", "options": ["Hold", "Reroute", "Escalate"] }

# 3. Register participants (human or agent)
POST /api/add_participants.php
Body: { "session_id": "...", "participants": ["agent.dispatch", "agent.yard"] }

# 4. Participants submit signals
POST /api/submit_signal.php
Body: { "token": "cr_pt_<participant_token>", "option_id": "..." }

# 5. Read the tally
GET /api/get_tally.php?session_id=...

# 6. Authority opens the gate and records the decision
POST /api/record_decision.php
Body: { "authority_token": "cr_at_<token>", "option_id": "..." }
```

Step 6 is the gate. Steps 1–5 can complete with total agreement and **nothing happens** until an authority token is presented.

---

## Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST   | `/api/create_session.php`   | Initialize a coordination event |
| POST   | `/api/add_options.php`      | Load decision options |
| POST   | `/api/add_participants.php` | Register participants (human or agent) |
| POST   | `/api/submit_signal.php`    | Submit a signal |
| GET    | `/api/get_tally.php`        | Retrieve the live tally |
| POST   | `/api/record_decision.php`  | Open the gate, close the session, stamp the outcome |
| GET    | `/api/keys.php`             | List API keys |
| POST   | `/api/keys.php`             | Generate a new key |
| DELETE | `/api/keys.php`             | Revoke a key |

---

## Architecture

**Zero-trust auth.** Single-use cryptographic participant tokens. Authority tokens gate session control *from outside* the participant set. No static passwords, no persistent session-hijacking vectors. A participant cannot authorize itself.

**Stateless logic.** Listen → aggregate → gate → close. Every request is self-contained. The engine computes the state transition and dissolves the transient memory space, scaling horizontally with no state bloat.

**Substrate invariance.** The six operations are not bound to one runtime. The same primitive is implemented independently as a **PHP web API** (this repo, live in production) and as **C++ firmware** targeting AVR and RP2350-class microcontrollers — separate codebases, identical operations. It also serves as the coordination substrate for **multi-agent orchestration**, demonstrated in Project Jeda: domain-bounded reasoners signaling through a live Cooren session under human authority. The logic loop is decoupled from the compute layer.

---

## Proven in production

Cooren's coordination loop was extracted from **[The Dinner Decider](https://thedinnerdecider.net)** — a deployed consumer app on **Google Play** and the **Apple App Store**. The loop that runs family meal voting every night is the same primitive Cooren exposes as infrastructure.

The loop is proven in production. Cooren generalizes it.

---

## Pricing — pay at the gate

Free coordination, paid consequence. Creating sessions, loading options, registering participants, submitting signals, and reading tallies are unlimited and free — always, for everyone, no tiers, no plan to pick. The only billable event is `record_decision` — the moment a decision becomes binding and the loop closes.

**$0.03 per decision. That's the whole price list.**

No included-call allowances to track, no overage rates, no plan switching. Sign up, get a key, start coordinating for free, and pay only when a decision actually closes. Signup, key management, usage visibility, and billing are all self-serve — no contact required.

Everything before the gate is exploration. Everything after it is consequence, and that's what's guaranteed: security, transparency, and an auditable chain of custody on every decision recorded.

---

## Roadmap — the gate with teeth

In software, the gate is enforced by code running in the **same trust domain** as the participants. That is sufficient for cooperative settings, but a determined participant could, in principle, reach the rules that bind it.

The hardware track closes that gap. On the **Raspberry Pi Pico 2 (RP2350)**, the design places the gate in **privileged mode behind a hardware MPU boundary**, with participant tasks running unprivileged. A participant that attempts to force a decision directly takes a hardware fault — the rules it is governed by are physically out of its reach. That boundary is the layer software-only orchestration cannot replicate.

**Status: specified, not yet built.** The coordination loop runs today in production across web and API surfaces; the hardware privilege boundary is future work. Claims here are held to exactly that line.

---

## Use cases

- **Agentic systems** — a human or policy authority gating actions proposed by autonomous agents. Agreement among agents never executes on its own.
- **Logistics** — load tendering with verified carrier acceptance; machine-to-machine consensus across dispatch nodes, gated by a dispatcher.
- **Enterprise** — asynchronous decision workflows across time zones. Stakeholders signal; an authority closes. No meeting required.
- **Consumer** — group decision apps. Dinner, movies, travel — any multi-party choice with a human authority.
- **Embedded / IoT** — signal-driven coordination across machine nodes without a central scheduler.

---

## Status

- **Live** — the six coordination endpoints at `cooren.dev`, with self-serve signup, key management, and metered billing ($0.03 per `record_decision`)
- **Live** — MCP server at `mcp.cooren.dev` (remote HTTP, OAuth 2.1, per-account billing)
- **Live** — The Dinner Decider reference implementation (Google Play, App Store)
- **Live** — Project Jeda multi-agent orchestration demo ([cooren.dev/project/jeda](https://cooren.dev/project/jeda))
- **In development** — Node.js SDK, expanded docs
- **Roadmap** — RP2350 hardware privilege boundary (specified, not yet built)

---

## About

Built and maintained by **McLeod Interactive Group LLC** — Jacksonville, Florida.

> *"The bottleneck is always the gap between request and decision. Cooren closes that gap — and decides, deliberately, when to."*

*Hold Fast · Find A Way · Shine Brightly*

© 2026 McLeod Interactive Group LLC. Proprietary software. Commercial use requires a license — see `LICENSE`.
