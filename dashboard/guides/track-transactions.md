# Track transactions

The **Transactions** page is every payment your organization has initiated, with
live status.

## What you see

One row per payment, showing the corridor, the amount sent, your recipient
reference, when it started, and its current state.

Payments you started in this browser appear immediately — before the backend
list has caught up — and are merged with the full history so you never see the
same payment twice.

Rows for payments still in flight refresh on their own every couple of seconds.
Once a payment reaches a terminal state, polling stops.

## The states

| State | What has happened |
|---|---|
| **Initiated** | Recorded and validated. Nothing has moved |
| **Quoted** | A price is locked |
| **Funded** | Funds have left your wallet and are held by Genifi |
| **Routing** | A route is chosen and the crossing has begun |
| **Settling** | The payout instruction is with the local partner |
| **Paid out** | The recipient has been paid |
| **Reconciled** | Everything agrees. **The payment is done** |

### And when things go differently

| State | What has happened |
|---|---|
| **Failed** | The payment stopped, cleanly |
| **Refunding** | A step failed after funds moved; earlier steps are reversing |
| **Refunded** | The unwind completed. You have your money back |
| **In doubt** | We cannot yet tell whether funds moved. Parked for reconciliation |

## Two states worth understanding

### Paid out is not the end

The recipient has the money, but the books have not yet been proven to agree with
the provider and the chain.

Tell your customer their money has landed at **paid out**. Close your own
accounting at **reconciled** — that is when the final amounts are settled.

### In doubt is not a failure

It means the honest answer is *we don't know yet*: a provider call timed out, or
a response was ambiguous, and Genifi parks the payment rather than guessing.

Genifi re-queries the provider on a schedule and the payment resumes on its own.
It will reach a terminal state without anyone doing anything.

{% hint style="danger" %}
**Do not send a replacement payment for an in-doubt one.** A replacement is how
an ambiguous payment becomes a confirmed double payment. Escalate to your
operations team and wait.
{% endhint %}

## Opening a payment

Click a row for the detail:

* The full **timeline** — every state it passed through, in order, with notes
* **What the recipient received**, in their currency
* Which **route** carried it
* Which **payout mode** it actually used — which may differ from the one you
  requested, if instant liquidity was not available
* A **payout difference**, shown only if the amount confirmed paid differed from
  the amount quoted. If you don't see it, promised and paid agreed exactly

## Payouts view

The **Payouts** page shows the same payments from the settlement side — what has
landed, what is still settling, what needs attention. Same data, different
question.

## Finding a payment

Search by recipient reference or transaction ID. Your reference is usually the
fastest route, which is a good reason to make it something your team recognises
when you send.

## Exporting

Export the list as CSV for a spreadsheet, or produce a printable receipt for a
single payment. See [Exports and receipts](exports-and-receipts.md).

## Getting told instead of looking

If your systems need to know about state changes, register a webhook endpoint and
Genifi pushes every transition to you — signed, with retries. See
[API keys and webhooks](api-keys-and-webhooks.md).
