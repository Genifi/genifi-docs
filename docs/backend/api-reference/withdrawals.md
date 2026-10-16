---
description: "Getting money out: USDC to your own wallet, or fiat to a bank account."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Withdrawals and off-ramps

Getting money out: USDC to your own wallet, or fiat to a bank account.

| Endpoint | |
|---|---|
| [`GET /treasury/withdrawals/quote`](#get-treasurywithdrawalsquote) | Price a USDC withdrawal to your own wallet |
| [`POST /treasury/withdrawals`](#post-treasurywithdrawals) | Withdraw USDC to your own wallet |
| [`GET /treasury/withdrawals`](#get-treasurywithdrawals) | List your withdrawals |
| [`GET /treasury/withdrawals/:withdrawalId`](#get-treasurywithdrawalswithdrawalid) | A withdrawal's status and timeline |
| [`POST /treasury/beneficiaries`](#post-treasurybeneficiaries) | Register a bank account an off-ramp can pay |
| [`GET /treasury/beneficiaries`](#get-treasurybeneficiaries) | Your registered beneficiaries |
| [`GET /treasury/offramp/quote`](#get-treasuryofframpquote) | Price an off-ramp to a beneficiary |
| [`POST /treasury/offramp`](#post-treasuryofframp) | Pay USDC out to a beneficiary's bank account |
| [`GET /treasury/offramps`](#get-treasuryofframps) | List your off-ramps |
| [`GET /treasury/offramp/:offRampId`](#get-treasuryofframpofframpid) | An off-ramp's status and timeline |

## GET /treasury/withdrawals/quote

Price a USDC withdrawal to your own wallet.

**Auth:** API key · **Rate limit:** `read`

What sending this much USDC to this network costs, and what arrives.

[Read the guide](../api/withdrawals.md#get-treasurywithdrawalsquote)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `amount` | string | yes | matches `^\d+(\.\d+)?$` | USDC, as a decimal string. |
| `chain` | string | yes | 1–40 chars | The network your destination address is on. See `GET /treasury/crypto-deposit` for the networks this environment supports. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `422` | Understood, but cannot be done as asked | `quote_unavailable`, `unsupported_network` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `amount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `network` | string | yes |  |
| `fee` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `net` | [MoneyJSON](types.md#moneyjson) | yes |  |

## POST /treasury/withdrawals

Withdraw USDC to your own wallet.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write`

Re-priced when you send it; the response carries the fee that applied. **An on-chain send cannot be reversed**: check the address and network. Your `idempotencyKey` becomes the withdrawal ID, and reusing it with a different amount, network or address is a `409`.

[Read the guide](../api/withdrawals.md#post-treasurywithdrawals)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `idempotencyKey` | string | yes | matches `^[A-Za-z0-9_.:-]{8,200}$` | Your idempotency key. 8 to 200 characters of `A-Z a-z 0-9 _ . : -`. It becomes the withdrawal ID. |
| `amount` | object | yes |  |  |
| `amount.amount` | string | yes | matches `^\d+(\.\d+)?$` | Decimal string. |
| `amount.currency` | string | yes | 3–5 chars | `USDC`. |
| `chain` | string | yes | 1–40 chars | The network to send on. Must be the network your address is on. |
| `toAddress` | string | yes | matches `^0x[0-9a-fA-F]{40}$` | A `0x`-prefixed EVM address. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `409` | Conflict | `idempotency_key_conflict`, `idempotency_key_reused` |
| `422` | Understood, but cannot be done as asked | `ambiguous_home_wallet`, `no_home_wallet`, `quote_unavailable`, `unsupported_network` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `withdrawalId` | string | yes |  |
| `amount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `network` | string | yes |  |
| `fee` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `net` | [MoneyJSON](types.md#moneyjson) | yes |  |

## GET /treasury/withdrawals

List your withdrawals.

**Auth:** API key · **Rate limit:** `read`

Newest first, cursor-paginated. Read each one's status from its own endpoint.

[Read the guide](../api/withdrawals.md#withdrawals)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `limit` | integer | no | ≥ 1; ≤ 200 |  |
| `cursor` | string | no | non-empty |  |

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
| `withdrawals` | [WithdrawalSummary](types.md#withdrawalsummary)[] | yes |  |
| `count` | number | yes |  |
| `nextCursor` | string or null | yes |  |
| `hasMore` | boolean | yes |  |

## GET /treasury/withdrawals/:withdrawalId

A withdrawal's status and timeline.

**Auth:** API key · **Rate limit:** `read`

[Read the guide](../api/withdrawals.md#get-treasurywithdrawalswithdrawalid)

**Path parameters**

| Name | Type |
|---|---|
| `withdrawalId` | string |

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
| `withdrawalId` | string | yes |  |
| `state` | string | yes |  |
| `running` | boolean | yes |  |
| `timeline` | object[] | yes |  |
| `timeline[].state` | string | yes |  |
| `timeline[].detail` | string | no |  |
| `net` | [MoneyJSON](types.md#moneyjson) | no |  |
| `txHash` | string | no |  |

## POST /treasury/beneficiaries

Register a bank account an off-ramp can pay.

**Auth:** API key · **Permission:** `beneficiary.manage` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write`

An off-ramp pays a registered beneficiary by ID, never bank details sent inline. Every field is required. Registering is your assertion; nothing here verifies the account.

[Read the guide](../api/withdrawals.md#post-treasurybeneficiaries)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `name` | string | yes | 1–140 chars | The account holder. |
| `accountId` | string | yes | 1–140 chars | IBAN or account number. |
| `bic` | string | yes | 8–11 chars | BIC/SWIFT, 8 or 11 characters. |
| `bankName` | string | yes | 1–140 chars |  |
| `city` | string | yes | 1–140 chars |  |
| `addressLine` | string | yes | 1–140 chars |  |
| `country` | string | yes | matches `^[A-Za-z]{2}$` | ISO 3166-1 alpha-2. |
| `currency` | string | yes | exactly 3 chars | The account's currency. The beneficiary is paid in it. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `201` | Created |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`201` body** · [BeneficiaryView](types.md#beneficiaryview)

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `name` | string | yes |  |
| `accountId` | string | yes |  |
| `bic` | string | yes |  |
| `bankName` | string | yes |  |
| `city` | string | yes |  |
| `addressLine` | string | yes |  |
| `country` | string | yes |  |
| `currency` | string | yes |  |
| `createdAt` | string | yes |  |

## GET /treasury/beneficiaries

Your registered beneficiaries.

**Auth:** API key · **Rate limit:** `read`

[Read the guide](../api/withdrawals.md#post-treasurybeneficiaries)

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
| `beneficiaries` | [BeneficiaryView](types.md#beneficiaryview)[] | yes |  |
| `count` | number | yes |  |

## GET /treasury/offramp/quote

Price an off-ramp to a beneficiary.

**Auth:** API key · **Rate limit:** `read`

What the beneficiary receives, in their currency, for this much USDC. Where the figure is firm the response carries a `quoteId`; pass it to `POST /treasury/offramp` and the beneficiary is paid exactly the quoted `net`.

[Read the guide](../api/withdrawals.md#get-treasuryofframpquote)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `amount` | string | yes | matches `^\d+(\.\d+)?$` | USDC, as a decimal string. |
| `beneficiaryId` | string (uuid) | yes |  | From `POST /treasury/beneficiaries`. |

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
| `pulled` | object | yes |  |
| `pulled.amount` | string | yes |  |
| `pulled.currency` | string | yes |  |
| `pulled.scale` | number | yes |  |
| `settlementCurrency` | string | yes |  |
| `atPar` | boolean | yes |  |
| `feeBps` | number | yes |  |
| `fee` | object | no |  |
| `fee.amount` | string | yes |  |
| `fee.currency` | string | yes |  |
| `fee.scale` | number | yes |  |
| `deliveryFee` | object | yes |  |
| `deliveryFee.amount` | string | yes |  |
| `deliveryFee.currency` | string | yes |  |
| `deliveryFee.scale` | number | yes |  |
| `net` | object | no |  |
| `net.amount` | string | yes |  |
| `net.currency` | string | yes |  |
| `net.scale` | number | yes |  |
| `fxBufferBps` | number | no |  |
| `minimum` | object | yes |  |
| `minimum.amount` | string | yes |  |
| `minimum.currency` | string | yes |  |
| `minimum.scale` | number | yes |  |
| `quoteId` | string | no |  |
| `expiresAt` | string | no |  |

## POST /treasury/offramp

Pay USDC out to a beneficiary's bank account.

**Auth:** API key · **Permission:** `treasury.move` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `write`

Takes USDC from your home wallet and pays a beneficiary you registered. Answers `202`; your `idempotencyKey` becomes the off-ramp ID.

[Read the guide](../api/withdrawals.md#post-treasuryofframp)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `idempotencyKey` | string | yes | matches `^[A-Za-z0-9_.:-]{8,200}$` | Your idempotency key. 8 to 200 characters of `A-Z a-z 0-9 _ . : -`. It becomes the off-ramp ID. |
| `amount` | object | yes |  |  |
| `amount.amount` | string | yes | matches `^\d+(\.\d+)?$` | USDC, as a decimal string. |
| `amount.currency` | string | yes | non-empty | `USDC`. |
| `beneficiaryId` | string (uuid) | yes |  | From `POST /treasury/beneficiaries`. |
| `chain` | string | no | matches `^[a-z0-9-]{1,40}$` | Optional, and only your home network is accepted. Omit it. |
| `quoteId` | string | no | matches `^oq_[A-Za-z0-9-]{8,80}$` | From `GET /treasury/offramp/quote`. Binds the payment to the quoted `net`. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `409` | Conflict | `idempotency_key_conflict` |
| `422` | Understood, but cannot be done as asked | `not_home_chain`, `quote_already_used`, `quote_mismatch`, `quote_not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `offRampId` | string | yes |  |
| `status` | string | yes |  |

## GET /treasury/offramps

List your off-ramps.

**Auth:** API key · **Rate limit:** `read`

Newest first, cursor-paginated.

[Read the guide](../api/withdrawals.md#off-ramps)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `limit` | integer | no | ≥ 1; ≤ 200 |  |
| `cursor` | string | no | non-empty |  |

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
| `offramps` | [OffRampSummary](types.md#offrampsummary)[] | yes |  |
| `count` | number | yes |  |
| `nextCursor` | string or null | yes |  |
| `hasMore` | boolean | yes |  |

## GET /treasury/offramp/:offRampId

An off-ramp's status and timeline.

**Auth:** API key · **Rate limit:** `read`

[Read the guide](../api/withdrawals.md#get-treasuryofframpofframpid)

**Path parameters**

| Name | Type |
|---|---|
| `offRampId` | string |

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
| `offRampId` | string | yes |  |
| `stage` | string | yes |  |
| `state` | string | no |  |
| `closedAs` | string | no |  |
| `timeline` | object[] | yes |  |
| `timeline[].stage` | string | yes |  |
| `timeline[].at` | string | yes |  |
| `timeline[].detail` | string | no |  |
| `credited` | object | no |  |
| `credited.amount` | string | yes |  |
| `credited.currency` | string | yes |  |
| `credited.scale` | number | yes |  |
| `net` | object | no |  |
| `net.amount` | string | yes |  |
| `net.currency` | string | yes |  |
| `net.scale` | number | yes |  |
| `fee` | object | no |  |
| `fee.amount` | string | yes |  |
| `fee.currency` | string | yes |  |
| `fee.scale` | number | yes |  |
| `deliveryFee` | object | no |  |
| `deliveryFee.amount` | string | yes |  |
| `deliveryFee.currency` | string | yes |  |
| `deliveryFee.scale` | number | yes |  |
| `slippage` | object | no |  |
| `slippage.amount` | string | yes |  |
| `slippage.currency` | string | yes |  |
| `slippage.scale` | number | yes |  |
| `fxBufferBps` | number | no |  |
