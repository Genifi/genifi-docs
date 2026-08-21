---
description: How to tell what your environment can actually do, before you build on it.
---

# Environments

Genifi environments differ in one important way: how much of the rail is backed
by a real provider. Rather than making you guess, the API publishes it.

```bash
curl "$GENIFI_API/capabilities"
```

```json
{
  "version": "1.4.2",
  "environment": "sandbox",
  "corridors": [
    { "from": "USDC", "to": "VND", "destinationCountry": "VN", "ownedRail": true,  "instant": true  },
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

{% hint style="success" %}
**This is a promise, not an accident.** An unwired feature returns `501` with a
reason — never an empty list, never a fabricated success. An empty list because
nothing is configured and an empty list because you genuinely have nothing must
not look the same, so they don't.
{% endhint %}

## Building against capabilities

The recommended pattern is to fetch `/capabilities` at startup, cache it, and
gate your UI and your retry logic on it:

```ts
const caps = await fetch(`${base}/capabilities`).then(r => r.json());

const canSendTo = (country: string) =>
  caps.corridors.some((c) => c.destinationCountry === country);

const instantAvailable = (country: string) =>
  caps.corridors.some((c) => c.destinationCountry === country && c.instant);
```

Do not hard-code the corridor list. It grows, and `/capabilities` is the only
place that is guaranteed to be current for *your* environment.

## Health

`GET /health` is the liveness check, separate from capabilities:

```json
{ "status": "ok", "version": "1.4.2", "uptimeSeconds": 41822, "checks": { "temporal": "ok" } }
```

It returns `503` with `"status": "degraded"` when the workflow engine is
unreachable. In that state the API cannot start new payments — every payment is
a durable workflow, so there is no degraded mode where sends quietly proceed
without one. Reads generally still work.

Point your load balancer and your uptime monitor at `/health`; point your
feature flags at `/capabilities`.
