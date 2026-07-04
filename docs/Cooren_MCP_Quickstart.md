# Cooren Coordination Engine — MCP Server

Secure multi-party decision loops with built-in authority gating.

**Server URL:** `https://mcp.cooren.dev/mcp`

---

## Authentication — OAuth, Not API Keys

Cooren MCP uses OAuth 2.1 with Dynamic Client Registration. No static keys are used or accepted on the MCP interface. If you have a `cr_live_` key from the [cooren.dev](https://cooren.dev) dashboard, that key is for the **REST API only** — it will not work here, and the server will reject it with a 401.

Connecting is a one-time consent flow, not a config edit:

1. Add the server URL to your client (see configs below) — **no Authorization header needed.**
2. Your client detects the server requires OAuth and opens a browser window automatically.
3. You'll see a consent screen asking you to authorize the connection.
4. Click **Allow** (or **Consent**).
5. The browser redirects back and your client shows the connection as active.

That's it. Your client stores the token and reconnects automatically for 30 days — no re-authorizing needed until it expires.

---

## Quickstart Configurations

### Claude Desktop / Claude Code

```json
{
  "mcpServers": {
    "cooren": {
      "url": "https://mcp.cooren.dev/mcp"
    }
  }
}
```

Add this, then restart or reload the client. It will prompt you through the consent screen on first connection — no header required.

### Cursor / Windsurf / VS Code

Add to your settings or `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "cooren": {
      "url": "https://mcp.cooren.dev/mcp"
    }
  }
}
```

Same flow: the client will trigger the browser consent screen the first time it tries to connect.

### Command Line

```bash
claude mcp add --transport http cooren https://mcp.cooren.dev/mcp
```

No `--header` flag — omitting it lets the OAuth flow trigger normally on first use.

---

## Billing

Coordination is free and unlimited: creating a session, adding participants, submitting signals, and reading the tally cost nothing. You pay **$0.03** only when an authority opens the gate to record a final decision.

This applies regardless of which credential system you're using — REST API key or MCP OAuth token — since both route through the same gate-billing logic on `record_decision`.

---

## Available Tools

| Tool | Description | Billable? |
|---|---|---|
| `session_create` | Open a coordination session with options | No |
| `add_participants` | Register participants to a session | No |
| `submit_signal` | Submit a participant's signal | No |
| `get_tally` | Read the aggregated tally | No |
| `record_decision` | Authority opens the gate, binds the final decision | **Yes ($0.03)** |

---

## Quick Test Prompt

Once connected, try:

> Use Cooren to decide on "Favorite color" with options Red, Blue, Green. Create the session, add two participants, show the tally, and describe how to open the gate.

You don’t need to use the exact tool name. Natural language like “create a session”, “start a new decision”, or “run a coordination loop” is enough — your client’s LLM will automatically call session_create (and the other tools) on your behalf.

**A note on naming, since it looks like a typo but isn't:** the MCP tool is `session_create` (verb-second, matching `add_participants`, `submit_signal`, `get_tally`, `record_decision`). The REST API's equivalent endpoint is `create-session.php` (verb-first, standard REST convention). These are two different naming conventions for two different substrates — not an inconsistency, and not interchangeable. If you're integrating directly against one or the other, use the name that matches that system.

---

## Troubleshooting

- **OAuth consent screen doesn't appear:** Remove the server from your client and re-add it — some clients cache a failed handshake.
- **401 errors:** Confirm you have not added an `Authorization` header. The MCP server rejects `cr_live_` REST keys outright; omit the header entirely and let the OAuth flow trigger.
- **Tools not showing up after connecting:** Restart your client. Most MCP clients only refresh their tool list on launch or reconnect.

---

## Note on the REST API

If you're building a direct integration rather than using an MCP-compatible client, the REST API at `cooren.dev/api/` uses a separate `cr_live_` Bearer key from your dashboard and has its own middleware. The two systems are not interchangeable — pick the one that matches your client.

---

*McLeod Interactive Group LLC · Jacksonville, Florida*
*Hold Fast · Find A Way · Shine Brightly*