# Balance and wallets

You hold **one USDC balance**. It lives in your **home wallet**, a wallet JET
provisions for you at onboarding on your home network. Payments, batches,
withdrawals and off-ramps are all funded from it, and deposits land in it.

Every field is in the [API reference](../api-reference/balance.md).

## You never pick a network

USDC on one network is not the same asset as USDC on another, so JET keeps
your balance on one: your home network. You do not name a wallet or a network
when you pay. If a payment request does name a wallet, it must be the home wallet
(`422 not_home_wallet` otherwise), because USDC anywhere else is on its way home.

USDC can still *arrive* on other networks: when you or a payer
[send USDC on chain](treasury.md#usdc-deposits), you choose the network. Anything
that lands on a network other than your home one is moved home automatically,
and shows as `settling` until it gets there.

## GET /balance

=== "Request"

    ```bash
    curl "$JET_API/balance" -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "currency": "USDC",
      "scale": 6,
      "available": "2500000000",
      "settling": "120000000",
      "total": "2620000000",
      "complete": true,
      "reportedAt": "2026-08-21T14:22:11.000Z"
    }
    ```

| Field | Notes |
|---|---|
| `available` | On your home network. What a payment, withdrawal or off-ramp can spend **now** |
| `settling` | On other networks, on its way home. Not spendable yet |
| `total` | `available` + `settling` |
| `complete` | `false` if a wallet could not be read. The figures are then a **floor**, never a claim that the unread wallet is empty |
| `reportedAt` | When the figures were read |

Amounts are integer minor units at `scale`: `2500000000` is 2,500 USDC. See
[Amounts and currencies](../concepts/money.md).

`501` means no wallet provider is configured on this environment, which is not
the same answer as an empty balance.

## Why listings carry no balances

JET's ledger is the source of truth for value. What a wallet provider reports
is a *report*, reconciled against the ledger, never treated as authoritative.
So account listings carry no balances, and every balance JET returns is stamped
with when it was read.

## GET /wallets

Your wallets.

=== "Request"

    ```bash
    curl "$JET_API/wallets" -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "accounts": [
        {
          "id": "acct_9c1a…",
          "kind": "wallet",
          "provider": "dfns",
          "externalId": "wa-01k…",
          "label": "USDC",
          "currencies": ["USDC"],
          "address": "0x…",
          "chain": "polygon",
          "status": "active",
          "createdAt": "2026-07-02T09:14:00.000Z"
        }
      ],
      "count": 1
    }
    ```

| Field | Notes |
|---|---|
| `id` | JET's ID for the registration |
| `externalId` | The wallet's ID at its provider. Either ID works wherever a wallet ID is accepted |
| `address` | The on-chain address |
| `chain` | The network the wallet is on. `null` means it is not recorded, and is never guessed |
| `status` | `active` when usable |

You may hold a wallet on every network JET supports, at the same address. Only
the one on your home network is your spendable balance.

## GET /wallets/:walletId

One wallet, by `id` or `externalId`. Same shape as a listing entry. A wallet you
do not own returns `404`, not `403`; see
[Conventions](conventions.md#404-also-means-not-yours).

## GET /wallets/:walletId/balances

What the wallet provider reports one of your wallets holds, per asset, with
`reportedAt`. `501` when no wallet provider is configured.

For "how much can I spend", use [`GET /balance`](#get-balance); this endpoint is
for looking at one wallet.

## Issuer accounts

Issuer accounts held for you, in the same shape as a wallet listing with a
different `kind`:

```bash
curl "$JET_API/issuers/accounts" -H "Authorization: Bearer $JET_API_KEY"
```

## Issuer directory

The stablecoin issuers this environment supports:

```bash
curl "$JET_API/issuers" -H "Authorization: Bearer $JET_API_KEY"
```

```json
{ "issuers": [ … ], "count": 2 }
```

Returns `501` where no issuer is configured.

## Endpoints that are not available

These exist in the contract and answer `501` with a reason on every environment
today. Check [`/capabilities`](discovery.md#get-capabilities) rather than calling
them speculatively.

| Endpoint | Instead |
|---|---|
| `POST /wallets/:walletId/transfers` | Not needed: your balance is one wallet |
| `POST /issuers/:issuerId/mint` | Deposit fiat: it is converted to USDC for you. See [Deposits](treasury.md) |
| `POST /banking/wire-transfers` | Pay fiat out with an [off-ramp](withdrawals.md#off-ramps) |

```json
{
  "error": "not_implemented",
  "feature": "banking",
  "message": "…",
  "requires": "…"
}
```

## Removed

These no longer exist. Each now answers `404`.

| Endpoint | Why, and what to use |
|---|---|
| `POST /wallets` | Wallets are provisioned by JET at onboarding. You never register one |
| `GET /issuers/:issuerId/assets`, `GET /issuers/:issuerId/balances` | They reported JET's own position at the issuer, not yours |
| `GET /banking/accounts`, `GET /banking/accounts/:id/balances` | JET holds no bank account for you. Where to send money is [`GET /treasury/deposit-instructions`](treasury.md#get-treasurydeposit-instructions) |

Redemption (`POST /issuers/:issuerId/redeem`) is closed to integrators and answers
`501`: to turn USDC into fiat, use an [off-ramp](withdrawals.md#off-ramps).
