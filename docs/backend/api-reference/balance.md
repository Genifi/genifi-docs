---
description: "Your USDC balance and the wallets that hold it."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Balance and wallets

Your USDC balance and the wallets that hold it.

| Endpoint | |
|---|---|
| [`GET /balance`](#get-balance) | Your USDC balance |
| [`GET /wallets`](#get-wallets) | Your wallets |
| [`GET /wallets/:walletId`](#get-walletswalletid) | One of your wallets |
| [`GET /wallets/:walletId/balances`](#get-walletswalletidbalances) | What the wallet provider reports a wallet holds |
| [`GET /issuers/accounts`](#get-issuersaccounts) | Issuer accounts held for you |
| [`GET /issuers`](#get-issuers) | The stablecoin issuers this environment supports |
| [`POST /wallets/:walletId/transfers`](#post-walletswalletidtransfers) | Move funds between your wallets |
| [`POST /issuers/:issuerId/mint`](#post-issuersissueridmint) | Mint a stablecoin at par |
| [`POST /banking/wire-transfers`](#post-bankingwire-transfers) | Send an outbound wire |

## GET /balance

Your USDC balance.

**Auth:** API key · **Rate limit:** `read`

`available` is what you can spend now; `settling` is USDC that arrived on another network and is on its way to your home wallet; `total` is both. `complete: false` means a read failed and the figures are a floor.

[Read the guide](../api/wallets-and-accounts.md#get-balance)

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
| `currency` | string | yes |  |
| `scale` | `6` | yes |  |
| `available` | string | yes |  |
| `settling` | string | yes |  |
| `total` | string | yes |  |
| `complete` | boolean | yes |  |
| `reportedAt` | string | yes |  |

## GET /wallets

Your wallets.

**Auth:** API key · **Rate limit:** `read`

The wallets registered to your organization. No balances: those are a separate read.

[Read the guide](../api/wallets-and-accounts.md#get-wallets)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `accounts` | object[] | yes |  |
| `accounts[].id` | string | yes |  |
| `accounts[].kind` | [AccountKindName](types.md#accountkindname) | yes |  |
| `accounts[].provider` | string | yes |  |
| `accounts[].externalId` | string | yes |  |
| `accounts[].label` | string or null | yes |  |
| `accounts[].currencies` | string[] | yes |  |
| `accounts[].address` | string or null | yes |  |
| `accounts[].chain` | string or null | yes |  |
| `accounts[].status` | `"active"`, `"disabled"` | yes |  |
| `accounts[].createdAt` | string | yes |  |
| `count` | number | yes |  |

## GET /wallets/:walletId

One of your wallets.

**Auth:** API key · **Rate limit:** `read`

By our `id` or the wallet's `externalId`. A wallet that is not yours is `404`.

[Read the guide](../api/wallets-and-accounts.md#get-walletswalletid)

**Path parameters**

| Name | Type |
|---|---|
| `walletId` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `kind` | [AccountKindName](types.md#accountkindname) | yes |  |
| `provider` | string | yes |  |
| `externalId` | string | yes |  |
| `label` | string or null | yes |  |
| `currencies` | string[] | yes |  |
| `address` | string or null | yes |  |
| `chain` | string or null | yes |  |
| `status` | `"active"`, `"disabled"` | yes |  |
| `createdAt` | string | yes |  |

## GET /wallets/:walletId/balances

What the wallet provider reports a wallet holds.

**Auth:** API key · **Rate limit:** `read`

A provider report, stamped with when it was read. `501` when no wallet provider is configured, which is not the same answer as an empty wallet.

[Read the guide](../api/wallets-and-accounts.md#get-walletswalletidbalances)

**Path parameters**

| Name | Type |
|---|---|
| `walletId` | string |

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
| `walletId` | string | yes |  |
| `balances` | [ReportedBalance](types.md#reportedbalance)[] | yes |  |

## GET /issuers/accounts

Issuer accounts held for you.

**Auth:** API key · **Rate limit:** `read`

[Read the guide](../api/wallets-and-accounts.md#issuer-accounts)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `accounts` | object[] | yes |  |
| `accounts[].id` | string | yes |  |
| `accounts[].kind` | [AccountKindName](types.md#accountkindname) | yes |  |
| `accounts[].provider` | string | yes |  |
| `accounts[].externalId` | string | yes |  |
| `accounts[].label` | string or null | yes |  |
| `accounts[].currencies` | string[] | yes |  |
| `accounts[].address` | string or null | yes |  |
| `accounts[].chain` | string or null | yes |  |
| `accounts[].status` | `"active"`, `"disabled"` | yes |  |
| `accounts[].createdAt` | string | yes |  |
| `count` | number | yes |  |

## GET /issuers

The stablecoin issuers this environment supports.

**Auth:** API key · **Rate limit:** `read`

[Read the guide](../api/wallets-and-accounts.md#issuer-directory)

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
| `issuers` | [IssuerCatalogEntry](types.md#issuercatalogentry)[] | yes |  |
| `count` | number | yes |  |

## POST /wallets/:walletId/transfers

Move funds between your wallets.

**Auth:** API key · **Rate limit:** `write`

**Not available:** Not offered yet. Answers `501` with a reason on every environment.

**Path parameters**

| Name | Type |
|---|---|
| `walletId` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

## POST /issuers/:issuerId/mint

Mint a stablecoin at par.

**Auth:** API key · **Rate limit:** `write`

**Not available:** Not offered yet. Answers `501` with a reason on every environment.

**Path parameters**

| Name | Type |
|---|---|
| `issuerId` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

## POST /banking/wire-transfers

Send an outbound wire.

**Auth:** API key · **Rate limit:** `write`

**Not available:** Not offered. To pay fiat out of your balance, use an off-ramp (`POST /treasury/offramp`).

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |
