# Batch payouts

A batch pays many recipients in one instruction: up to 1,000 payments, to one
corridor, on one rail, funded from your home wallet in a **single** transfer.
Use it for payroll, supplier runs, or any file of payouts you would otherwise
send one by one.

Every field is in the [API reference](../api-reference/batches.md).

## How a batch behaves

**Accepted or refused as a whole.** Either the whole batch is refused and nothing
starts, or every item is started. There is no partial acceptance.

**Paid item by item.** Once accepted, each item is an ordinary
[payment](transactions.md) with its own ID, `<batchId>:<itemRef>`, its own state
and its own webhooks. Items succeed or fail independently: a payout that has
settled cannot be undone because another one failed, so 1,000 items with three
failures is 997 paid and three waiting for attention.

**Funded once.** The batch moves the exact sum of its items from your home wallet
in one transfer, and each item draws exactly its own amount from it.

**One rail, which you name.** A single payment is routed to the best rail
automatically. A batch is funded at one provider, so you choose it in `provider`.
Use the `provider` that a [quote](quotes.md) for the corridor returns.

## POST /batches

Requires the `payment.initiate` permission and an approved KYB status, exactly like a
single payment.

=== "Request"

    ```bash
    curl -X POST "$JET_API/batches" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "batchId": "payroll-2026-10-15",
        "corridor": { "to": "VND", "destinationCountry": "VN" },
        "provider": "tazapay",
        "items": [
          {
            "itemRef": "emp-0001",
            "sendAmount": { "amount": "450.00", "currency": "USDC" },
            "recipientReference": "employee-0001",
            "beneficiary": {
              "name": "NGUYEN VAN A",
              "account": "0123456789",
              "bankName": "Vietcombank",
              "bic": "BFTVVNVX",
              "country": "VN"
            }
          },
          {
            "itemRef": "emp-0002",
            "sendAmount": { "amount": "380.00", "currency": "USDC" },
            "recipientReference": "employee-0002",
            "recipientId": "rcp_7f21…"
          }
        ]
      }'
    ```

=== "202 Accepted"

    ```json
    {
      "batchId": "payroll-2026-10-15",
      "state": "accepted",
      "itemCount": 2,
      "note": "every item was accepted; each is paid independently and can fail on its own"
    }
    ```

=== "400 Duplicate itemRef"

    ```json
    {
      "error": "duplicate_item_ref",
      "message": "every itemRef must be unique within the batch",
      "itemRefs": ["emp-0002"]
    }
    ```

- **`batchId` is your idempotency key** and becomes the batch ID. Resubmitting
  the same batch is safe: nothing is paid twice. A `batchId` already used by
  another organization is `409 idempotency_key_conflict`.
- **`itemRef`** must be unique within the batch, 1 to 64 characters of
  `A-Z a-z 0-9 _ . : -`.
- **Each item names its recipient** with either an inline `beneficiary` or the
  `recipientId` of a [saved recipient](recipients.md). Saved recipients are
  looked up on our side, so a large batch does not have to repeat bank details.
- **One currency.** Every item's `sendAmount.currency` must be the same.
- **`originator`**, if you send one, is the sender of the whole batch.

### What is refused immediately

Problems with the request itself are refused before anything starts. Where items
are at fault, `itemRefs` names them. Fix the request and resubmit with the same
`batchId`:

| Status | `error` | Cause |
|---|---|---|
| `403` | `kyb_required` | Your organization is not verified yet |
| `400` | `validation` | The body does not parse, or has more than 1,000 items |
| `400` | `duplicate_item_ref` | Two items share an `itemRef` |
| `400` | `invalid_send_amount` | An amount is malformed, over-precise, or in a different currency from the rest |
| `400` | `unknown_recipient` | A `recipientId` names no recipient saved to your organization |
| `409` | `idempotency_key_conflict` | The `batchId` belongs to another batch |
| `422` | `not_home_wallet`, `no_home_wallet` | The funding wallet is not, or you have no, home wallet |

### What is refused after acceptance

A `202` means the batch was accepted for checking, not that it will be paid.
Sanctions screening and per-corridor checks (such as a beneficiary missing a field
the corridor needs) run next. If any item fails them, **the whole batch** is
rejected with nothing funded and nothing started: its state becomes `REJECTED`
with a `rejectedReason`, and you receive a `batch.rejected` webhook.

Screening comes first on purpose: a sanctions finding is recorded even when the
same batch has a simpler problem too.

## Rate limits

A batch draws from the `initiate` bucket **once per item**: a 200-item batch
costs 200 tokens, the same as 200 single payments. Size your batches so they fit
your burst allowance. See [Rate limits](../reference/rate-limits.md).

## GET /batches/:batchId

=== "Request"

    ```bash
    curl "$JET_API/batches/payroll-2026-10-15?limit=100" \
      -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "batchId": "payroll-2026-10-15",
      "state": "DISPATCHED",
      "corridor": { "from": "USDC", "to": "VND", "destinationCountry": "VN" },
      "provider": "tazapay",
      "chain": "polygon",
      "itemCount": 2,
      "totalSendAmount": { "amount": "830000000", "currency": "USDC" },
      "items": { "RECONCILED": 1, "SETTLING": 1 },
      "createdAt": "2026-10-15T08:00:00.000Z",
      "updatedAt": "2026-10-15T08:04:12.000Z",
      "page": {
        "items": [
          {
            "itemRef": "emp-0001",
            "transactionId": "payroll-2026-10-15:emp-0001",
            "state": "RECONCILED",
            "sendAmount": { "amount": "450000000", "currency": "USDC" },
            "recipientReference": "employee-0001"
          },
          {
            "itemRef": "emp-0002",
            "transactionId": "payroll-2026-10-15:emp-0002",
            "state": "SETTLING",
            "sendAmount": { "amount": "380000000", "currency": "USDC" },
            "recipientReference": "employee-0002"
          }
        ],
        "nextCursor": null,
        "hasMore": false
      }
    }
    ```

- **`items`** counts every item by state, in one read, however large the batch.
- **`page`** lists the items themselves, paginated with `limit` and `cursor`
  like any list. A failed item carries `failureReason`.
- **An item's `state`** is the state of its payment (see
  [How a payment works](../concepts/payment-lifecycle.md)), starting at
  `ACCEPTED`. For the full detail of one item, read
  `GET /transactions/<batchId>:<itemRef>`.

Amounts here are integer minor units of the batch's currency.

### Batch states

| `state` | Meaning |
|---|---|
| `ACCEPTED` | Recorded; checks have not run yet |
| `SCREENING` | Sanctions and corridor checks are running |
| `REJECTED` | Refused as a whole. Nothing was funded and no item started. See `rejectedReason` |
| `DISPATCHED` | Funded, and every item has been started |
| `IN_DOUBT` | The funding transfer could not be confirmed. No item was started; it is being reconciled |
| `COMPLETED` | No item is still running. **Not** the same as every item paid: read `items` |

`COMPLETED` arrives with a `batch.completed` [webhook](../webhooks/events.md#batches)
carrying the same counts. An item waiting for our operations team
(`NEEDS_ATTENTION`) counts as finished for this purpose, so a batch does not stay
open for as long as the slowest manual review takes.

## GET /batches

Your batches, newest first, cursor-paginated. Each row carries the batch's
`state`, corridor, provider, item count and total.
