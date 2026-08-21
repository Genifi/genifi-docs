# Rate limits

Limits are per **organization**, not per IP, and split across three buckets so
cheap status polling can never starve your ability to send a payment.

## The buckets

| Bucket | Routes | Burst | Sustained |
|---|---|---:|---:|
| `initiate` | `POST /transactions`, `POST /issuers/:issuerId/redeem` | 30 | 5 / second |
| `write` | Every other `POST`, `PUT`, `PATCH`, `DELETE` | 60 | 10 / second |
| `read` | Every `GET` and `HEAD` | 300 | 50 / second |

Each is a token bucket: **burst** is how many requests you can make back to back
from full, and **sustained** is how fast tokens refill.

At 5 payments a second sustained, one credential can start 18,000 payments an
hour — far beyond any real corridor volume, and far below the point where a
runaway loop does damage.

### Why `initiate` is separate

`POST /transactions` is categorically more expensive than a status read. It
writes a compliance record and starts a durable process that keeps running —
calling providers, posting ledger entries — long after the HTTP response is sent.

Sharing a bucket with reads would let a client start thousands of payments inside
an allowance sized for polling. `POST /issuers/:issuerId/redeem` is in the same
bucket for the same reason: it starts a durable operation, not because it happens
to be a `POST`.

## Headers

Every authenticated response carries them — not only a `429`:

```
ratelimit-limit: 30
ratelimit-remaining: 27
ratelimit-reset: 6
```

| Header | Meaning |
|---|---|
| `ratelimit-limit` | The bucket's capacity |
| `ratelimit-remaining` | Tokens left right now |
| `ratelimit-reset` | Seconds until the bucket refills |

They are on every response so you can slow down **before** being rejected, rather
than discovering the limit by hitting it.

## When you are limited

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 6
```

```json
{
  "error": "rate_limited",
  "message": "Too many initiate requests. Retry after 6s.",
  "retryAfterSeconds": 6
}
```

Wait at least `retryAfterSeconds`, then retry **with the same idempotency key**.
A `429` means the request was rejected before doing anything, so retrying is
always safe.

## Staying under the limit

**Use webhooks instead of polling.** The single biggest source of read traffic is
polling every open payment. [Register an endpoint](../webhooks/README.md) and the
problem disappears.

**Watch `ratelimit-remaining`.** When it drops below a threshold you choose, slow
down. Do not wait for the `429`.

**Back off exponentially.** A tight retry loop against a limit keeps the bucket
empty and makes the outage last longer.

**Queue your sends.** If you batch payments, pace them at your sustained rate
rather than firing the whole batch and absorbing rejections.

## Higher limits

The figures above are defaults. Per-organization limits can be raised — talk to
your account contact, ideally with a `429` rate from your own metrics so the
increase can be sized to real traffic.
