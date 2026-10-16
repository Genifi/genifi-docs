# Payments

The core of the API: start a payment, list your payments, read one's status.
Every field and error code is in the [API reference](../api-reference/payments.md);
this page explains how to use them.

## POST /transactions

Starts a payment. Returns `202` immediately; the payment runs to completion on
our side.

Requires the `payment.initiate` permission and an approved KYB status.

=== "Request"

    ```bash
    curl -X POST "$JET_API/transactions" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "pay-2026-08-21-0001",
        "quoteId": "q_3d6e9932-61c7-49cb-9259-9703dabf3cdc",
        "corridor": { "to": "VND", "destinationCountry": "VN" },
        "sendAmount": { "amount": "1000.00", "currency": "USDC" },
        "recipientReference": "customer-4471",
        "beneficiary": {
          "name": "NGUYEN VAN A",
          "account": "0123456789",
          "bankName": "Vietcombank",
          "bic": "BFTVVNVX",
          "country": "VN"
        },
        "originator": {
          "name": "Jan Novak",
          "country": "CZ",
          "dateOfBirth": "1988-04-12"
        }
      }'
    ```

=== "202 Accepted"

    ```json
    { "transactionId": "pay-2026-08-21-0001", "status": "accepted" }
    ```

=== "400 Beneficiary incomplete"

    ```json
    {
      "error": "beneficiary_incomplete",
      "message": "…",
      "corridor": "USDC->IDR@ID",
      "missing": ["phone"],
      "byProvider": [ … ]
    }
    ```

=== "422 Insufficient funds"

    ```json
    {
      "error": "insufficient_funds",
      "message": "…",
      "available": { "amount": "250000000", "currency": "USDC", "scale": 6 }
    }
    ```

What to know about the body:

- **`idempotencyKey` becomes the transaction ID.** Retry the same request as often
  as you like; there is one payment. See [Idempotency](../concepts/idempotency.md).
- **You do not name a wallet.** A payment is funded from your **home wallet**,
  where your whole USDC balance lives (see [Balance and wallets](wallets-and-accounts.md)).
  `integratorWalletId` is optional; if you send it, it must be your home wallet,
  or the payment is refused with `422 not_home_wallet`.
- **`quoteId` executes at a price you already showed.** Optional. Without it the
  payment is priced when it runs. See [Quotes](quotes.md).
- **`beneficiary` is where the money lands.** Which fields a corridor needs varies:
  a Vietnamese bank transfer and an Indonesian one do not want the same routing
  detail, and some corridors need a postal address (`addressLine`, `city`,
  `state`, `postalCode`) or a `phone`. A missing field is a `400
  beneficiary_incomplete` that names it in `missing`, before anything moves.
  The schema calls `beneficiary` optional, but a payment without one has nowhere to
  pay. Send it.
- **`originator` is the sender, for the Travel Rule.** Every field is optional:
  send as much of your own KYC as you hold.
- **`payoutMode`** is accepted for older integrations and always runs as
  `regular`. See [How a payment works](../concepts/payment-lifecycle.md#payout-mode).

!!! info

    There is no `integratorId` field. Your organization comes from the API key,
    always. A client must not be able to claim to be someone else by editing a body.

!!! warning

    `originator.account` and `originator.idNumber` are **references**: masked or
    last-four, never a full card number or an unmasked identifier. The beneficiary
    party recorded in our compliance log is derived from `beneficiary` with its
    account **masked** automatically. The full account is routing detail for the
    payout partner, not something the audit log keeps in the clear.

### Sanctions screening

JET screens the Travel Rule parties and the recipient reference before any
value moves. A hit refuses the payment and **the payment is never started**:

```json
{
  "error": "compliance_hold",
  "message": "transaction blocked pending compliance review",
  "transactionId": "pay-2026-08-21-0001"
}
```

The response deliberately does not say which list or which name matched. That is
the screened party's personal data, and a probing caller must not be able to use
JET as a sanctions oracle. Route these to your compliance team, not to an
automatic retry.

### Errors worth handling

| Status | `error` | What to do |
|---|---|---|
| `400` | `beneficiary_incomplete` | Add the fields named in `missing`, then resend with the same key |
| `403` | `kyb_required` | Your organization is not verified yet. Nothing to retry |
| `403` | `compliance_hold` | Route to your compliance team. Do not retry |
| `422` | `insufficient_funds` | Your home wallet holds less than the payment. `available` says how much it holds |
| `422` | `quote_expired`, `quote_not_found`, `quote_mismatch` | Request a new quote; for `quote_mismatch`, send the corridor and amount you quoted |
| `422` | `not_home_wallet` | Omit `integratorWalletId` |

The full list per status is in the
[API reference](../api-reference/payments.md#post-transactions).

## GET /transactions/:id

Live status of one payment.

=== "Request"

    ```bash
    curl "$JET_API/transactions/pay-2026-08-21-0001" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "transactionId": "pay-2026-08-21-0001",
      "running": false,
      "state": "RECONCILED",
      "provider": "tazapay",
      "payoutMode": "regular",
      "receiveAmount": { "amount": "24512750", "currency": "VND", "scale": 0 },
      "timeline": [
        { "state": "INITIATED" },
        { "state": "QUOTED", "detail": "locked q_3d6e9932…" },
        { "state": "FUNDED" },
        { "state": "ROUTING" },
        { "state": "SETTLING" },
        { "state": "PAID_OUT" },
        { "state": "RECONCILED" }
      ]
    }
    ```

=== "404 Not found"

    ```json
    {
      "error": "not_found",
      "message": "…",
      "transactionId": "pay-2026-08-21-0001"
    }
    ```

| Field | Notes |
|---|---|
| `running` | `true` while the payment is still in flight |
| `state` | See [How a payment works](../concepts/payment-lifecycle.md) |
| `provider` | Opaque route label, for support and reconciliation |
| `receiveAmount` | What the recipient received, once known |
| `payoutDelta` | Present **only** if what was paid differs from what was quoted. Absence means they agreed exactly |
| `timeline` | Ordered state history. Treat `detail` as text for people, not a field to parse |
| `failure` | Set when the payment ended without a result |

!!! warning

    A `404` here means *no such payment, **or** not yours*. The two are
    indistinguishable by design. See
    [Conventions](conventions.md#404-also-means-not-yours).

### Polling

Poll every couple of seconds while `running` is `true`. Stop when the state is
terminal (`RECONCILED`, `REFUNDED`, `FAILED`) or needs a person (`IN_DOUBT`,
`NEEDS_ATTENTION`).

Better: [register a webhook](../webhooks/README.md) and stop polling entirely.

## GET /transactions

Your organization's payment history, newest first.

=== "Request"

    ```bash
    curl "$JET_API/transactions?limit=50" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "transactions": [
        {
          "transactionId": "pay-2026-08-21-0001",
          "corridor": { "from": "USDC", "to": "VND", "destinationCountry": "VN" },
          "sendAmount": { "amount": "1000000000", "currency": "USDC", "scale": 6 },
          "recipientReference": "customer-4471",
          "createdAt": "2026-08-21T14:02:07.000Z"
        }
      ],
      "count": 1,
      "nextCursor": "eyJ0IjoiMjAyNi0wOC0yMVQxNDowMjowNy4wMDBaIn0",
      "hasMore": true
    }
    ```

Page through by sending `nextCursor` back as `?cursor=` until it comes back
`null`. See [Pagination](conventions.md#pagination).

This is a record of what was **initiated**. Rows carry no state: call
`GET /transactions/:id` per row for that, or keep the state your webhooks told
you.
