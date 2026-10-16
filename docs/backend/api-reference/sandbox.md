---
description: "Helpers that exist only on a sandbox: make a deposit arrive, and fill forms with values the sandbox providers are proven to settle."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Sandbox tools

Helpers that exist only on a sandbox: make a deposit arrive, and fill forms with values the sandbox providers are proven to settle.

| Endpoint | |
|---|---|
| [`POST /treasury/onramp/:onRampId/simulate-payment`](#post-treasuryonramponrampidsimulate-payment) | Make a provider-collected deposit's payment arrive |
| [`POST /treasury/onramps/simulate-wire`](#post-treasuryonrampssimulate-wire) | Make a wire arrive at our account, quoting your reference |
| [`GET /treasury/sandbox-prefill`](#get-treasurysandbox-prefill) | Inputs the sandbox providers are proven to settle |
| [`POST /wallets/:walletId/topup-eurc`](#post-walletswalletidtopup-eurc) | Top up a simulated EURC balance |

## POST /treasury/onramp/:onRampId/simulate-payment

Make a provider-collected deposit's payment arrive.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **Rate limit:** `write` · **Sandbox only**

**Sandbox only.** Answers `501` on other environments.

For a deposit whose payer pays on a provider's page. Fakes only the payer; the deposit then runs exactly as it would for real money.

[Read the guide](../getting-started/environments.md#sandbox-tools)

**Path parameters**

| Name | Type |
|---|---|
| `onRampId` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `409` | Conflict | `not_simulatable` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `onRampId` | string | yes |  |
| `simulated` | boolean | yes |  |
| `detail` | string | yes |  |

## POST /treasury/onramps/simulate-wire

Make a wire arrive at our account, quoting your reference.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write` · **Sandbox only**

**Sandbox only.** Answers `501` on other environments.

Posts one incoming wire for this amount, quoting your reference. It is matched and converted exactly as a real wire is; the response says whether it was attributed.

[Read the guide](../getting-started/environments.md#sandbox-tools)

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `amount` | object | yes |  |  |
| `amount.amount` | string | yes | matches `^\d+(\.\d+)?$` |  |
| `amount.currency` | string | yes | non-empty |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `unknown_currency`, `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `simulated` | boolean | yes |  |
| `reference` | string | yes |  |
| `amount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `statementLineId` | string | yes |  |
| `onRampId` | string | yes |  |
| `disposition` | `"pending"`, `"attributed"`, `"parked"` | yes |  |
| `detail` | string | yes |  |

## GET /treasury/sandbox-prefill

Inputs the sandbox providers are proven to settle.

**Auth:** API key · **Rate limit:** `read` · **Sandbox only**

**Sandbox only.** Answers `501` on other environments.

Per corridor: a beneficiary and an amount that settle end to end, plus off-ramp beneficiaries and the providers' sandbox outcome bands. Never served in production.

[Read the guide](../getting-started/environments.md#sandbox-tools)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `payouts` | object[] | yes |  |
| `payouts[].corridor` | object | yes |  |
| `payouts[].corridor.to` | string | yes |  |
| `payouts[].corridor.destinationCountry` | string | yes |  |
| `payouts[].sendAmount` | object | yes |  |
| `payouts[].sendAmount.amount` | string | yes |  |
| `payouts[].sendAmount.currency` | string | yes |  |
| `payouts[].beneficiary` | [SandboxBeneficiary](types.md#sandboxbeneficiary) | yes |  |
| `payouts[].recipientReference` | string | yes |  |
| `offrampBeneficiaries` | [SandboxOffRampBeneficiary](types.md#sandboxofframpbeneficiary)[] | yes |  |
| `bands` | [SandboxOutcomeBand](types.md#sandboxoutcomeband)[] | yes |  |
| `notes` | string[] | yes |  |

## POST /wallets/:walletId/topup-eurc

Top up a simulated EURC balance.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write` · **Sandbox only**

**Sandbox only.** Answers `501` on other environments.

Demo only: credits a simulated EURC balance one-for-one. No real money moves.

**Path parameters**

| Name | Type |
|---|---|
| `walletId` | string |

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `amount` | string | yes | matches `^\d+(\.\d{1,2})?$` |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `404` | Not found, or not yours | `not_found` |
| `409` | Conflict | `conflict` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `walletId` | string | yes |  |
| `balance` | object | yes |  |
| `balance.asset` | string | yes |  |
| `balance.amount` | string | yes |  |
| `balance.scale` | `6` | yes |  |
