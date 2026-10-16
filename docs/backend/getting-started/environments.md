---
description: How to tell what your environment can actually do, before you build on it.
---

# Environments

JET environments differ in one important way: how much of the rail is backed
by a real provider. Rather than making you guess, the API publishes it.

```bash
curl "$JET_API/capabilities"
```

```json
{
  "version": "1.4.2",
  "environment": "sandbox",
  "corridors": [
    { "from": "USDC", "to": "VND", "destinationCountry": "VN", "ownedRail": false, "instant": false },
    { "from": "USDC", "to": "NGN", "destinationCountry": "NG", "ownedRail": false, "instant": false }
  ],
  "features": [
    { "feature": "transactions", "status": "live",    "reason": "…" },
    { "feature": "wallets",      "status": "partial", "reason": "…", "requires": "…" },
    { "feature": "banking",      "status": "planned", "reason": "…", "requires": "…" }
  ]
}
```

## The `environment` field

| Value | What it means |
|---|---|
| `simulated` | The full state machine runs, but external providers are simulators. Nothing settles in the real world. Use it to build and test your integration. |
| `sandbox` | Real provider sandboxes are wired. Value moves in test networks, not production money. |
| `production` | Live money. |

Simulators are *honest fakes*: they implement the same interfaces as real
providers and refuse to run in production. They never invent a success that a
real provider would not have produced.

## The `features` array

Each feature carries a status, so your client can adapt instead of discovering
gaps by hitting them.

| Status | What it means for you |
|---|---|
| `live` | Real endpoint, works now. Build on it. |
| `partial` | The **reads** work; the money-moving half is not wired yet. Expect `501` on the write. |
| `simulated` | Works end to end, against a simulator. |
| `planned` | No endpoint yet. Calling it returns `501`. |

When a feature is not available, the endpoint says so explicitly:

```json
{
  "error": "not_implemented",
  "feature": "banking",
  "message": "wire transfers need a custody bank adapter",
  "requires": "a production banking partner"
}
```

!!! success

    **This is a promise, not an accident.** An unwired feature returns `501` with a
    reason: never an empty list, never a fabricated success. An empty list because
    nothing is configured and an empty list because you genuinely have nothing must
    not look the same, so they don't.

## Building against capabilities

The recommended pattern is to fetch `/capabilities` at startup, cache it, and
gate your UI and your retry logic on it:

```ts
const caps = await fetch(`${base}/capabilities`).then(r => r.json());

const canSendTo = (country: string) =>
  caps.corridors.some((c) => c.destinationCountry === country);
```

Do not hard-code the corridor list. It grows, and `/capabilities` is the only
place that is guaranteed to be current for *your* environment.

## Health

`GET /health` is the liveness check, separate from capabilities:

```json
{ "status": "ok", "version": "1.4.2", "uptimeSeconds": 41822, "checks": { "temporal": "ok" } }
```

It returns `503` with `"status": "degraded"` when the workflow engine is
unreachable. In that state the API cannot start new payments: every payment is
a durable workflow, so there is no degraded mode where sends quietly proceed
without one. Reads generally still work.

Point your load balancer and your uptime monitor at `/health`; point your
feature flags at `/capabilities`.

## Sandbox tools

A sandbox talks to the providers' own test environments, where nobody sends a
real bank transfer and some account numbers are magic. Three endpoints exist only
there, to drive a flow end to end. Each answers `501` on any other environment.
They are listed under [Sandbox tools](../api-reference/sandbox.md) in the API
reference.

| Endpoint | What it does |
|---|---|
| `GET /treasury/sandbox-prefill` | Per corridor, a beneficiary and an amount the sandbox providers are proven to settle, plus off-ramp beneficiaries and each provider's sandbox outcome rules. Use it to fill a form rather than guessing values |
| `POST /treasury/onramps/simulate-wire` | Makes a wire arrive at our account quoting **your** reference, for the amount you name. It is matched and converted exactly as a real wire is |
| `POST /treasury/onramp/:onRampId/simulate-payment` | For a deposit the payer pays on a provider's page: marks that payment as made at the provider |

Each one fakes only the outside party, never JET's side: what happens after the
simulated payment is the same code that handles real money.

!!! warning

    Sandbox values are not real. An account number that settles in a sandbox is
    somebody else's account, or nobody's, in production.
