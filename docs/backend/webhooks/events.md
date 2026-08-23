# Event types

Six events, one per transaction transition worth telling you about.

| Event | Fires when | Typical reaction |
|---|---|---|
| `transaction.quoted` | A price has been locked for the payment | Record the rate |
| `transaction.funded` | Funds have left your wallet and are held by Genifi | Mark the customer's funds as committed |
| `transaction.paid_out` | The local partner confirms the recipient has been paid | Tell your customer the money has landed |
| `transaction.reconciled` | Books, chain, and provider all agree — the payment is **done** | Settle your own accounting |
| `transaction.refunded` | An unwind completed and the money came back | Return the funds to your customer |
| `transaction.in_doubt` | The outcome is ambiguous and has been parked | Alert operations. **Do not** retry the payment |

## Envelope

Every delivery has the same shape:

```json
{
  "deliveryId": "dlv_6ac1…",
  "event": "transaction.paid_out",
  "transactionId": "pay-2026-08-21-0001",
  "data": { "…": "…" }
}
```

| Field | Type | Notes |
|---|---|---|
| `deliveryId` | string | Stable across retries. **De-duplicate on this** |
| `event` | string | One of the six above |
| `transactionId` | string | The idempotency key you chose |
| `data` | object | Event-specific payload |

`data` carries the state and any amounts known at that point:

```json
{
  "state": "PAID_OUT",
  "receiveAmount": { "amount": "24512750", "currency": "VND" }
}
```

Amounts are integer minor units. See
[Amounts and currencies](../concepts/money.md).

## Which events actually matter

For most integrations, three:

**`transaction.paid_out`** — the recipient has the money. This is the moment your
customer wants to hear about.

**`transaction.reconciled`** — the payment is genuinely finished and the final
amounts are settled. This is the moment your *ledger* should hear about.

**`transaction.in_doubt`** — you need a human.

`quoted` and `funded` are useful for a detailed progress UI and noise otherwise.

## Do not settle on `paid_out`

The recipient has been paid, but the books have not yet been proven to agree with
the provider and the chain. If the confirmed amount turns out to differ from the
quoted one, that difference surfaces during reconciliation.

Show `paid_out` to your customer; settle your own accounting on `reconciled`.

## `in_doubt` is not a failure

It means the honest answer is "we don't know yet" — a provider call timed out, or
a response was ambiguous, and we would rather park the payment than guess.

Genifi re-queries the provider on a schedule and the payment resumes on its own.
It will reach a terminal state without you doing anything.

So: alert your operations team, and **do not** start a replacement payment. A
replacement is how an ambiguous payment becomes a confirmed double-payment.

## Ordering

Events are not guaranteed to arrive in order. Two deliveries for the same payment
can cross.

Guard your state machine accordingly: keep the state you have already recorded,
and ignore an event that would move a payment backwards. If you need certainty at
any moment, read [`GET /transactions/:id`](../api/transactions.md#get-transactionsid) —
that is always authoritative.

## Notification emails

The same six events back per-person email notifications in the dashboard. Those
are for humans; webhooks are for systems. See
[Notifications](../api/notifications.md).
