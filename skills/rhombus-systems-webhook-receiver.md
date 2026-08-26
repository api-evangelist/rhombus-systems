---
name: rhombus-webhook-receiver
description: Scaffold a webhook listener for Rhombus events in Express (Node.js), FastAPI (Python), or AWS Lambda. Use whenever the user asks to build a webhook endpoint, event listener, or receiver for Rhombus alerts, door events, LPR matches, or any Rhombus-triggered automation. Also trigger on phrases like "wire up a webhook", "receive Rhombus events", "PagerDuty integration", "Slack notifier for Rhombus", "alert webhook", or anything about handling Rhombus event payloads. Covers signature verification, idempotency, and known payload shapes.
argument-hint: "[express|fastapi|lambda] [feature]"
---

# Rhombus Webhook Receiver

Scaffold a production-ready webhook listener for Rhombus events. The scaffold includes signature verification, idempotency, and structured logging — the three things every Rhombus webhook listener needs.

## Supported targets

| Flag | Stack | Output |
|---|---|---|
| `express` | Node.js + Express + TypeScript | `index.ts`, `verify.ts`, `dedupe.ts`, `handlers/*.ts`, `package.json`, `.env.example` |
| `fastapi` | Python 3.11+ + FastAPI + uvicorn | `app.py`, `verify.py`, `dedupe.py`, `handlers/`, `pyproject.toml`, `.env.example` |
| `lambda` | AWS Lambda + API Gateway | `handler.py` or `handler.ts`, `template.yaml` (SAM), `.env.example` |

## Scaffolded components

Every scaffold includes these files:

1. **HTTP entry point** — receives `POST /webhook` with a raw body reader (critical for signature verification).
2. **Signature verifier** — HMAC-SHA256 of the raw body using the webhook secret from env, constant-time compared to the `X-Rhombus-Signature` header.
3. **Idempotency layer** — dedupes by event UUID using an in-memory LRU for dev; swap to Redis/DynamoDB for prod.
4. **Event router** — switches on event type and dispatches to a handler. Default handlers are stubs that log the payload.
5. **Structured logger** — logs every attempt with event UUID, type, signature-ok flag, duplicate flag, outcome.

## Known payload shapes

See `references/webhook-payloads.md` for sample JSON for each event category:

- Camera events (motion, person detection, custom analytics)
- Alert events (with linked `alertUuid`, `cameraUuid`, and media URIs)
- Door events (door unlocked/locked, access granted/denied)
- LPR / vehicle events (with matched plate text and confidence)
- User / access control events (credential issued, granted, revoked)
- IoT sensor events (threshold violations)

**Important:** payload shapes vary by event type. Identifier fields differ — don't assume every event has `cameraUuid`. The reference sheet shows the minimum required fields per type.

## Signature verification — the pitfall

Rhombus computes the signature over the **raw request body bytes**, not a re-serialized JSON. If your framework auto-parses JSON before you can access the raw body, signature verification will fail on any payload with non-canonical whitespace.

Framework-specific fixes:

- **Express:** use `express.raw({ type: 'application/json' })` on the webhook route; parse JSON *after* verification.
- **FastAPI:** use `await request.body()` (returns bytes) before `await request.json()`.
- **Lambda:** use the raw `event.body` (base64-decode if `isBase64Encoded`).

## Idempotency

Rhombus may retry a delivery if your listener:
- Returns a non-2xx status.
- Takes >5s to respond.
- Times out the TCP connection.

Always:
1. Dedupe by the event UUID (top-level `uuid` or `eventUuid`).
2. Respond 2xx within 1s — enqueue heavy work to a background worker.

## Next steps after scaffolding

1. Set `RHOMBUS_WEBHOOK_SECRET` in your `.env` (obtained when creating the webhook via the Developer Webservice or `mcp__rhombus__*` tools).
2. Expose your listener publicly (ngrok/cloudflared for dev, a real host for prod).
3. Register the webhook with Rhombus, pointing at your URL, selecting the event types you care about.
4. Send a test event and confirm you see it in the logs with `sig_ok=true, duplicate=false`.
5. Fill in handler logic for each event type you actually care about.

If deliveries aren't arriving, invoke the `rhombus-webhook-debugger` agent.
