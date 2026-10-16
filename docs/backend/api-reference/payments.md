---
description: "Price a payment, send it, and follow it to the end."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Quotes and payments

Price a payment, send it, and follow it to the end.

| Endpoint | |
|---|---|
| [`POST /quotes`](#post-quotes) | Get a firm, rate-locked quote |
| [`POST /transactions`](#post-transactions) | Send a payment |
| [`GET /transactions`](#get-transactions) | List your payments |
| [`GET /transactions/:id`](#get-transactionsid) | A payment's status and timeline |

## POST /quotes

Get a firm, rate-locked quote.

**Auth:** API key · **Permission:** `payment.initiate` (owner, admin, member) · **Rate limit:** `write`

Prices a payment without starting one. Returns a single-use `quoteId` that locks the rate for about a minute; pass it to `POST /transactions` to pay exactly this price. Supply `idempotencyKey` (the future payment ID) to bind the quote to that payment.

[Read the guide](../api/quotes.md)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `corridor` | object | yes |  |  |
| `corridor.to` | string | yes | non-empty | Destination currency, e.g. `VND`. |
| `corridor.destinationCountry` | string | yes | exactly 2 chars | ISO 3166-1 alpha-2 country code, e.g. `VN`. |
| `sendAmount` | object | yes |  |  |
| `sendAmount.amount` | string | yes | matches `^\d+(\.\d+)?$` | Decimal string. More decimals than the currency allows is refused, never rounded. |
| `sendAmount.currency` | string | yes | non-empty | The stablecoin you send, e.g. `USDC`. |
| `integratorWalletId` | string | no | non-empty | Optional. Must be your home wallet; omit it to use your home wallet. |
| `idempotencyKey` | string | no | matches `^[A-Za-z0-9_.:-]{8,200}$` | Optional. The ID the payment will carry; accepting the quote then requires the same key. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `422` | Understood, but cannot be done as asked | `no_route_for_chain`, `not_home_wallet`, `quote_unavailable` |
| `429` | Rate limited | `rate_limited` |

**`200` body** · [QuotePreviewResult](types.md#quotepreviewresult)

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `quoteId` | string | yes |  |
| `sendAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `receiveAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `fee` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `rate` | [RateJSON](types.md#ratejson) | yes |  |
| `provider` | string | yes |  |
| `routeKind` | `"orchestrator"`, `"owned"` | yes |  |
| `estimatedSeconds` | number | yes |  |
| `expiresAt` | string | yes |  |
| `margin` | [OwnedMarginJSON](types.md#ownedmarginjson) | no |  |
| `networkFee` | [MoneyJSON](types.md#moneyjson) | no |  |
| `routed` | boolean | no |  |

## POST /transactions

Send a payment.

**Auth:** API key · **Permission:** `payment.initiate` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `initiate`

Starts a durable payment and answers `202` straight away. Your `idempotencyKey` becomes the transaction ID: retrying the same request never pays twice. Funded from your home wallet. Follow it with `GET /transactions/:id` or webhooks.

[Read the guide](../api/transactions.md)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `idempotencyKey` | string | yes | matches `^[A-Za-z0-9_.:-]{8,200}$` | Your idempotency key. 8 to 200 characters of `A-Z a-z 0-9 _ . : -`. It becomes the transaction ID. |
| `corridor` | object | yes |  |  |
| `corridor.to` | string | yes | non-empty | Destination currency, e.g. `VND`. |
| `corridor.destinationCountry` | string | yes | exactly 2 chars | ISO 3166-1 alpha-2 country code, e.g. `VN`. |
| `sendAmount` | object | yes |  |  |
| `sendAmount.amount` | string | yes | matches `^\d+(\.\d+)?$` | Decimal string. More decimals than the currency allows is refused, never rounded. |
| `sendAmount.currency` | string | yes | non-empty | The stablecoin you send, e.g. `USDC`. |
| `integratorWalletId` | string | no | non-empty | Optional. Must be your home wallet; omit it to use your home wallet. |
| `recipientReference` | string | yes | non-empty | Your own reference for the recipient. Echoed back; never shown to them. |
| `originator` | object | no |  | The sender, for the Travel Rule. Forward as much of your own KYC as you hold. |
| `originator.name` | string | no | 1–200 chars |  |
| `originator.account` | string | no | 1–140 chars |  |
| `originator.address` | string | no | 1–280 chars |  |
| `originator.country` | string | no | exactly 2 chars |  |
| `originator.bic` | string | no | 8–11 chars |  |
| `originator.institution` | string | no | 1–200 chars |  |
| `originator.idNumber` | string | no | 1–80 chars |  |
| `originator.dateOfBirth` | string | no | matches `^\d{4}-\d{2}-\d{2}$` |  |
| `beneficiary` | object | no |  | Who is paid, and where. Required fields vary by corridor; a missing one is a `400 beneficiary_incomplete` naming it. |
| `beneficiary.name` | string | yes | 1–200 chars |  |
| `beneficiary.account` | string | yes | 1–140 chars |  |
| `beneficiary.bankCode` | string | no | 1–35 chars |  |
| `beneficiary.bic` | string | no | 8–11 chars |  |
| `beneficiary.bankName` | string | no | 1–200 chars |  |
| `beneficiary.country` | string | no | exactly 2 chars |  |
| `beneficiary.city` | string | no | 1–100 chars |  |
| `beneficiary.addressLine` | string | no | 1–280 chars |  |
| `beneficiary.state` | string | no | 1–100 chars |  |
| `beneficiary.postalCode` | string | no | 1–20 chars |  |
| `beneficiary.phone` | object | no |  |  |
| `beneficiary.phone.callingCode` | string | yes | matches `^\+?\d{1,4}$` |  |
| `beneficiary.phone.number` | string | yes | matches `^\d{4,15}$` |  |
| `payoutMode` | `"regular"`, `"instant"` | no |  | Accepted for compatibility. Every payment runs as `regular`. |
| `quoteId` | string | no | 1–200 chars | A quote from `POST /quotes`. With it the payment executes at that locked price. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `beneficiary_incomplete`, `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `compliance_hold`, `forbidden`, `kyb_required` |
| `422` | Understood, but cannot be done as asked | `ambiguous_home_wallet`, `insufficient_funds`, `no_home_wallet`, `not_home_wallet`, `quote_expired`, `quote_mismatch`, `quote_not_found`, `wallet_chain_unknown` |
| `429` | Rate limited | `rate_limited` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactionId` | string | yes |  |
| `status` | string | yes |  |

## GET /transactions

List your payments.

**Auth:** API key · **Rate limit:** `read`

Newest first, cursor-paginated. Rows are what you submitted; read each row's live status from `GET /transactions/:id`.

[Read the guide](../api/transactions.md#get-transactions)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `limit` | integer | no | > 0; ≤ 500 |  |
| `cursor` | string | no | 1–512 chars |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `invalid_cursor`, `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactions` | [TransactionListItem](types.md#transactionlistitem)[] | yes |  |
| `count` | number | yes |  |
| `nextCursor` | string or null | yes |  |
| `hasMore` | boolean | yes |  |

## GET /transactions/:id

A payment's status and timeline.

**Auth:** API key · **Rate limit:** `read`

A payment that is not yours is `404`, the same as one that does not exist.

[Read the guide](../api/transactions.md#get-transactionsid)

**Path parameters**

| Name | Type |
|---|---|
| `id` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |

**`200` body** · [TransactionStatus](types.md#transactionstatus)

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactionId` | string | yes |  |
| `running` | boolean | yes |  |
| `failure` | string | no |  |
| `state` | `"FAILED"`, `"IN_DOUBT"`, `"REFUNDED"`, `"INITIATED"`, `"QUOTED"`, `"FUNDED"`, `"ROUTING"`, `"SETTLING"`, `"PAID_OUT"`, `"RECONCILED"`, `"REFUNDING"`, `"PENDING_RETRY"`, `"NEEDS_ATTENTION"` | no |  |
| `provider` | string | no |  |
| `payoutMode` | `"regular"`, `"instant"` | no |  |
| `receiveAmount` | object | no |  |
| `receiveAmount.amount` | string | yes |  |
| `receiveAmount.currency` | string | yes |  |
| `receiveAmount.scale` | number | yes |  |
| `payoutDelta` | [PayoutDelta](types.md#payoutdelta) | no |  |
| `margin` | [OwnedMarginJSON](types.md#ownedmarginjson) | no |  |
| `timeline` | object[] | no |  |
| `timeline[].state` | string | yes |  |
| `timeline[].detail` | string | no |  |
