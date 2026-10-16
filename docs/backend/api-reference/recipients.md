---
description: "Your saved payees, reusable across payments and batches."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Recipients

Your saved payees, reusable across payments and batches.

| Endpoint | |
|---|---|
| [`GET /recipients`](#get-recipients) | Your saved recipients |
| [`POST /recipients`](#post-recipients) | Save a recipient |
| [`PATCH /recipients/:id`](#patch-recipientsid) | Edit a saved recipient |
| [`DELETE /recipients/:id`](#delete-recipientsid) | Delete a saved recipient |

## GET /recipients

Your saved recipients.

**Auth:** API key · **Permission:** `recipient.read` (owner, admin, member, viewer) · **Rate limit:** `read`

[Read the guide](../api/recipients.md#get-recipients)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `recipients` | object[] | yes |  |
| `recipients[].id` | string | yes |  |
| `recipients[].name` | string | yes |  |
| `recipients[].type` | [SavedRecipientType](types.md#savedrecipienttype) | yes |  |
| `recipients[].account` | string or null | yes |  |
| `recipients[].bankCode` | string or null | yes |  |
| `recipients[].bic` | string or null | yes |  |
| `recipients[].bankName` | string or null | yes |  |
| `recipients[].country` | string or null | yes |  |
| `recipients[].city` | string or null | yes |  |
| `recipients[].addressLine` | string or null | yes |  |
| `recipients[].state` | string or null | yes |  |
| `recipients[].postalCode` | string or null | yes |  |
| `recipients[].phoneCallingCode` | string or null | yes |  |
| `recipients[].phoneNumber` | string or null | yes |  |
| `recipients[].address` | string or null | yes |  |
| `recipients[].networkId` | string or null | yes |  |
| `recipients[].lastUsedAt` | string or null | yes |  |
| `recipients[].createdAt` | string | yes |  |
| `recipients[].updatedAt` | string | yes |  |
| `count` | number | yes |  |

## POST /recipients

Save a recipient.

**Auth:** API key · **Permission:** `recipient.manage` (owner, admin, member) · **Rate limit:** `write`

A `bank` recipient needs an `account`; a `wallet` recipient needs an `address`.

[Read the guide](../api/recipients.md#post-recipients)

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `name` | string | yes | 1–200 chars |  |
| `type` | `"wallet"`, `"bank"` | yes |  |  |
| `account` | string | no | 1–140 chars |  |
| `bankCode` | string | no | 1–40 chars |  |
| `bic` | string | no | 8–11 chars |  |
| `bankName` | string | no | 1–200 chars |  |
| `country` | string | no | exactly 2 chars |  |
| `city` | string | no | 1–120 chars |  |
| `addressLine` | string | no | 1–280 chars |  |
| `state` | string | no | 1–100 chars |  |
| `postalCode` | string | no | 1–20 chars |  |
| `phoneCallingCode` | string | no | matches `^\+?\d{1,4}$` |  |
| `phoneNumber` | string | no | matches `^\d{4,15}$` |  |
| `address` | string | no | 1–200 chars |  |
| `networkId` | string | no | 1–60 chars |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `201` | Created |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`201` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `name` | string | yes |  |
| `type` | [SavedRecipientType](types.md#savedrecipienttype) | yes |  |
| `account` | string or null | yes |  |
| `bankCode` | string or null | yes |  |
| `bic` | string or null | yes |  |
| `bankName` | string or null | yes |  |
| `country` | string or null | yes |  |
| `city` | string or null | yes |  |
| `addressLine` | string or null | yes |  |
| `state` | string or null | yes |  |
| `postalCode` | string or null | yes |  |
| `phoneCallingCode` | string or null | yes |  |
| `phoneNumber` | string or null | yes |  |
| `address` | string or null | yes |  |
| `networkId` | string or null | yes |  |
| `lastUsedAt` | string or null | yes |  |
| `createdAt` | string | yes |  |
| `updatedAt` | string | yes |  |

## PATCH /recipients/:id

Edit a saved recipient.

**Auth:** API key · **Permission:** `recipient.manage` (owner, admin, member) · **Rate limit:** `write`

Send only the fields to change. `null` clears a field. `type` cannot change.

[Read the guide](../api/recipients.md#patch-recipientsid)

**Path parameters**

| Name | Type |
|---|---|
| `id` | string |

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `name` | string | no | 1–200 chars |  |
| `account` | string or null | no |  |  |
| `bankCode` | string or null | no |  |  |
| `bic` | string or null | no |  |  |
| `bankName` | string or null | no |  |  |
| `country` | string or null | no |  |  |
| `city` | string or null | no |  |  |
| `addressLine` | string or null | no |  |  |
| `state` | string or null | no |  |  |
| `postalCode` | string or null | no |  |  |
| `phoneCallingCode` | string or null | no |  |  |
| `phoneNumber` | string or null | no |  |  |
| `address` | string or null | no |  |  |
| `networkId` | string or null | no |  |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `name` | string | yes |  |
| `type` | [SavedRecipientType](types.md#savedrecipienttype) | yes |  |
| `account` | string or null | yes |  |
| `bankCode` | string or null | yes |  |
| `bic` | string or null | yes |  |
| `bankName` | string or null | yes |  |
| `country` | string or null | yes |  |
| `city` | string or null | yes |  |
| `addressLine` | string or null | yes |  |
| `state` | string or null | yes |  |
| `postalCode` | string or null | yes |  |
| `phoneCallingCode` | string or null | yes |  |
| `phoneNumber` | string or null | yes |  |
| `address` | string or null | yes |  |
| `networkId` | string or null | yes |  |
| `lastUsedAt` | string or null | yes |  |
| `createdAt` | string | yes |  |
| `updatedAt` | string | yes |  |

## DELETE /recipients/:id

Delete a saved recipient.

**Auth:** API key · **Permission:** `recipient.manage` (owner, admin, member) · **Rate limit:** `write`

[Read the guide](../api/recipients.md#delete-recipientsid)

**Path parameters**

| Name | Type |
|---|---|
| `id` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `name` | string | yes |  |
| `type` | [SavedRecipientType](types.md#savedrecipienttype) | yes |  |
| `account` | string or null | yes |  |
| `bankCode` | string or null | yes |  |
| `bic` | string or null | yes |  |
| `bankName` | string or null | yes |  |
| `country` | string or null | yes |  |
| `city` | string or null | yes |  |
| `addressLine` | string or null | yes |  |
| `state` | string or null | yes |  |
| `postalCode` | string or null | yes |  |
| `phoneCallingCode` | string or null | yes |  |
| `phoneNumber` | string or null | yes |  |
| `address` | string or null | yes |  |
| `networkId` | string or null | yes |  |
| `lastUsedAt` | string or null | yes |  |
| `createdAt` | string | yes |  |
| `updatedAt` | string | yes |  |
