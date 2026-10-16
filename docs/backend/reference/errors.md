# Errors

Every error has the same shape:

```json
{ "error": "forbidden", "message": "your role may not initiate payments" }
```

Branch on `error`. It is the contract. `message` is written for humans and may be
reworded without notice.

Validation errors add `issues`:

```json
{
  "error": "validation",
  "message": "invalid request …",
  "issues": [
    { "path": ["sendAmount", "amount"], "message": "must be a positive decimal string" },
    { "path": ["corridor", "destinationCountry"], "message": "String must contain exactly 2 character(s)" }
  ]
}
```

Some errors add context fields: `transactionId`, `integratorWalletId`,
`itemRefs`, `missing`, `available`, `feature`, `requires`, `retryAfterSeconds`.
Read them; they usually name the exact thing to fix.

**The complete list** of codes, with the endpoints that return each one, is
[generated from the API source](../api-reference/errors.md). This page covers
what they mean and what to do.

## By status

### 400: your request

| `error` | Meaning | What to do |
|---|---|---|
| `validation` | A field is missing or malformed, or an amount has more decimals than its currency allows | Read `issues` |
| `beneficiary_incomplete` | The corridor needs a beneficiary field you did not send | Add the fields in `missing` |
| `duplicate_item_ref`, `invalid_send_amount`, `unknown_recipient` | A batch item is at fault | Fix the items named in `itemRefs` |
| `invalid_cursor` | A page cursor JET did not issue | Start again without `cursor` |
| `invalid_query` | A query parameter is malformed | Check the date format on `from`/`to` |
| `invalid_period` | `from` is not strictly before `to` | Swap or widen the window |
| `invalid_invite` | The invite token is invalid or already used | Ask for a fresh invitation |

Retrying an identical `400` produces an identical `400`. Fix the request.

### 401: credentials

| `error` | Meaning |
|---|---|
| `unauthorized` | Missing or invalid credentials |

Always the same message, whatever the cause: unknown key, malformed header,
suspended organization, expired session. The difference is information an
attacker can use.

Treat a `401` as *this credential is dead*. Do not retry it.

### 403: not allowed

| `error` | Meaning | What to do |
|---|---|---|
| `forbidden` | Your role may not do this | Use a credential with a higher role |
| `forbidden` | `integratorWalletId` is not a wallet you own | Omit it: payments are funded from your home wallet |
| `kyb_required` | Your organization is not yet verified | Contact your JET account team |
| `compliance_hold` | Sanctions screening blocked the payment | Route to your compliance team. **Never** auto-retry |

`compliance_hold` deliberately does not say which list or name matched: that is
the screened party's personal data, and JET must not function as a sanctions
oracle.

### 404: not found, or not yours

| `error` | Meaning |
|---|---|
| `not_found` | No such resource, **or** it belongs to another organization |

The two are indistinguishable on purpose. A distinct `403` would confirm that
another organization's payment exists, which is exactly the enumeration this
prevents. Do not read a `404` as proof that an ID was never valid.

### 409: conflict

| `error` | Meaning |
|---|---|
| `idempotency_key_conflict` | The key is already used, by you or another organization, for a different operation | Choose a new key |
| `idempotency_key_reused` | You reused a key with a different request | Resend the original request, or choose a new key |
| `conflict` | The resource already exists, or is owned by someone else |
| `member_required` | Authenticated as the organization, but this endpoint needs a person |
| `no_member_email` | The member has no address on file |

### 422: understood, but cannot be done as asked

The request is well formed; something about your account or the market means it
cannot go ahead **as asked**. Whether to retry depends on the code:

| `error` | Meaning | What to do |
|---|---|---|
| `quote_unavailable` | The corridor could not be priced at this moment | Retry shortly. Check `/capabilities` if it persists |
| `quote_expired`, `quote_not_found` | The quote's lock is gone, or it never existed | Request a new quote and show the new price |
| `quote_mismatch` | The request does not match the quote | Send the amount and corridor (or beneficiary) you quoted |
| `quote_already_used` | An off-ramp quote was already accepted | Request a new quote |
| `insufficient_funds` | Your home wallet holds less than the payment | Fund your balance; `available` says what it holds |
| `not_home_wallet`, `not_home_chain` | You named a wallet or network other than your home one | Omit it |
| `no_home_wallet`, `ambiguous_home_wallet`, `wallet_chain_unknown` | Your account's wallet setup is incomplete | Contact us. Retrying will not help |
| `no_route_for_chain` | No rail can be funded from your balance's network | Contact us. Retrying will not help |
| `unsupported_network` | The network you named is not available for withdrawals | Use a network listed by `GET /treasury/crypto-deposit` |
| `pix_onramp_unavailable` | PIX could not open a code right now | Retry shortly |

### 429: rate limited

```json
{
  "error": "rate_limited",
  "message": "Too many initiate requests. Retry after 6s.",
  "retryAfterSeconds": 6
}
```

Honour `Retry-After`. See [Rate limits](rate-limits.md).

### 501: not wired up here

```json
{
  "error": "not_implemented",
  "feature": "banking",
  "message": "wire transfers need a custody bank adapter",
  "requires": "a production banking partner"
}
```

The endpoint exists in the contract but its adapter is not configured on your
environment. `requires` says what it would take. Check
[`/capabilities`](../api/discovery.md#get-capabilities) rather than probing.

A `501` is never a transient condition. Do not retry it.

### 503: our problem

Returned when a dependency of ours is unavailable, including any internally
retryable failure.

Retry with exponential backoff **and the same idempotency key**. That is exactly
the case idempotency exists for.

### 500: unexpected

```json
{ "error": "internal" }
```

No detail, deliberately. If it persists, contact support with the timestamp and
your transaction ID.

## Retry decision table

| Response | Retry? | With the same key? |
|---|---|---|
| Timeout / no response | Yes | Yes: always |
| `429` | Yes, after `Retry-After` | Yes |
| `503` | Yes, with backoff | Yes |
| `500` | Once, with backoff | Yes |
| `400` | No | n/a |
| `401` | No | n/a |
| `403` | No | n/a |
| `404` | No | n/a |
| `409` | No | n/a |
| `422` | Only `quote_unavailable` and `pix_onramp_unavailable`, after a pause. Otherwise change the request | Yes |
| `501` | No | n/a |

When in doubt on a money-moving call, retry with the same idempotency key. It is
always safe, and always safer than not knowing whether the payment exists. See
[Idempotency](../concepts/idempotency.md).
