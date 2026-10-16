# Deposits

Three ways to get money into your JET balance. Each one ends as USDC in your
[home wallet](wallets-and-accounts.md):

| Way in | The payer sends | Start with |
|---|---|---|
| [Bank wire](#get-treasurydeposit-instructions) | EUR or USD, from a bank | [`GET /treasury/deposit-instructions`](#get-treasurydeposit-instructions) |
| [PIX](#pix-deposits) | BRL, by PIX | [`POST /treasury/onramp/pix`](#pix-deposits) |
| [USDC on chain](#usdc-deposits) | USDC, from an exchange or a wallet | [`GET /treasury/crypto-deposit`](#usdc-deposits) |

A deposit is not a payment: it has no corridor and no recipient. Bank and PIX
deposits are durable operations, called **on-ramps**, that you follow the same
way as a payment. Every field is in the [API reference](../api-reference/deposits.md).

Moving money is KYB-gated and needs the `treasury.move` permission; reading
prices and lists is not.

## GET /treasury/deposit-instructions

JET's bank details and **your reference**: everything a payer needs to wire
fiat in. Standing, so you can read it before starting anything.

=== "Request"

    ```bash
    curl "$JET_API/treasury/deposit-instructions" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "reference": "GEN-7K2QX4C9J",
      "accounts": [
        {
          "currency": "USD",
          "accountName": "…",
          "bankName": "Standard Chartered Bank (US)",
          "accountNumber": "3582021234001",
          "bic": "SCBLUS33XXX"
        },
        {
          "currency": "EUR",
          "accountName": "…",
          "bankName": "Standard Chartered Bank (DE)",
          "iban": "DE89370400440532013000",
          "bic": "SCBLDEFXXXX"
        }
      ]
    }
    ```

=== "403 KYB required"

    ```json
    { "error": "kyb_required", "message": "KYB approval required" }
    ```

=== "501 Not wired"

    ```json
    {
      "error": "not_implemented",
      "feature": "deposit-instructions",
      "message": "no house account is configured on this environment"
    }
    ```

| Field | Type | Notes |
|---|---|---|
| `reference` | string | The code the payer quotes. **One per organization**, not per currency |
| `accounts[].currency` | string | `USD` or `EUR` |
| `accounts[].accountName` | string | The beneficiary: our legal entity, never you |
| `accounts[].bankName` | string | |
| `accounts[].accountNumber` | string | Present for an account routed by number |
| `accounts[].iban` | string | Present for an account routed by IBAN |
| `accounts[].bic` | string | BIC/SWIFT, where we hold one |
| `accounts[].bankAddress` | string | Where we hold one |

**Only the fields we actually hold are returned.** A field we do not have is
omitted rather than sent empty: a wire instruction gets retyped into a bank by a
person, and a blank or guessed line is a payment that disappears. Likewise, a
currency with no configured account is simply absent from `accounts`.

### The reference is the whole attribution mechanism

Every customer wires to the **same** JET account, so the bank cannot tell one
payer from another. The reference on the transfer is the only thing that says
whose money arrived.

- It identifies **your organization**, not the transfer and not the currency. The
  same code goes on a USD wire and on a EUR one.
- It is stable. Reading this endpoint again returns the code you already have,
  because reissuing would silently invalidate instructions a payer may already be
  acting on.
- Where you have several deposits open at once, currency, amount and arrival
  window are what tell them apart.
- A transfer that arrives with no reference, or a mangled one, is **not** matched
  on amount. It parks for manual reconciliation. Crediting a guess is worse than
  waiting.

It is not a secret and not a credential. Quoting someone else's would credit
*them*, which is exactly why it carries a check character: it is built to
survive a typo being caught, not to be hidden.

### Send USD to the USD account, EUR to the EUR account

The accounts are not interchangeable. A EUR transfer into the USD account is a
cross-currency credit our statement matching will not attribute, and it needs
tracing by hand.

### Why this read is KYB-gated

Reads are otherwise open to a pending integrator; the KYB gate is on value
moving, not on looking. This is the exception, because it is a read that *causes*
value to move: these instructions are the mechanism by which money enters, and
issuing them before verification means someone's wire lands in JET's account
against an organization nobody has diligenced.

!!! info "Also in the dashboard"

    **Deposits & Withdrawals → Your deposit details** shows the same reference and
    the same per-currency bank details, with per-field copy buttons.

## GET /treasury/onramp/quote

What a wire deposit of this size costs and delivers, without starting anything.

=== "Request"

    ```bash
    curl "$JET_API/treasury/onramp/quote?amount=5000.00&currency=USD" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK (USD)"

    ```json
    {
      "sent": { "amount": "500000", "currency": "USD", "scale": 2 },
      "targetStablecoin": "USDC",
      "feeBps": 5,
      "deliveryFee": { "amount": "45000000", "currency": "USDC", "scale": 6 },
      "deliveryMethod": "mint+transfer",
      "minimum": { "amount": "2000", "currency": "USD", "scale": 2 },
      "atPar": true,
      "gross": { "amount": "5000000000", "currency": "USDC", "scale": 6 },
      "fee": { "amount": "2500000", "currency": "USDC", "scale": 6 },
      "net": { "amount": "4952500000", "currency": "USDC", "scale": 6 }
    }
    ```

- **`minimum`** is the smallest deposit accepted. A quote below it is not an
  error, so you can quote as the payer types; starting one is.
- **`gross`, `fee` and `net` are exact only for USD** (`atPar: true`): USD
  converts to USDC one for one. For EUR (`atPar: false`) they are left out,
  because the euro rate is known only when the conversion runs. `feeBps` and
  `deliveryFee` still tell you what will be deducted.
- **`deliveryFee`** is the cost of getting the USDC to your wallet, deducted from
  what you receive.

## POST /treasury/onramp

Opens a wire deposit. Use it when you want an ID for the deposit before the money
is sent. A wire that arrives quoting your reference with no deposit open starts
one by itself; see [`GET /treasury/onramps`](#get-treasuryonramps).

=== "Request"

    ```bash
    curl -X POST "$JET_API/treasury/onramp" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "onramp-2026-08-21-a",
        "amount": { "amount": "50000.00", "currency": "EUR" }
      }'
    ```

=== "202 Accepted"

    ```json
    {
      "onRampId": "onramp-2026-08-21-a",
      "status": "accepted",
      "instructions": {
        "reference": "…",
        "accountName": "…",
        "iban": "…",
        "bic": "…"
      }
    }
    ```

- **`idempotencyKey` becomes the on-ramp ID.** Reusing it for a different
  deposit is `409 idempotency_key_conflict`.
- **The USDC goes to your home wallet.** `destinationAddress` is still accepted
  for older integrations and ignored.
- **`instructions`** says where to send the wire and the **reference** to quote.
  It is absent when the environment cannot produce them; the bank details and
  reference from [`GET /treasury/deposit-instructions`](#get-treasurydeposit-instructions)
  apply regardless.

Below the minimum is a `400`. With no home wallet to deliver to, `422
no_home_wallet`.

## GET /treasury/onramps

Your deposits, newest first, cursor-paginated (`limit` up to 500, `cursor`). Filter
with `origin`:

| `origin` | The deposit was started by |
|---|---|
| `api` | You, with `POST /treasury/onramp` |
| `deposit` | A wire that arrived quoting your reference. Its ID is `dep-…`, which you never chose: this list is how you find it |
| `pix` | You, with `POST /treasury/onramp/pix` |

Rows carry no state. Read each deposit's status from
[`GET /treasury/onramp/:onRampId`](#get-treasuryonramponrampid).

## GET /treasury/onramp/:onRampId

=== "Request"

    ```bash
    curl "$JET_API/treasury/onramp/onramp-2026-08-21-a" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "onRampId": "onramp-2026-08-21-a",
      "running": false,
      "state": "DELIVERED",
      "stage": "DELIVERED",
      "gross": { "amount": "54165000000", "currency": "USDC" },
      "fee": { "amount": "27080000", "currency": "USDC" },
      "net": { "amount": "54092920000", "currency": "USDC" },
      "txHash": "0x…",
      "timeline": [
        { "stage": "AWAITING_PAYMENT", "at": "2026-08-21T09:00:02.000Z" },
        { "stage": "PAYMENT_RECEIVED", "at": "2026-08-21T13:41:10.000Z" },
        { "stage": "CONVERTING", "at": "2026-08-21T13:41:12.000Z" },
        { "stage": "DELIVERING", "at": "2026-08-21T13:44:40.000Z" },
        { "stage": "DELIVERED", "at": "2026-08-21T13:46:05.000Z" }
      ]
    }
    ```

`stage` is where a running deposit is now; `state` is how it ended, once it has.

| `state` | Meaning |
|---|---|
| `DELIVERED` | The USDC is in your wallet. Done |
| `REFUNDED` | The deposit unwound and the payer's money was returned |
| `FAILED` | It stopped before delivery |
| `EXPIRED` | The payment never arrived within the window |
| `IN_DOUBT` | We cannot yet tell whether funds moved. Parked for reconciliation |

`EXPIRED` is the common one in practice and it is usually benign: the deposit was
opened but the money was never sent, or was sent too late. Open a new one with a
new key.

You also get an `onramp.delivered` or `onramp.refunded`
[webhook](../webhooks/events.md#deposits-and-off-ramps).

## PIX deposits

A payer in Brazil pays BRL by PIX; USDC is sent to your home wallet. Delivery is
confirmed **on chain** before anything is credited, not on the provider's word.

First price it. The price is live but indicative: it is fixed when the PIX code is
opened, seconds later.

```bash
curl "$JET_API/treasury/onramp/pix/quote?amount=500.00" \
  -H "Authorization: Bearer $JET_API_KEY"
```

```json
{
  "sent": { "amount": "50000", "currency": "BRL", "scale": 2 },
  "targetStablecoin": "USDC",
  "feeBps": 50,
  "chain": "polygon",
  "available": true,
  "indicative": true,
  "net": { "amount": "89912000", "currency": "USDC", "scale": 6 },
  "fee": { "amount": "452000", "currency": "USDC", "scale": 6 },
  "providerFees": { … },
  "provider": "avenia",
  "minimum": { "amount": "1000", "currency": "BRL", "scale": 2 },
  "maximum": { "amount": "100000", "currency": "BRL", "scale": 2 }
}
```

`available: false`, with a `reason`, means it cannot be priced right now. Respect
`minimum` and `maximum`: outside them, opening a code is a `400`.

Then open the PIX code:

```bash
curl -X POST "$JET_API/treasury/onramp/pix" \
  -H "Authorization: Bearer $JET_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "idempotencyKey": "pix-2026-08-21-a",
    "amount": { "amount": "500.00", "currency": "BRL" }
  }'
