---
description: What happens after your POST returns, and how to read the states.
---

# How a payment works

`POST /transactions` returns `202 Accepted` in a few hundred milliseconds. That
response does not mean the payment finished: it means the payment **started**,
as a durable process that will run to completion on our side without further
input from you.

Understanding the states below is the difference between an integration that
reports payment status correctly and one that tells your customer their money
arrived when it hasn't.

## The states

```
INITIATED → QUOTED → FUNDED → ROUTING → SETTLING → PAID_OUT → RECONCILED
```

| State | What has happened |
|---|---|
| `INITIATED` | Your request was validated and recorded. Nothing has moved. |
| `QUOTED` | A price is locked, with a time-to-live. |
| `FUNDED` | Funds have left your wallet and are held by JET. |
| `ROUTING` | A route is chosen and the crossing has begun. |
| `SETTLING` | The payout instruction is with the local partner; we are waiting for confirmation. |
| `PAID_OUT` | The local partner confirms the recipient has been paid. |
| `RECONCILED` | Our ledger, the chain, and the provider's records all agree. **The payment is now done.** |

### When it does not go straight through

| State | What has happened |
|---|---|
| `FAILED` | The payment stopped **before** any of your funds moved. Nothing to undo. |
| `PENDING_RETRY` | Your funds have moved, but the payout could not be made yet. It is retried on a backoff, automatically. |
| `NEEDS_ATTENTION` | Retries are exhausted and the payment is waiting for our operations team. The funds are held and accounted for: not paid out, and not returned. |
| `REFUNDING` | The payment was cancelled; the full amount is on its way back to your home wallet. |
| `REFUNDED` | The full amount is back in your home wallet. |
| `IN_DOUBT` | We cannot currently tell whether funds moved. Parked for reconciliation. |

## Two states people get wrong

**`PAID_OUT` is not the end.** The recipient has the money, but the books have
not yet been proven to agree with the provider and the chain. If you settle your
own internal accounting on `PAID_OUT`, you will occasionally book a payment whose
final amount differs from the one you recorded. Wait for `RECONCILED`.

**`IN_DOUBT` is not a failure.** It means the honest answer is "we don't know
yet". A payment reaches it when a provider call times out, or a response is
ambiguous, and we would rather park it than guess. JET re-queries the provider
on a schedule and the payment resumes on its own: it will end up in a terminal
state without you doing anything. Show it to your operations team; do not retry
the payment, and do not tell your customer it failed.

!!! info

    This is the "fail closed" rule. On any ambiguity about whether money moved, we
    treat the payment as in doubt and reconcile, rather than assuming success. A
    system that guesses is a system that eventually double-pays.

## A funded payment does not refund itself

Once your funds have moved, a payment that cannot be paid out is **not** sent
back automatically. It retries (`PENDING_RETRY`), and if retrying does not
succeed it waits for a person (`NEEDS_ATTENTION`). A payment that can still be
paid should be paid, and a refund issued too eagerly is a refund of money that
was about to arrive.

Our operations team then either retries the payout or cancels the payment. A
cancel always returns the **full** amount to your home wallet
(`REFUNDING → REFUNDED`); there are no partial refunds.

Every attempt that fails undoes its own bookkeeping immediately, so nothing is
half-unwound in between. If an unwind itself cannot complete cleanly, the
payment lands in `IN_DOUBT` for a human, never in a silent, ambiguous state.

## Payout mode

Every payment is paid the `regular` way: the recipient is paid once the
settling legs complete. `payoutMode: "instant"` is still accepted on
`POST /transactions`, so older integrations keep working, but it runs as
`regular`. Instant payout would need money pre-positioned in the destination
country, and JET holds none.

## Payout deltas

The status response may include a `payoutDelta`:

```json
"payoutDelta": { "…": "…" }
```

It is present **only** when the amount the provider confirms paying differed from
the amount the executing route quoted. Its absence means promised and paid agreed
exactly, so you can branch on whether the field exists, without parsing it.

## Reading the timeline

The status response carries a `timeline` array of the states the payment has
passed through, in order, with optional detail:

```json
"timeline": [
  { "state": "INITIATED" },
  { "state": "QUOTED", "detail": "locked q_3d6e9932…" },
  { "state": "FUNDED" },
  { "state": "ROUTING" },
  { "state": "SETTLING" },
  { "state": "PAID_OUT" },
  { "state": "RECONCILED" }
]
```

It is the right thing to render in a customer-facing payment tracker. Treat
`detail` as human-readable text, not as a parseable field.

## Where to get updates

* **[Webhooks](../webhooks/README.md)**: recommended. We push each transition to
  you, signed and retried.
* **[Polling](../api/transactions.md#get-transactionsid)**: fine for a first
  integration. Poll on a couple of seconds while `running` is `true`; stop once
  the state is terminal (`RECONCILED`, `REFUNDED`, `FAILED`) or needs a person
  (`IN_DOUBT`, `NEEDS_ATTENTION`).
