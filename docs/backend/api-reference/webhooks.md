---
description: "Where signed event notifications are delivered, and which events exist."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Webhooks

Where signed event notifications are delivered, and which events exist.

| Endpoint | |
|---|---|
| [`GET /webhooks/endpoint`](#get-webhooksendpoint) | The registered webhook URL |
| [`PUT /webhooks/endpoint`](#put-webhooksendpoint) | Register or replace the webhook URL |
| [`DELETE /webhooks/endpoint`](#delete-webhooksendpoint) | Stop webhook delivery |

## GET /webhooks/endpoint

The registered webhook URL.

**Auth:** API key · **Rate limit:** `read`

The URL only. The signing secret is shown once, when you register.

[Read the guide](../webhooks/README.md)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `url` | string or null | yes |  |

## PUT /webhooks/endpoint

Register or replace the webhook URL.

**Auth:** API key · **Permission:** `webhook.manage` (owner, admin) · **Rate limit:** `write`

Returns a new signing `secret`, **once**. `https` only; private and loopback addresses are refused.

[Read the guide](../webhooks/README.md)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `url` | string (uri) | yes |  |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `url` | string | yes |  |
| `secret` | string | yes |  |

## DELETE /webhooks/endpoint

Stop webhook delivery.

**Auth:** API key · **Permission:** `webhook.manage` (owner, admin) · **Rate limit:** `write`

Removes the URL and destroys the secret. Safe to repeat: deleting when nothing is registered is a `200` with `deleted: false`.

[Read the guide](../webhooks/README.md)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `url` | null | yes |  |
| `deleted` | boolean | yes |  |

## Event types

Every event a delivery can carry. The delivery's `transactionId` is the ID of the subject.

| Event | Subject | When |
|---|---|---|
| `transaction.quoted` | payment | The payment was priced and the rate locked. |
| `transaction.funded` | payment | The payment's funds have left your wallet and are held by JET. |
| `transaction.paid_out` | payment | The recipient has been paid. Tell your customer now. |
| `transaction.reconciled` | payment | The books, the chain and the provider agree. Terminal: close your ledger now. |
| `transaction.refunded` | payment | The payment was cancelled and the full amount returned to your home wallet. Terminal. |
| `transaction.in_doubt` | payment | We cannot yet tell whether funds moved. It is being reconciled; do not retry it. |
| `batch.rejected` | batch | The batch was refused before anything moved: no item started, nothing was funded. `reason` says why. |
| `batch.completed` | batch | No item is still running. Not the same as every item paid: `items` counts them by final state. |
| `onramp.delivered` | deposit | A deposit converted and the USDC is in your wallet. |
| `onramp.refunded` | deposit | A deposit could not be completed and the payer's money was returned. |
| `offramp.paid` | off-ramp | An off-ramp paid the beneficiary. |

Payloads and signatures: [Webhooks](../webhooks/README.md).
