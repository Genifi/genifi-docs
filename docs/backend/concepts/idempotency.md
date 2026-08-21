---
description: Retry anything, as often as you like, and send exactly one payment.
---

# Idempotency

Networks fail after the request arrives and before the response gets back. Your
process restarts mid-send. A queue redelivers. Every one of those situations
produces a duplicate call, and on a payment rail a duplicate call must not
produce a duplicate payment.

Genifi's answer is simple enough to state in one line:

> **Your idempotency key *is* the transaction ID.**

## How it works

```json
{
  "idempotencyKey": "pay-2026-08-21-0001",
  "…": "…"
}
```

```json
{ "transactionId": "pay-2026-08-21-0001", "status": "accepted" }
```

They are the same string. Send that request again — the same key, ten times, from
three processes — and there is still exactly one payment with that ID. Later
calls are accepted and de-duplicated rather than starting anything new.

That is also why every status read uses the key you chose:

```bash
curl "$GENIFI_API/transactions/pay-2026-08-21-0001" -H "Authorization: Bearer $GENIFI_API_KEY"
```

You never have to store a Genifi-generated ID to find your payment again. You
already know it, because you named it.

## Choosing keys

**Format.** 8–200 characters of `A–Z`, `a–z`, `0–9`, and `_ . : -`.

**Derive it from your own domain.** A good key is one your system can reproduce
from the same inputs after a crash:

```
order-88431-payout
inv_2026_08_21:cust_4471
transfer-5f2a8c1e-9b3d-4a77-8e21-0c6d9f4b2a10
```

**Do not use a random value generated at call time.** If your process dies before
recording it, your retry produces a *new* key — and a second payment. The key
must come from something stable: your order ID, your ledger entry ID, a UUID you
persisted *before* calling.

**Never reuse a key for a different payment.** The key identifies one payment
forever. Reusing it with different details does not create a second payment; it
collides with the first.

## Where idempotency keys apply

| Endpoint | Key becomes |
|---|---|
| `POST /transactions` | The transaction ID |
| `POST /treasury/onramp` | The on-ramp ID |
| `POST /issuers/:issuerId/redeem` | The redemption ID |
| `POST /quotes` | *Optional* — links the quote to the payment you intend to make |

On `POST /quotes` the key is optional. Supply the future payment's key to bind
the quote's audit record to that payment and to require the same key when the
quote is accepted; omit it for a bare price check.

## Idempotency downstream

The same discipline applies inside Genifi, not just at your boundary. Every
external money-moving call we make carries its own key and is wrapped so that a
retry cannot double-execute it. We record intent before the call and the result
after, and assume any call may be retried.

That is why a network blip between Genifi and a payout partner does not become a
double payout to your recipient.

## What to do on an error

| Response | Safe to retry with the same key? |
|---|---|
| Timeout, no response | **Yes.** This is exactly the case idempotency exists for. |
| `503` | **Yes**, with backoff. The condition is ours and transient. |
| `429` | **Yes**, after `Retry-After`. |
| `400` | No — fix the request first. Retrying identical invalid input gets an identical rejection. |
| `403` | No. Your role, your KYB status, or wallet ownership needs to change. |
| `202` | No need — it already worked. |

When in doubt, retry with the same key. That is always safe, and it is always
safer than not knowing whether the payment exists.
