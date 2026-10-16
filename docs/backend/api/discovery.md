# Discovery

Three unauthenticated endpoints that describe the service itself.

## GET /health

Liveness. Point your load balancer and uptime monitor here.

=== "Request"

    ```bash
    curl "$JET_API/health"
    ```

=== "200 OK"

    ```json
    {
      "status": "ok",
      "version": "1.4.2",
      "uptimeSeconds": 41822,
      "checks": { "temporal": "ok" }
    }
    ```

=== "503 Degraded"

    ```json
    {
      "status": "degraded",
      "version": "1.4.2",
      "uptimeSeconds": 41822,
      "checks": { "temporal": "unreachable" }
    }
    ```

| Field | Type | Notes |
|---|---|---|
| `status` | `"ok"` \| `"degraded"` | `degraded` is returned with HTTP `503` |
| `version` | string | The deployed API version |
| `uptimeSeconds` | number | Seconds since this instance started |
| `checks` | object | Per-dependency reachability |

`degraded` means the workflow engine is unreachable, so **no new payment can be
started**. Every payment is a durable workflow; there is no fallback mode where
sends quietly proceed without one. Reads generally still work.

## GET /capabilities

What this environment can actually do. See
[Environments](../getting-started/environments.md) for how to build against it.

=== "Request"

    ```bash
    curl "$JET_API/capabilities"
    ```

=== "200 OK"

    ```json
    {
      "version": "1.4.2",
      "environment": "sandbox",
      "corridors": [
        { "from": "USDC", "to": "VND", "destinationCountry": "VN", "ownedRail": false, "instant": false }
      ],
      "features": [
        { "feature": "transactions", "status": "live", "reason": "…" },
        { "feature": "banking", "status": "planned", "reason": "no custody bank adapter", "requires": "a production banking partner" }
      ]
    }
    ```

| Field | Type | Notes |
|---|---|---|
| `environment` | `"simulated"` \| `"sandbox"` \| `"production"` | How real the providers behind this environment are |
| `corridors[].ownedRail` | boolean | JET operates this corridor end to end |
| `corridors[].instant` | boolean | Always `false` today; see [Corridors](../concepts/corridors.md) |
| `features[].status` | `"live"` \| `"partial"` \| `"simulated"` \| `"planned"` | Per-feature availability |
| `features[].requires` | string \| absent | For `partial`/`planned`: what it would take to finish it |

## GET /me

Not unauthenticated, but it belongs here: it answers *which organization is this
key?* with no side effects.

=== "Request"

    ```bash
    curl "$JET_API/me" -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "id": "int_4f21…",
      "name": "Northwind Payments",
      "role": "owner",
      "memberId": "mem_7c93…"
    }
    ```

| Field | Type | Notes |
|---|---|---|
| `id` | string | Your organization's ID |
| `name` | string | Your organization's name |
| `role` | `"owner"` \| `"admin"` \| `"member"` \| `"viewer"` | What this credential may do: see [Authentication](../getting-started/authentication.md#roles) |
| `memberId` | string \| absent | Present when the caller is a named person rather than the organization key |

Identity only, deliberately: no counts, no balances, no key material. Use it to
validate a key at startup, and to tell environments apart when you hold several.
