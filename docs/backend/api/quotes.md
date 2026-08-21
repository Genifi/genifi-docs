# Quotes

A quote is a **firm, rate-locked price** you can show a customer before
committing. It starts nothing and moves nothing.

## POST /quotes

Requires the `payment.initiate` permission — a `viewer` cannot fish for prices.

=== "Request"

    ```bash
    curl -X POST "$GENIFI_API/quotes" \
      -H "Authorization: Bearer $GENIFI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "corridor": { "to": "VND", "destinationCountry": "VN" },
        "sendAmount": { "amount": "1000.00", "currency": "USDC" },
        "idempotencyKey": "pay-2026-08-21-0001"
      }'
    ```

=== "200 OK"

    ```json
    {
      "quoteId": "qte_8b3d…",
      "sendAmount": { "amount": "1000000000", "currency": "USDC", "scale": 6 },
      "receiveAmount": { "amount": "24512750", "currency": "VND", "scale": 0 },
      "fee": { "amount": "4200000", "currency": "USDC", "scale": 6 },
      "rate": { "pair": "USDC/VND", "value": "24512.75" },
      "provider": "owned-vn",
      "routeKind": "owned",
      "estimatedSeconds": 45,
      "expiresAt": "2026-08-21T14:03:11.000Z"
    }
    ```

=== "422 Unavailable"

    ```json
    {
      "error": "quote_unavailable",
      "message": "Could not produce a quote for this corridor right now. Please try again shortly."
    }
    ```

### Request body

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `corridor.to` | string | ✅ | Destination currency, e.g. `"VND"` |
| `corridor.destinationCountry` | string(2) | ✅ | ISO country code, e.g. `"VN"` |
| `sendAmount.amount` | decimal string | ✅ | e.g. `"1000.00"`. Never a JSON number |
| `sendAmount.currency` | string | ✅ | e.g. `"USDC"` |
| `idempotencyKey` | string | — | The key of the payment you intend to make. See below |

### Response

| Field | Type | Notes |
|---|---|---|
| `quoteId` | string | Single-use. Pass to `POST /transactions` to execute at this price |
| `sendAmount` | Money | Echoed, in integer minor units |
| `receiveAmount` | Money | **What the recipient gets**, floored to whole destination units |
| `fee` | Money | The winning route's disclosed fee |
| `rate` | object | The effective end-to-end rate, send → receive |
| `provider` | string | Opaque route label, for support and reconciliation |
| `routeKind` | `"orchestrator"` \| `"owned"` | Whether Genifi runs this corridor end to end |
| `estimatedSeconds` | number | Expected time to payout |
| `expiresAt` | ISO 8601 | After this the quote can no longer be accepted |

Show `receiveAmount` to your customer. It is the number they care about, and it
is the number that will land.

## The lock

`expiresAt` is roughly a minute out, bounded by the underlying provider's own
quote. It is a real lock, not a hint: accept before it, and the payment executes
at exactly this price.

Accept after it and the payment fails cleanly, **before any money moves**. That
is the intended behaviour — re-quote and show the customer the new number rather
than silently executing at a price they never saw.

## Accepting a quote

Pass `quoteId` on the payment:

```json
{
  "idempotencyKey": "pay-2026-08-21-0001",
  "quoteId": "qte_8b3d…",
  "corridor": { "to": "VND", "destinationCountry": "VN" },
  "sendAmount": { "amount": "1000.00", "currency": "USDC" },
  "integratorWalletId": "wal_your_wallet",
  "recipientReference": "customer-4471"
}
```

The corridor and send amount on the payment **must match the quote**. We verify
it at execution: a quote ID is unguessable, but binding it to the request is the
defence that stops a mismatched quote being applied to a different payment.

If you supplied an `idempotencyKey` when quoting, the same key is required when
accepting.

## Quoting is optional

`POST /transactions` without a `quoteId` prices the payment itself, at execution
time. That is simpler and perfectly safe.

Quote first when you need to **show a price before charging**, which is most
consumer-facing flows. Skip it for back-office transfers where nobody is looking
at a number.

## Errors

| Code | Meaning |
|---|---|
| `403 forbidden` | Your role may not request quotes |
| `422 quote_unavailable` | The corridor could not be priced right now. Retry shortly |
| `400 validation` | Malformed body, or an over-precise amount |

A `422` is client-actionable and carries no internal detail. It usually means a
pricing source is momentarily unavailable, not that the corridor is closed —
check `/capabilities` if you need to tell the two apart.
