# Payments

The core of the API: start a payment, list your payments, read one's status.

## POST /transactions

Starts a payment. Returns `202` immediately; the payment runs to completion on
our side.

Requires the `payment.initiate` permission and an approved KYB status.

=== "Request"

    ```bash
    curl -X POST "$GENIFI_API/transactions" \
      -H "Authorization: Bearer $GENIFI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "idempotencyKey": "pay-2026-08-21-0001",
        "corridor": { "to": "VND", "destinationCountry": "VN" },
        "sendAmount": { "amount": "1000.00", "currency": "USDC" },
        "integratorWalletId": "wal_your_wallet",
        "recipientReference": "customer-4471",
        "payoutMode": "regular",
        "beneficiary": {
          "name": "NGUYEN VAN A",
          "account": "0123456789",
          "bankName": "Vietcombank",
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

=== "403 Wrong wallet"

    ```json
    {
      "error": "forbidden",
      "message": "integratorWalletId is not a wallet registered to this integrator",
      "integratorWalletId": "wal_not_yours"
    }
    ```

### Request body

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `idempotencyKey` | string | yes | 8–200 chars of `[A-Za-z0-9_.:-]`. **Becomes the transaction ID** |
| `corridor.to` | string | yes | Destination currency |
| `corridor.destinationCountry` | string(2) | yes | ISO country code |
| `sendAmount.amount` | decimal string | yes | e.g. `"1000.00"` |
| `sendAmount.currency` | string | yes | e.g. `"USDC"` |
| `integratorWalletId` | string | yes | The `externalId` of a wallet **you** own |
| `recipientReference` | string | yes | Your own reference for the recipient |
| `payoutMode` | `"regular"` or `"instant"` | no | Defaults to `"regular"` |
| `quoteId` | string | no | Execute at a previously locked [quote](quotes.md) |
| `beneficiary` | object | no | Where the money is actually paid. See below |
| `originator` | object | no | Who is sending, for Travel Rule. See below |

!!! info

    There is no `integratorId` field. Your organization comes from the API key,
    always. A client must not be able to claim to be someone else by editing a body.

### beneficiary — where the money lands

The structured payout destination, passed through to the local payout partner.

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `name` | string(1–200) | yes | The recipient's name as the destination bank holds it |
| `account` | string(1–140) | yes | The real destination account |
| `bankCode` | string(1–35) | no | Local routing code |
| `bic` | string(8–11) | no | SWIFT/BIC |
| `bankName` | string(1–200) | no | |
| `country` | string(2) | no | ISO country code |
| `city` | string(1–100) | no | |
| `addressLine` | string(1–280) | no | |

Which optional fields are needed depends on the corridor — a Vietnamese bank
transfer and a Nigerian one do not want the same routing detail.

`beneficiary` is optional in the schema, but a payment without one has no
destination account to pay. Send it.

### originator — Travel Rule

Your end user's identity, forwarded for FATF Travel Rule compliance. Every field
is optional: send as much of your own KYC as you hold.

| Field | Type | Notes |
|---|---|---|
| `name` | string(1–200) | |
| `account` | string(1–140) | A **reference** — masked or last-four, never a full number |
| `address` | string(1–280) | |
| `country` | string(2) | ISO country code |
| `bic` | string(8–11) | |
| `institution` | string(1–200) | |
| `idNumber` | string(1–80) | A reference, not a raw document number |
| `dateOfBirth` | `YYYY-MM-DD` | |

!!! warning

    Never put a full card number or an unmasked account identifier in `originator`.
    The field is a reference by contract. The beneficiary party recorded in our
    compliance log is derived from `beneficiary` with its account **masked**
    automatically — the full account is routing detail for the payout partner, not
    something the audit log keeps in the clear.

### Sanctions screening

Genifi screens the Travel Rule parties and the recipient reference before any
value moves. A hit refuses the payment and **the workflow is never started**:

```json
{
  "error": "compliance_hold",
  "message": "transaction blocked pending compliance review",
  "transactionId": "pay-2026-08-21-0001"
}
```

The response deliberately does not say which list or which name matched. That is
the screened party's personal data, and a probing caller must not be able to use
Genifi as a sanctions oracle. Route these to your compliance team, not to an
automatic retry.

### Errors

| Code | Error | Meaning |
|---|---|---|
| `400` | `validation` | Malformed body, over-precise amount, bad idempotency key format |
| `401` | `unauthorized` | Missing or invalid key |
| `403` | `forbidden` | Your role may not initiate payments |
| `403` | `forbidden` | `integratorWalletId` is not a wallet you own |
| `403` | `kyb_required` | Your organization is not yet verified |
| `403` | `compliance_hold` | Sanctions screening blocked the payment |
| `429` | `rate_limited` | See [Rate limits](../reference/rate-limits.md) |
| `503` | *varies* | A dependency is unavailable. Retry with the same key |

## GET /transactions/:id

Live status of one payment.

=== "Request"

    ```bash
    curl "$GENIFI_API/transactions/pay-2026-08-21-0001" \
      -H "Authorization: Bearer $GENIFI_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "transactionId": "pay-2026-08-21-0001",
      "running": false,
      "state": "RECONCILED",
      "provider": "owned-vn",
      "payoutMode": "instant",
      "receiveAmount": { "amount": "24512750", "currency": "VND" },
      "margin": { "amountMinorUnits": "1840000", "bps": 42, "realized": true },
      "timeline": [
        { "state": "INITIATED" },
        { "state": "QUOTED", "detail": "locked qte_8b3d" },
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
    { "error": "not_found", "transactionId": "pay-2026-08-21-0001" }
    ```

| Field | Type | Notes |
|---|---|---|
| `transactionId` | string | The key you chose |
| `running` | boolean | `true` while the payment is still in flight |
| `state` | string | See [How a payment works](../concepts/payment-lifecycle.md) |
| `provider` | string | Opaque route label |
| `payoutMode` | `"regular"` or `"instant"` | **What actually happened**, not what you asked for |
| `receiveAmount` | Money | What the recipient received, once known |
| `payoutDelta` | object or absent | Present **only** if paid differs from quoted. Absence means they agreed exactly |
| `margin` | object or absent | Rate economics, where both route kinds competed |
| `timeline` | array | Ordered state history with optional human-readable detail |
| `failure` | object or absent | Set when the payment ended without a result |

!!! warning

    A `404` here means *no such payment, **or** not yours* — the two are
    indistinguishable by design. See
    [Conventions](conventions.md#404-also-means-not-yours).

### Polling

Poll every couple of seconds while `running` is `true`; stop as soon as the state
is terminal (`RECONCILED`, `REFUNDED`, `FAILED`, or `IN_DOUBT`).

Better: [register a webhook](../webhooks/README.md) and stop polling entirely.

## GET /transactions

Your organization's payment history.

=== "Request"

    ```bash
    curl "$GENIFI_API/transactions?limit=50" \
      -H "Authorization: Bearer $GENIFI_API_KEY"
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

| Query | Type | Notes |
|---|---|---|
| `limit` | integer | Positive, maximum 500. Rows per page |
| `cursor` | string | `nextCursor` from the previous page. Omit for the first |

| Field | Notes |
|---|---|
| `count` | Rows on **this page**. Not a total |
| `nextCursor` | Cursor for the next page, or `null` at the end of the list |
| `hasMore` | Whether another page exists |

Page through your history by sending `nextCursor` back as `?cursor=` until it
comes back `null`:

```bash
curl "$GENIFI_API/transactions?limit=50&cursor=$CURSOR" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

The cursor is opaque and keyset-based, so payments you initiate while paging
land above your current position and cannot make a page repeat or skip rows. See
[Pagination](conventions.md#pagination) for the full contract, including the
`400 invalid_cursor` response.

This is a record of what was **initiated** — never a balance and never a live
state. It exists so you can enumerate your payments; call
`GET /transactions/:id` per row for current status.
