# Treasury

Two operations that move value without being payments: getting fiat **in** as a
stablecoin, and converting a stablecoin holding back to fiat at par.

Neither has a corridor or a recipient. Both are durable operations that return
`202` and are polled for an outcome, exactly like a payment.

## POST /treasury/onramp

Converts fiat you wire in into a stablecoin, delivered to a wallet address you
control. This is how you pre-fund the wallet that later funds payments.

Requires an approved KYB status — an on-ramp collects money from you, which is
the direction with the sharpest AML exposure.

=== "Request"

    ```bash
    curl -X POST "$GENIFI_API/treasury/onramp" \
      -H "Authorization: Bearer $GENIFI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "onramp-2026-08-21-a",
        "amount": { "amount": "50000.00", "currency": "EUR" },
        "targetStablecoin": "USDC",
        "destinationAddress": "0x…"
      }'
    ```

=== "202 Accepted"

    ```json
    {
      "onRampId": "onramp-2026-08-21-a",
      "status": "accepted",
      "instructions": {
        "beneficiary": "…",
        "iban": "…",
        "reference": "onramp-2026-08-21-a"
      }
    }
    ```

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `idempotencyKey` | string | yes | 8–200 chars of `[A-Za-z0-9_.:-]`. **Becomes the on-ramp ID** |
| `amount.amount` | decimal string | yes | The fiat you will wire |
| `amount.currency` | string | yes | e.g. `"EUR"` |
| `targetStablecoin` | `"USDC"` | no | Defaults to `"USDC"` |
| `destinationAddress` | string | yes | The wallet address the stablecoin credits |

### Bank instructions

When the environment can produce them, the `202` includes an `instructions`
object with where to send the wire and — critically — the **reference** to quote.

Quote it exactly. The reference is how an inbound wire is matched to the on-ramp
you started. A wire without it, or with a mangled one, arrives unattributed and
needs manual reconciliation.

`instructions` is absent when the environment cannot produce them yet. The
on-ramp still exists and is still polled the same way.

## GET /treasury/onramp/:onRampId

=== "Request"

    ```bash
    curl "$GENIFI_API/treasury/onramp/onramp-2026-08-21-a" \
      -H "Authorization: Bearer $GENIFI_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "onRampId": "onramp-2026-08-21-a",
      "running": false,
      "state": "DELIVERED",
      "net": { "amount": "54120000000", "currency": "USDC", "scale": 6 },
      "fee": { "amount": "45000", "currency": "EUR", "scale": 2 },
      "timeline": [ … ]
    }
    ```

### On-ramp states

| State | Meaning |
|---|---|
| `DELIVERED` | The stablecoin is in your wallet. Done |
| `REFUNDED` | The on-ramp unwound and your fiat was returned |
| `FAILED` | It stopped before delivery |
| `EXPIRED` | The wire never arrived within the window |
| `IN_DOUBT` | We cannot yet tell whether funds moved. Parked for reconciliation |

`net` is what was delivered; `fee` is what was retained. Both appear once the
conversion has run.

`EXPIRED` is the common one in practice and it is usually benign: the on-ramp was
started but the wire was never sent, or was sent too late. Start a new one with a
new key.

## POST /issuers/:issuerId/redeem

Converts a stablecoin holding back into fiat **at par** — 1:1 face value through
the issuer, rather than swapping on a market and paying slippage.

Requires an approved KYB status.

=== "Request"

    ```bash
    curl -X POST "$GENIFI_API/issuers/iss_a1b2/redeem" \
      -H "Authorization: Bearer $GENIFI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "redeem-2026-08-21-a",
        "amount": { "amount": "25000.00", "currency": "USDC" },
        "to": "USD"
      }'
    ```

=== "202 Accepted"

    ```json
    { "redeemId": "redeem-2026-08-21-a", "status": "accepted" }
    ```

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `idempotencyKey` | string | yes | **Becomes the redemption ID** |
| `amount.amount` | decimal string | yes | Over-precision is rejected, never rounded |
| `amount.currency` | string | yes | e.g. `"USDC"` |
| `to` | string | no | The fiat to settle into. Defaults to `"USD"` |

!!! info

    Note what this body does **not** contain: no source wallet, no destination
    account. A redemption converts a held position into custody on both ends, and the
    accounts are fixed. Letting a caller name either end would turn a treasury
    operation into a withdrawal.

## GET /issuers/redemptions/:redeemId

=== "Request"

    ```bash
    curl "$GENIFI_API/issuers/redemptions/redeem-2026-08-21-a" \
      -H "Authorization: Bearer $GENIFI_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "redeemId": "redeem-2026-08-21-a",
      "running": false,
      "state": "SETTLED",
      "timeline": [ … ]
    }
    ```

### Redemption states

| State | Meaning |
|---|---|
| `INITIATED` | Recorded; nothing submitted yet |
| `SUBMITTED` | The redemption is with the issuer |
| `SETTLING` | Awaiting settlement into custody |
| `SETTLED` | Done |
| `FAILED` | Stopped, cleanly |
| `IN_DOUBT` | Ambiguous outcome, parked for reconciliation |

The path deliberately omits the issuer. A redemption ID is unique on its own, and
requiring the issuer to read one back would let you ask the wrong issuer about a
real redemption and get a `404` that means "wrong path" rather than "no such
redemption".

## Availability

Both operations depend on adapters that are not wired on every environment. When
they are not, the endpoint returns `501` with a reason:

```json
{
  "error": "not_implemented",
  "feature": "onramp",
  "message": "no on-ramp provider is configured on this environment",
  "requires": "an on-ramp provider adapter"
}
```

Check [`/capabilities`](discovery.md#get-capabilities) first.
