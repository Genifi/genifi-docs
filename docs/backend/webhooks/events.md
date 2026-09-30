# Event types

Six events for a payment, one per transition worth telling you about, and three
for deposits and withdrawals, one per outcome.

| Event | Fires when | Typical reaction |
|---|---|---|
| `transaction.quoted` | A price has been locked for the payment | Record the rate |
| `transaction.funded` | Funds have left your wallet and are held by Genifi | Mark the customer's funds as committed |
| `transaction.paid_out` | The local partner confirms the recipient has been paid | Tell your customer the money has landed |
| `transaction.reconciled` | Books, chain, and provider all agree — the payment is **done** | Settle your own accounting |
| `transaction.refunded` | An unwind completed and the money came back | Return the funds to your customer |
| `transaction.in_doubt` | The outcome is ambiguous and has been parked | Alert operations. **Do not** retry the payment |
| `onramp.delivered` | A deposit was converted and the USDC reached your wallet | Credit your customer |
| `onramp.refunded` | A deposit could not be converted and the money went back to the payer | Tell the payer; nothing reached your wallet |
| `offramp.paid` | A withdrawal to a bank account reached the beneficiary | Tell the beneficiary the money has been sent |

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
| `event` | string | One of the events above |
| `transactionId` | string | The idempotency key you chose — for a payment, or for the deposit or withdrawal an `onramp.*` / `offramp.*` event is about |
| `data` | object | Event-specific payload |

`data` carries the state, the transaction's own terms, and any amounts known at
that point:

```json
{
  "state": "PAID_OUT",
  "sendAmount": { "amount": "100000", "currency": "USD", "scale": 2 },
  "receiveAmount": { "amount": "24512750", "currency": "VND", "scale": 0 },
  "corridor": { "from": "USD", "to": "VND", "destinationCountry": "VN" },
  "occurredAt": "2026-08-21T09:14:02.000Z",
  "provider": "tazapay",
  "payoutMode": "regular"
}
```

| Field | Present when |
|---|---|
| `state` | Always |
| `sendAmount` | Always — what you asked us to send |
| `corridor` | Always |
| `occurredAt` | Always — when **this transition** happened, UTC |
| `receiveAmount` | Once the recipient amount is known |
| `provider` | Once a route has been chosen |
| `payoutMode` | Once a route has been chosen (`regular` or `instant`) |

New fields may be added to `data`. Ignore ones you do not recognise rather than
rejecting the delivery.

Amounts are integer minor units. See
[Amounts and currencies](../concepts/money.md).

## Deposit and withdrawal events

A deposit or withdrawal is announced once, when it finishes. The envelope is the
same, and `transactionId` carries the idempotency key you sent to
`POST /treasury/onramp`, `POST /treasury/onramp/pix` or `POST /treasury/offramp`.
A wire that arrived without a request of yours carries the id Genifi gave it:

```json
{
  "deliveryId": "dep-2026-09-30-0001:onramp.delivered",
  "event": "onramp.delivered",
  "transactionId": "dep-2026-09-30-0001",
  "data": {
    "onRampId": "dep-2026-09-30-0001",
    "state": "DELIVERED",
    "sendAmount": { "amount": "100000", "currency": "EUR", "scale": 2 },
    "receiveAmount": { "amount": "107946000", "currency": "USDC", "scale": 6 },
    "fee": { "amount": "54000", "currency": "USDC", "scale": 6 },
    "txHash": "0x9c4e…"
  }
}
```

| Field | Event | Meaning |
|---|---|---|
| `onRampId` / `offRampId` | all | The deposit or withdrawal id, again |
| `state` | all | `DELIVERED`, `REFUNDED` or `PAID` |
| `sendAmount` | all | What came in: the fiat deposited, or the USDC withdrawn |
| `receiveAmount` | `onramp.delivered`, `offramp.paid` | What went out: the USDC credited, or the fiat the beneficiary receives |
| `fee` | when charged | Genifi's fee |
| `txHash` | `onramp.delivered` | The on-chain transaction that credited your wallet |
| `reason` | `onramp.refunded` | Why the deposit could not be converted |
| `beneficiaryId` | `offramp.paid` | The beneficiary you registered |

A deposit or withdrawal that parks for review sends nothing — it is still running.
Read its status from the dashboard or the API.

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

The same events, except `transaction.quoted` and `transaction.funded`, back
per-person email notifications in the dashboard. Those are for humans; webhooks
are for systems. See [Notifications](../api/notifications.md).
