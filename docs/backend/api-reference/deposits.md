---
description: "Getting money in: bank wires, PIX and USDC sent on chain."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Deposits

Getting money in: bank wires, PIX and USDC sent on chain.

| Endpoint | |
|---|---|
| [`GET /treasury/deposit-instructions`](#get-treasurydeposit-instructions) | Where to wire fiat, and the reference to quote |
| [`GET /treasury/onramp/quote`](#get-treasuryonrampquote) | Price a wire deposit without starting one |
| [`POST /treasury/onramp`](#post-treasuryonramp) | Open a wire deposit |
| [`GET /treasury/onramps`](#get-treasuryonramps) | List your deposits |
| [`GET /treasury/onramp/:onRampId`](#get-treasuryonramponrampid) | A deposit's status and timeline |
| [`GET /treasury/onramp/pix/quote`](#get-treasuryonramppixquote) | Price a PIX deposit |
| [`POST /treasury/onramp/pix`](#post-treasuryonramppix) | Open a PIX deposit |
| [`GET /treasury/crypto-deposit`](#get-treasurycrypto-deposit) | Where to send USDC on chain, and what each network costs |

## GET /treasury/deposit-instructions

Where to wire fiat, and the reference to quote.

**Auth:** API key · **KYB:** approved organizations only · **Rate limit:** `read`

Our bank details per currency and **your reference**. Every customer wires to the same account; the reference is the only thing that says the money is yours. Standing: read it before starting anything.

[Read the guide](../api/treasury.md#get-treasurydeposit-instructions)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `kyb_required` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body** · [DepositInstructions](types.md#depositinstructions)

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `reference` | string | yes |  |
| `accounts` | object[] | yes |  |
| `accounts[].currency` | string | yes |  |
| `accounts[].accountName` | string | yes |  |
| `accounts[].bankName` | string | no |  |
| `accounts[].iban` | string | no |  |
| `accounts[].accountNumber` | string | no |  |
| `accounts[].bic` | string | no |  |
| `accounts[].bankAddress` | string | no |  |

## GET /treasury/onramp/quote

Price a wire deposit without starting one.

**Auth:** API key · **Rate limit:** `read`

What a deposit of this size costs and delivers, and the `minimum`. Below the minimum is not an error here, so you can quote as the payer types. `gross` and `net` are exact only for USD (`atPar: true`); for EUR they are known once the conversion runs.

[Read the guide](../api/treasury.md#get-treasuryonrampquote)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `amount` | string | yes | matches `^\d+(\.\d+)?$` | Decimal string. |
| `currency` | string | yes | non-empty | The fiat the payer wires, e.g. `EUR` or `USD`. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `atPar` | boolean | yes |  |
| `sent` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `targetStablecoin` | string | yes |  |
| `feeBps` | number | yes |  |
| `deliveryFee` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `deliveryMethod` | `"mint+transfer"`, `"transfer"` | yes |  |
| `minimum` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `gross` | [MoneyJSON](types.md#moneyjson) | no |  |
| `fee` | [MoneyJSON](types.md#moneyjson) | no |  |
| `net` | [MoneyJSON](types.md#moneyjson) | no |  |

## POST /treasury/onramp

Open a wire deposit.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write`

Converts fiat you wire in into USDC in your home wallet. Answers `202` with bank instructions where the environment can produce them. Your `idempotencyKey` becomes the on-ramp ID.

[Read the guide](../api/treasury.md#post-treasuryonramp)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `idempotencyKey` | string | yes | matches `^[A-Za-z0-9_.:-]{8,200}$` | Your idempotency key. 8 to 200 characters of `A-Z a-z 0-9 _ . : -`. It becomes the on-ramp ID. |
| `amount` | object | yes |  |  |
| `amount.amount` | string | yes | matches `^\d+(\.\d+)?$` | Decimal string. At least the minimum `GET /treasury/onramp/quote` reports. |
| `amount.currency` | string | yes | non-empty | The fiat you will wire, e.g. `EUR`. |
| `targetStablecoin` | `"USDC"` | no | default `"USDC"` | The stablecoin delivered. `USDC` today. |
| `destinationAddress` | string | no | non-empty | Ignored. USDC is always delivered to your home wallet. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `409` | Conflict | `idempotency_key_conflict`, `idempotency_key_reused` |
| `422` | Understood, but cannot be done as asked | `ambiguous_home_wallet`, `no_home_wallet` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `instructions` | [OnRampInstructions](types.md#onrampinstructions) | no |  |
| `onRampId` | string | yes |  |
| `status` | string | yes |  |

## GET /treasury/onramps

List your deposits.

**Auth:** API key · **Rate limit:** `read`

Newest first, cursor-paginated. Includes deposits started by a wire you sent without an API call, whose IDs you never chose (`dep-…`). Filter with `origin`.

[Read the guide](../api/treasury.md#get-treasuryonramps)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `limit` | integer | no | > 0; ≤ 500 |  |
| `cursor` | string | no | 1–512 chars |  |
| `origin` | `"api"`, `"deposit"`, `"pix"` | no |  |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `invalid_cursor`, `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `onramps` | [OnRampSummary](types.md#onrampsummary)[] | yes |  |
| `count` | number | yes |  |
| `nextCursor` | string or null | yes |  |
| `hasMore` | boolean | yes |  |

## GET /treasury/onramp/:onRampId

A deposit's status and timeline.

**Auth:** API key · **Rate limit:** `read`

Wire and PIX deposits alike. A deposit that is not yours is `404`.

[Read the guide](../api/treasury.md#get-treasuryonramponrampid)

**Path parameters**

| Name | Type |
|---|---|
| `onRampId` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `onRampId` | string | yes |  |
| `running` | boolean | yes |  |
| `failure` | string | no |  |
| `state` | `"FAILED"`, `"IN_DOUBT"`, `"REFUNDED"`, `"DELIVERED"`, `"EXPIRED"` | no |  |
| `net` | object | no |  |
| `net.amount` | string | yes |  |
| `net.currency` | string | yes |  |
| `fee` | object | no |  |
| `fee.amount` | string | yes |  |
| `fee.currency` | string | yes |  |
| `stage` | `"FAILED"`, `"IN_DOUBT"`, `"REFUNDED"`, `"DELIVERED"`, `"EXPIRED"`, `"OPENING_COLLECTION"`, `"AWAITING_PAYMENT"`, `"PAYMENT_RECEIVED"`, `"FORWARDING"`, `"CONVERTING"`, `"DELIVERING"` | no |  |
| `timeline` | [OnRampEvent](types.md#onrampevent)[] | no |  |
| `gross` | object | no |  |
| `gross.amount` | string | yes |  |
| `gross.currency` | string | yes |  |
| `feeBps` | number | no |  |
| `fxProvider` | string | no |  |
| `deliveryMethod` | `"provider"`, `"mint+transfer"`, `"transfer"`, `"mint"` | no |  |
| `txHash` | string | no |  |
| `instructions` | [OnRampInstructions](types.md#onrampinstructions) | no |  |

## GET /treasury/onramp/pix/quote

Price a PIX deposit.

**Auth:** API key · **Rate limit:** `read`

A live, indicative price for paying BRL by PIX: what lands in your wallet, the fee, and this environment's minimum and maximum. `available: false` with a `reason` when it cannot be priced right now.

[Read the guide](../api/treasury.md#pix-deposits)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `amount` | string | yes | matches `^\d+(\.\d{1,2})?$` | BRL, at most 2 decimals. |
| `currency` | `"BRL"` | no | default `"BRL"` | `BRL`, the only currency. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `maximum` | [MoneyJSON](types.md#moneyjson) | no |  |
| `minimum` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `autoPaidBelow` | [MoneyJSON](types.md#moneyjson) | no |  |
| `sent` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `targetStablecoin` | string | yes |  |
| `feeBps` | number | yes |  |
| `chain` | string | yes |  |
| `available` | boolean | yes |  |
| `reason` | string | no |  |
| `basePrice` | string | no |  |
| `indicative` | boolean | no |  |
| `net` | [MoneyJSON](types.md#moneyjson) | no |  |
| `fee` | [MoneyJSON](types.md#moneyjson) | no |  |
| `providerFees` | [MoneyJSON](types.md#moneyjson) | no |  |
| `provider` | string | no |  |

## POST /treasury/onramp/pix

Open a PIX deposit.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write`

Answers `202` with a PIX code to show as a QR. When it is paid, USDC is sent to your home wallet; delivery is confirmed on chain before anything is credited. Retrying with the same key returns the same code.

[Read the guide](../api/treasury.md#pix-deposits)

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `idempotencyKey` | string | yes | matches `^[A-Za-z0-9_.:-]{8,200}$` | Your idempotency key. 8 to 200 characters of `A-Z a-z 0-9 _ . : -`. It becomes the on-ramp ID. |
| `amount` | object | yes |  |  |
| `amount.amount` | string | yes | matches `^\d+(\.\d{1,2})?$` | BRL, at most 2 decimals, within the limits the quote reports. |
| `amount.currency` | `"BRL"` | no | default `"BRL"` | `BRL`, the only currency. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `409` | Conflict | `idempotency_key_conflict`, `idempotency_key_reused` |
| `422` | Understood, but cannot be done as asked | `pix_onramp_unavailable` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `instructions` | [OnRampInstructions](types.md#onrampinstructions) | no |  |
| `onRampId` | string | yes |  |
| `status` | string | yes |  |

## GET /treasury/crypto-deposit

Where to send USDC on chain, and what each network costs.

**Auth:** API key · **Rate limit:** `read`

Your deposit address on every network we credit. USDC sent on your home network is credited as it is; USDC sent on any other network is moved to your home network and the `fee` for that is deducted. Each non-home network carries a live fee estimate.

[Read the guide](../api/treasury.md#usdc-deposits)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `422` | Understood, but cannot be done as asked | `no_home_wallet` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `networks` | object[] | yes |  |
| `networks[].feeUnavailable` | string | no |  |
| `networks[].fee` | [MoneyJSON](types.md#moneyjson) | no |  |
| `networks[].network` | string | yes |  |
| `networks[].address` | string | yes |  |
| `networks[].home` | boolean | yes |  |
| `quotedAt` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `address` | string | no |  |
| `currency` | string | yes |  |
