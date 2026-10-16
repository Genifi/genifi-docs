---
description: "What this environment can do, whether it is up, and who your key belongs to."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Discovery

What this environment can do, whether it is up, and who your key belongs to.

| Endpoint | |
|---|---|
| [`GET /health`](#get-health) | Liveness check |
| [`GET /capabilities`](#get-capabilities) | What this environment can do |
| [`GET /me`](#get-me) | Who this API key belongs to |

## GET /health

Liveness check.

**Auth:** none

Open, no key needed. `503` with `status: "degraded"` when the workflow engine is unreachable: in that state no payment can start, so point your load balancer and uptime monitor here.

[Read the guide](../getting-started/environments.md#health)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `503` | Temporarily unavailable; retry with backoff |  |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `status` | `"ok"`, `"degraded"` | yes |  |
| `version` | string | yes |  |
| `uptimeSeconds` | number | yes |  |
| `checks` | object | yes |  |
| `checks.temporal` | `"ok"`, `"unreachable"` | yes |  |

## GET /capabilities

What this environment can do.

**Auth:** none

Open, no key needed. The live corridors and the status of every feature on this environment. Fetch it at startup and gate your integration on it rather than hard-coding a corridor list.

[Read the guide](../api/discovery.md#get-capabilities)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |

**`200` body** · [BackendCapabilities](types.md#backendcapabilities)

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `version` | string | yes |  |
| `environment` | `"simulated"`, `"sandbox"`, `"production"` | yes |  |
| `corridors` | [CorridorCapability](types.md#corridorcapability)[] | yes |  |
| `features` | [FeatureCapability](types.md#featurecapability)[] | yes |  |

## GET /me

Who this API key belongs to.

**Auth:** API key · **Rate limit:** `read`

Your organization, your role and, for a member key, which member you are. No side effects, so it is also the honest way to check that a key works.

[Read the guide](../api/discovery.md#get-me)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `actor` | object | no |  |
| `actor.name` | string or null | yes |  |
| `actor.email` | string | yes |  |
| `memberId` | string | no |  |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `role` | [IntegratorRole](types.md#integratorrole) | yes |  |
