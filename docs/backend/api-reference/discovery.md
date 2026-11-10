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
| [`GET /corridors`](#get-corridors) | The corridors your account may use |
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

Open, no key needed. The corridors this environment serves and the status of every feature on it. Which of those corridors YOUR account may use is `GET /corridors`.

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

## GET /corridors

The corridors your account may use.

**Auth:** API key · **Rate limit:** `read`

Corridors are enabled per account, each at the price agreed for it. A corridor not listed here is refused at quote and payment with `403 corridor_not_enabled`; ask your account manager to add one. Any role may read it.

[Read the guide](../api/discovery.md#get-corridors)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `corridors` | object[] | yes |  |
| `corridors[].key` | string | yes |  |
| `corridors[].direction` | `"payout"`, `"payin"` | yes |  |
| `corridors[].from` | `"USDC"`, `"EURC"`, `"USD"`, `"EUR"`, `"GBP"`, `"CZK"`, `"VND"`, `"NGN"`, `"PHP"`, `"INR"`, `"THB"`, `"MYR"`, `"GHS"`, `"NPR"`, `"IDR"`, `"MAD"`, `"BRL"`, `"SGD"`, `"HKD"`, `"KRW"`, `"AUD"`, `"MXN"`, `"TRY"`, `"PKR"`, `"PLN"`, `"CHF"`, `"DKK"`, `"NOK"`, `"SEK"`, `"AED"`, `"USDT"`, `"POL"`, `"ETH"`, `"AVAX"` | yes |  |
| `corridors[].to` | `"USDC"`, `"EURC"`, `"USD"`, `"EUR"`, `"GBP"`, `"CZK"`, `"VND"`, `"NGN"`, `"PHP"`, `"INR"`, `"THB"`, `"MYR"`, `"GHS"`, `"NPR"`, `"IDR"`, `"MAD"`, `"BRL"`, `"SGD"`, `"HKD"`, `"KRW"`, `"AUD"`, `"MXN"`, `"TRY"`, `"PKR"`, `"PLN"`, `"CHF"`, `"DKK"`, `"NOK"`, `"SEK"`, `"AED"`, `"USDT"`, `"POL"`, `"ETH"`, `"AVAX"` | yes |  |
| `corridors[].destinationCountry` | object | yes |  |
| `corridors[].destinationCountry.length` | number | yes |  |
| `corridors[].destinationCountry.__brand` | `"CountryCode"` | yes |  |
| `corridors[].enabledAt` | string | yes |  |

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
