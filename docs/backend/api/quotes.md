# Quotes

A quote is a **firm, rate-locked price** you can show a customer before
committing. It starts nothing and moves nothing.

## POST /quotes

Requires the `payment.initiate` permission: a `viewer` cannot fish for prices.

=== "Request"

    ```bash
    curl -X POST "$JET_API/quotes" \
      -H "Authorization: Bearer $JET_API_KEY" \
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
      "quoteId": "q_3d6e9932-61c7-49cb-9259-9703dabf3cdc",
      "sendAmount": { "amount": "1000000000", "currency": "USDC", "scale": 6 },
      "receiveAmount": { "amount": "24904406", "currency": "VND", "scale": 0 },
      "fee": { "amount": "0", "currency": "VND", "scale": 0 },
      "rate": {
        "from": "USDC",
        "to": "VND",
        "numerator": "24929336132594337",
        "denominator": "1000000000000"
      },
      "provider": "tazapay",
      "routeKind": "orchestrator",
      "estimatedSeconds": 300,
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

Every field, with its rules, is in the
[API reference](../api-reference/payments.md#post-quotes). The ones that need
explaining:

| Field | Notes |
|---|---|
| `idempotencyKey` | Optional. The key of the payment you intend to make. See below |
| `integratorWalletId` | Optional. Quotes are priced for your home wallet; if you name a wallet, it must be that one |
| `receiveAmount` | **What the recipient gets**, floored to whole destination units |
| `fee` | The winning route's disclosed fee |
| `rate` | The exchange rate as an exact fraction: one whole unit of `from` buys `numerator / denominator` whole units of `to`. Never a float. Fees and rounding sit between it and `receiveAmount` |
| `networkFee` | Present when the payment has to cross to another network to reach the rail. Already deducted from what the recipient gets |
| `provider` | Opaque route label, for support and reconciliation |
| `expiresAt` | After this the quote can no longer be accepted |

Show `receiveAmount` to your customer. It is the number they care about, and it
is the number that will land.

## The lock

`expiresAt` is roughly a minute out, bounded by the underlying provider's own
quote. It is a real lock, not a hint: accept before it, and the payment executes
at exactly this price.

Accept after it and the payment fails cleanly, **before any money moves**. That
is the intended behaviour: re-quote and show the customer the new number rather
than silently executing at a price they never saw.

## Accepting a quote

Pass `quoteId` on the payment:

```json
{
  "idempotencyKey": "pay-2026-08-21-0001",
  "quoteId": "q_3d6e9932-61c7-49cb-9259-9703dabf3cdc",
  "corridor": { "to": "VND", "destinationCountry": "VN" },
  "sendAmount": { "amount": "1000.00", "currency": "USDC" },
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

| Status | `error` | Meaning |
|---|---|---|
| `400` | `validation` | Malformed body, or an over-precise amount |
| `403` | `forbidden` | Your role may not request quotes, or you named a wallet that is not yours |
| `422` | `quote_unavailable` | The corridor could not be priced right now. Retry shortly |
| `422` | `no_route_for_chain` | No rail can be funded from your balance's network. Retrying will not help; contact us |
| `422` | `not_home_wallet` | You named a wallet that is not your home wallet. Omit `integratorWalletId` |

`quote_unavailable` carries no internal detail. It usually means a pricing
source is momentarily unavailable, not that the corridor is closed: check
`/capabilities` if you need to tell the two apart.
