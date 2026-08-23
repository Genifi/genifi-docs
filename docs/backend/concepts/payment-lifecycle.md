---
description: What happens after your POST returns, and how to read the states.
---

# How a payment works

`POST /transactions` returns `202 Accepted` in a few hundred milliseconds. That
response does not mean the payment finished — it means the payment **started**,
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
| `FUNDED` | Funds have left your wallet and are held by Genifi. |
| `ROUTING` | A route is chosen and the crossing has begun. |
| `SETTLING` | The payout instruction is with the local partner; we are waiting for confirmation. |
| `PAID_OUT` | The local partner confirms the recipient has been paid. |
| `RECONCILED` | Our ledger, the chain, and the provider's records all agree. **The payment is now done.** |

### Failure branches

| State | What has happened |
|---|---|
| `FAILED` | The payment stopped before funds moved, or after a clean unwind. |
| `REFUNDING` | A step failed after funds moved; earlier steps are being reversed. |
| `REFUNDED` | The unwind completed. You have your money back. |
| `IN_DOUBT` | We cannot currently tell whether funds moved. Parked for reconciliation. |

## Two states people get wrong

**`PAID_OUT` is not the end.** The recipient has the money, but the books have
not yet been proven to agree with the provider and the chain. If you settle your
own internal accounting on `PAID_OUT`, you will occasionally book a payment whose
final amount differs from the one you recorded. Wait for `RECONCILED`.

**`IN_DOUBT` is not a failure.** It means the honest answer is "we don't know
yet". A payment reaches it when a provider call times out, or a response is
ambiguous, and we would rather park it than guess. Genifi re-queries the provider
on a schedule and the payment resumes on its own — it will end up in a terminal
state without you doing anything. Show it to your operations team; do not retry
the payment, and do not tell your customer it failed.

!!! info

    This is the "fail closed" rule. On any ambiguity about whether money moved, we
    treat the payment as in doubt and reconcile, rather than assuming success. A
    system that guesses is a system that eventually double-pays.

## Compensation

Every step that moves funds registers a way to reverse itself. If a later step
fails, the earlier ones unwind — which is what `REFUNDING → REFUNDED` is.

The guarantee is that a failed payment ends in a *marked* terminal state, never
half-unwound and silent. If the unwind itself cannot complete cleanly, the
payment lands in `IN_DOUBT` for a human, rather than being left ambiguous.

## Regular and instant payouts

`payoutMode` on `POST /transactions` chooses between two ways of paying the
recipient.

| Mode | Behaviour |
|---|---|
| `regular` (default) | The recipient is paid once the settling legs complete. Cheaper, slower. |
| `instant` | The recipient is paid immediately out of pre-positioned local liquidity, while the settling legs catch up behind. |

`instant` is a *request*, not a guarantee. It applies only where the corridor
supports it (check `instant` in [`/capabilities`](../getting-started/environments.md))
and only when there is enough local liquidity at that moment. If either condition
fails, the payment proceeds the regular way and nothing breaks.

Read back what actually happened from `payoutMode` in the status response — do
not assume the mode you asked for is the mode you got.

## Payout deltas

The status response may include a `payoutDelta`:

```json
"payoutDelta": { "…": "…" }
```

It is present **only** when the amount the provider confirms paying differed from
the amount the executing route quoted. Its absence means promised and paid agreed
exactly — so you can branch on whether the field exists, without parsing it.

## Reading the timeline

The status response carries a `timeline` array of the states the payment has
passed through, in order, with optional detail:

```json
"timeline": [
  { "state": "INITIATED" },
  { "state": "QUOTED", "detail": "locked qte_8b3d…" },
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

* **[Webhooks](../webhooks/README.md)** — recommended. We push each transition to
  you, signed and retried.
* **[Polling](../api/transactions.md#get-transactionsid)** — fine for a first
  integration. Poll on a couple of seconds while `running` is `true`; stop once
  the state is terminal.
