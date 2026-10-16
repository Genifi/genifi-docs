---
description: "One instruction, many recipients, one rail."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Batch payouts

One instruction, many recipients, one rail.

| Endpoint | |
|---|---|
| [`POST /batches`](#post-batches) | Submit a batch payout |
| [`GET /batches`](#get-batches) | List your batches |
| [`GET /batches/:batchId`](#get-batchesbatchid) | A batch, its item counts and a page of items |

## POST /batches

Submit a batch payout.

**Auth:** API key · **Permission:** `payment.initiate` (owner, admin, member) · **KYB:** approved organizations only · **Rate limit:** `initiate`

Up to 1,000 payments to one corridor on one rail, funded from your home wallet in a single transfer. A batch is accepted or refused **as a whole**; once accepted, each item is paid independently and can fail on its own. Counts against the `initiate` rate limit **once per item**.

[Read the guide](../api/batches.md)

**Request body** · `application/json` · unknown fields are **ignored**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `batchId` | string | yes | matches `^[A-Za-z0-9_.:-]{8,200}$` | Your idempotency key. 8 to 200 characters of `A-Z a-z 0-9 _ . : -`. It becomes the batch ID; resubmitting the same batch is safe. |
| `corridor` | object | yes |  |  |
| `corridor.to` | string | yes | non-empty | Destination currency, shared by every item. |
| `corridor.destinationCountry` | string | yes | exactly 2 chars | ISO 3166-1 alpha-2 country code, shared by every item. |
| `provider` | string | yes | 1–64 chars | The rail the whole batch runs on. One rail per batch, because the batch is funded at one provider. |
| `integratorWalletId` | string | no | non-empty | Optional. Must be your home wallet; omit it to use your home wallet. |
| `originator` | object | no |  | The sender, for the Travel Rule. One per batch. |
| `originator.name` | string | no | 1–200 chars |  |
| `originator.account` | string | no | 1–140 chars |  |
| `originator.address` | string | no | 1–280 chars |  |
| `originator.country` | string | no | exactly 2 chars |  |
| `originator.bic` | string | no | 8–11 chars |  |
| `originator.institution` | string | no | 1–200 chars |  |
| `originator.idNumber` | string | no | 1–80 chars |  |
| `originator.dateOfBirth` | string | no | matches `^\d{4}-\d{2}-\d{2}$` |  |
| `items` | object[] | yes | at least 1 item; at most 1000 items | Between 1 and 1,000 items, all in the same currency. |
| `items[].itemRef` | string | yes | matches `^[A-Za-z0-9_.:-]{1,64}$` | Unique within the batch. The item's transaction ID is `<batchId>:<itemRef>`. |
| `items[].sendAmount` | object | yes |  |  |
| `items[].sendAmount.amount` | string | yes | matches `^\d+(\.\d+)?$` | Decimal string. |
| `items[].sendAmount.currency` | string | yes | non-empty |  |
| `items[].recipientReference` | string | yes | non-empty | Your own reference for the recipient. |
| `items[].beneficiary` | object | no |  | Who is paid. Give this or `recipientId`. |
| `items[].beneficiary.name` | string | yes | 1–200 chars |  |
| `items[].beneficiary.account` | string | yes | 1–140 chars |  |
| `items[].beneficiary.bankCode` | string | no | 1–35 chars |  |
| `items[].beneficiary.bic` | string | no | 8–11 chars |  |
| `items[].beneficiary.bankName` | string | no | 1–200 chars |  |
| `items[].beneficiary.country` | string | no | exactly 2 chars |  |
| `items[].beneficiary.city` | string | no | 1–100 chars |  |
| `items[].beneficiary.addressLine` | string | no | 1–280 chars |  |
| `items[].beneficiary.state` | string | no | 1–100 chars |  |
| `items[].beneficiary.postalCode` | string | no | 1–20 chars |  |
| `items[].beneficiary.phone` | object | no |  |  |
| `items[].beneficiary.phone.callingCode` | string | yes | matches `^\+?\d{1,4}$` |  |
| `items[].beneficiary.phone.number` | string | yes | matches `^\d{4,15}$` |  |
| `items[].recipientId` | string | no | 1–200 chars | A saved recipient, resolved on our side. Give this or `beneficiary`. |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `400` | Invalid request | `duplicate_item_ref`, `invalid_send_amount`, `unknown_recipient`, `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden`, `kyb_required` |
| `409` | Conflict | `idempotency_key_conflict` |
| `422` | Understood, but cannot be done as asked | `ambiguous_home_wallet`, `no_home_wallet`, `not_home_wallet`, `wallet_chain_unknown` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `batchId` | string | yes |  |
| `state` | string | yes |  |
| `itemCount` | number | yes |  |
| `note` | string | yes |  |

## GET /batches

List your batches.

**Auth:** API key · **Rate limit:** `read`

Newest first, cursor-paginated.

[Read the guide](../api/batches.md#get-batches)

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
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `batches` | object[] | yes |  |
| `batches[].batchId` | string | yes |  |
| `batches[].state` | string | yes |  |
| `batches[].corridor` | object | yes |  |
| `batches[].corridor.from` | string | yes |  |
| `batches[].corridor.to` | string | yes |  |
| `batches[].corridor.destinationCountry` | string | yes |  |
| `batches[].provider` | string | yes |  |
| `batches[].itemCount` | number | yes |  |
| `batches[].totalSendAmount` | object | yes |  |
| `batches[].totalSendAmount.amount` | string | yes |  |
| `batches[].totalSendAmount.currency` | string | yes |  |
| `batches[].createdAt` | string | yes |  |
| `count` | number | yes |  |
| `nextCursor` | string or null | yes |  |
| `hasMore` | boolean | yes |  |

## GET /batches/:batchId

A batch, its item counts and a page of items.

**Auth:** API key · **Rate limit:** `read`

`items` counts every item by state; `page` lists the items themselves, cursor-paginated with `limit` and `cursor`. A batch that is not yours is `404`.

[Read the guide](../api/batches.md#get-batchesbatchid)

**Path parameters**

| Name | Type |
|---|---|
| `batchId` | string |

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
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `corridor` | object | yes |  |
| `corridor.from` | string | yes |  |
| `corridor.to` | string | yes |  |
| `corridor.destinationCountry` | string | yes |  |
| `provider` | string | yes |  |
| `chain` | string | yes |  |
| `itemCount` | number | yes |  |
| `totalSendAmount` | object | yes |  |
| `totalSendAmount.amount` | string | yes |  |
| `totalSendAmount.currency` | string | yes |  |
| `items` | map of number | yes |  |
| `createdAt` | string | yes |  |
| `updatedAt` | string | yes |  |
| `page` | object | yes |  |
| `page.items` | object[] | yes |  |
| `page.items[].failureReason` | string | no |  |
| `page.items[].itemRef` | string | yes |  |
| `page.items[].transactionId` | string | yes |  |
| `page.items[].state` | string | yes |  |
| `page.items[].sendAmount` | object | yes |  |
| `page.items[].sendAmount.amount` | string | yes |  |
| `page.items[].sendAmount.currency` | string | yes |  |
| `page.items[].recipientReference` | string | yes |  |
| `page.nextCursor` | string or null | yes |  |
| `page.hasMore` | boolean | yes |  |
| `rejectedReason` | string | no |  |
| `batchId` | string | yes |  |
| `state` | string | yes |  |