```

```json
{
  "onRampId": "pix-2026-08-21-a",
  "status": "accepted",
  "instructions": {
    "reference": "…",
    "brCode": "00020126…",
    "pay": { "amount": "50000", "currency": "BRL", "scale": 2 },
    "receive": { "amount": "89912000", "currency": "USDC", "scale": 6 },
    "expiresAt": "2026-08-21T09:30:00.000Z"
  }
}
```

Show `brCode` to the payer as a QR code. Retrying with the same key returns the
**same** code, so a retry never gives the payer a second code to pay. Follow it
like any deposit, with [`GET /treasury/onramp/:onRampId`](#get-treasuryonramponrampid).

PIX bodies are strict: an unknown field is a `400`.

## USDC deposits

To deposit USDC itself, send it on chain to your deposit address.

```bash
curl "$JET_API/treasury/crypto-deposit" -H "Authorization: Bearer $JET_API_KEY"
```

```json
{
  "currency": "USDC",
  "address": "0x…",
  "networks": [
    { "network": "polygon", "address": "0x…", "home": true },
    {
      "network": "arbitrum",
      "address": "0x…",
      "home": false,
      "fee": { "amount": "120000", "currency": "USDC", "scale": 6 }
    }
  ],
  "quotedAt": { "amount": "100000000", "currency": "USDC", "scale": 6 }
}
```

- **Send on your home network** (`home: true`) and the USDC is credited as it is.
- **Send on any other listed network** and it is credited, then moved to your home
  network. The `fee` for that move is deducted, and until it arrives it shows as
  `settling` in [`GET /balance`](wallets-and-accounts.md#get-balance).
- **`fee` is a live estimate**, priced for a deposit of `quotedAt`. Where it
  cannot be priced, `feeUnavailable` says why.
- **`address`** at the top level is present when every network shares one address.

!!! danger

    Send only USDC, only on a listed network. Anything else sent to the address
    may not be recoverable.

`422 no_home_wallet` means your account has no wallet to receive deposits yet;
contact us.

## In a sandbox

No real wire or PIX payment is sent in a sandbox. Make one "arrive" with the
[sandbox tools](../getting-started/environments.md#sandbox-tools).
