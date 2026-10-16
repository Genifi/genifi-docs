# Withdrawals and off-ramps

Two ways to take money out of your JET balance:

| Way out | Arrives as | Start with |
|---|---|---|
| [Withdrawal](#withdrawals) | USDC, in a wallet **you** control | [`GET /treasury/withdrawals/quote`](#get-treasurywithdrawalsquote) |
| [Off-ramp](#off-ramps) | Fiat, in a bank account you registered | [`POST /treasury/beneficiaries`](#post-treasurybeneficiaries) |

Both are funded from your [home wallet](wallets-and-accounts.md), both need the
`treasury.move` permission and an approved KYB status, and both are durable
operations you follow like a payment. Every field is in the
[API reference](../api-reference/withdrawals.md).

To pay **someone else** in their local currency, send a
[payment](transactions.md) instead.

## Withdrawals

Sends USDC from your balance to an address you give, on the network you choose.
This and [USDC deposits](treasury.md#usdc-deposits) are the only places you name
a network.

!!! danger "An on-chain send cannot be reversed"

    Check the address, and check that it is on the network you name. USDC sent to
    the right address on the wrong network, or to a mistyped address, cannot be
    recovered by JET or anyone else.

### GET /treasury/withdrawals/quote

=== "Request"

    ```bash
    curl "$JET_API/treasury/withdrawals/quote?amount=250.00&chain=arbitrum" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "amount": { "amount": "250000000", "currency": "USDC", "scale": 6 },
      "network": "arbitrum",
      "fee": { "amount": "180000", "currency": "USDC", "scale": 6 },
      "net": { "amount": "249820000", "currency": "USDC", "scale": 6 }
    }
    ```

=== "422 Unsupported network"

    ```json
    {
      "error": "unsupported_network",
      "message": "this network is not available for withdrawals"
    }
    ```

`net` is what arrives. A withdrawal on your home network costs the network's gas;
one on another network also costs moving the USDC there. The networks you can
use are the ones listed by
[`GET /treasury/crypto-deposit`](treasury.md#usdc-deposits).

### POST /treasury/withdrawals

=== "Request"

    ```bash
    curl -X POST "$JET_API/treasury/withdrawals" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "wd-2026-08-21-a",
        "amount": { "amount": "250.00", "currency": "USDC" },
        "chain": "arbitrum",
        "toAddress": "0x8f3C…a91B"
      }'
    ```

=== "202 Accepted"

    ```json
    {
      "withdrawalId": "wd-2026-08-21-a",
      "amount": { "amount": "250000000", "currency": "USDC", "scale": 6 },
      "network": "arbitrum",
      "fee": { "amount": "180000", "currency": "USDC", "scale": 6 },
      "net": { "amount": "249820000", "currency": "USDC", "scale": 6 }
    }
    ```

- **It is re-priced when you send it**, with no quote ID. The fee includes headroom
  for gas moving between your quote and the send, and the response carries the
  figure that applied.
- **`idempotencyKey` becomes the withdrawal ID.** Retrying the same request is
  safe. Reusing the key with a different amount, network or address is
  `409 idempotency_key_reused`; a key already used for someone else's withdrawal is
  `409 idempotency_key_conflict`.

### GET /treasury/withdrawals/:withdrawalId

```json
{
  "withdrawalId": "wd-2026-08-21-a",
  "state": "SENT",
  "running": false,
  "net": { "amount": "249820000", "currency": "USDC", "scale": 6 },
  "txHash": "0x…",
  "timeline": [
    { "state": "INITIATED" },
    { "state": "PULLING" },
    { "state": "HELD" },
    { "state": "CROSSING" },
    { "state": "SENDING" },
    { "state": "SENT" }
  ]
}
```

| `state` | Meaning |
|---|---|
| `SENT` | Confirmed on chain. `txHash` is the transaction. Done |
| `PENDING_RETRY` | The send has not been confirmed yet and is being retried |
| `NEEDS_ATTENTION` | Waiting for our operations team. The funds are held, not lost |
| `RETURNING`, `RETURNED` | The withdrawal was cancelled and the USDC is going, or has gone, back to your balance |
| `FAILED` | It stopped before any USDC moved |
| `IN_DOUBT` | We cannot yet tell whether the send landed. Parked for reconciliation; do not send a replacement |

A withdrawal counts as done only when the send is **confirmed** on chain. A send
that cannot be confirmed is parked rather than repeated, because repeating a send
that did land would pay twice.

`GET /treasury/withdrawals` lists your withdrawals, newest first, cursor-paginated
(`limit` up to 200). Rows carry no state.

## Off-ramps

Converts USDC from your balance into fiat and pays it to a bank account. The
account must be registered first, as a **beneficiary**: an off-ramp names a
beneficiary by ID and never carries bank details itself.

!!! info

    Off-ramps are not enabled on every environment. Where they are not, these
    endpoints answer `501`. Check [`/capabilities`](discovery.md#get-capabilities).

### POST /treasury/beneficiaries

=== "Request"

    ```bash
    curl -X POST "$JET_API/treasury/beneficiaries" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "Northwind Payments GmbH",
        "accountId": "DE89370400440532013000",
        "bic": "COBADEFFXXX",
        "bankName": "Commerzbank",
        "city": "Frankfurt am Main",
        "addressLine": "1 Kaiserstrasse",
        "country": "DE",
        "currency": "EUR"
      }'
    ```

=== "201 Created"

    ```json
    {
      "id": "6e40ab02-1054-42be-86f3-98f5423105c2",
      "name": "Northwind Payments GmbH",
      "accountId": "DE89370400440532013000",
      "bic": "COBADEFFXXX",
      "bankName": "Commerzbank",
      "city": "Frankfurt am Main",
      "addressLine": "1 Kaiserstrasse",
      "country": "DE",
      "currency": "EUR",
      "createdAt": "2026-08-21T10:02:00.000Z"
    }
    ```

Needs the `beneficiary.manage` permission. Every field is required: a bank
payment rejected for an incomplete creditor costs a whole new off-ramp, so the
check happens here. The beneficiary is paid in its `currency`.

!!! warning

    Registering is **your** assertion that the account is right. Nothing here
    confirms the account exists or belongs to the name you gave. Registration has
    no idempotency key, so a retried request registers the beneficiary twice; list
    first if you are unsure.

`GET /treasury/beneficiaries` lists them. They cannot be edited or deleted
through the API today.

### GET /treasury/offramp/quote

What the beneficiary receives for this much USDC.

=== "Request"

    ```bash
    curl "$JET_API/treasury/offramp/quote?amount=500.00&beneficiaryId=6e40ab02-1054-42be-86f3-98f5423105c2" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "pulled": { "amount": "500000000", "currency": "USDC", "scale": 6 },
      "settlementCurrency": "EUR",
      "atPar": false,
      "feeBps": 0,
      "fee": { "amount": "0", "currency": "EUR", "scale": 2 },
      "deliveryFee": { "amount": "203", "currency": "EUR", "scale": 2 },
      "minimum": { "amount": "204", "currency": "EUR", "scale": 2 },
      "net": { "amount": "42612", "currency": "EUR", "scale": 2 },
      "fxBufferBps": 75,
      "quoteId": "oq_5b1c7e0a-…",
      "expiresAt": "2026-08-21T10:05:00.000Z"
    }
    ```

- **`net`** is what the beneficiary receives. Where the conversion rate is not yet
  known, the quote holds `fxBufferBps` back from the expected rate so that `net`
  is a figure JET can honour.
- **`quoteId`** is present when `net` is. Pass it to `POST /treasury/offramp` and
  the beneficiary is paid exactly that `net`. Without it, the off-ramp pays
  whatever the conversion produces.

### POST /treasury/offramp

=== "Request"

    ```bash
    curl -X POST "$JET_API/treasury/offramp" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "offramp-2026-08-21-a",
        "amount": { "amount": "500.00", "currency": "USDC" },
        "beneficiaryId": "6e40ab02-1054-42be-86f3-98f5423105c2",
        "quoteId": "oq_5b1c7e0a-…"
      }'
    ```

=== "202 Accepted"

    ```json
    { "offRampId": "offramp-2026-08-21-a", "status": "accepted" }
    ```

The `amount` and `beneficiaryId` must match the quote, or the request is refused
with `422` before anything moves. An expired or already-used quote is refused the
same way; request a new one. `chain` is accepted only as your home network; omit
it.

### GET /treasury/offramp/:offRampId

```json
{
  "offRampId": "offramp-2026-08-21-a",
  "stage": "PAID",
  "state": "PAID",
  "net": { "amount": "42612", "currency": "EUR", "scale": 2 },
  "fee": { "amount": "0", "currency": "EUR", "scale": 2 },
  "deliveryFee": { "amount": "203", "currency": "EUR", "scale": 2 },
  "timeline": [
    { "stage": "PULLING", "at": "1" },
    { "stage": "PULLED", "at": "2" },
    { "stage": "DEPOSITING", "at": "3" },
    { "stage": "CREDITED", "at": "4" },
    { "stage": "CONVERTING", "at": "5" },
    { "stage": "SETTLING", "at": "6" },
    { "stage": "PAYING", "at": "7" },
    { "stage": "PAID", "at": "8" }
  ]
}
```

`stage` is where it is now; `state` is how it ended, once it has: `PAID`,
`REFUNDED` (your USDC came back), `FAILED`, or `IN_DOUBT`. In the timeline, `at`
orders the events. **It is not a timestamp.**

You also get an `offramp.paid`
[webhook](../webhooks/events.md#deposits-and-off-ramps) when the beneficiary is
paid. `GET /treasury/offramps` lists your off-ramps, newest first,
cursor-paginated (`limit` up to 200).
