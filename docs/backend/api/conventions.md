---
description: The rules that hold across every endpoint.
---

# Conventions

## Base URL

JET issues your base URL at onboarding. Throughout this reference it is
written `$JET_API`.

```bash
export JET_API="https://…"
export JET_API_KEY="gk_…"
```

The same contract is published as an
[OpenAPI 3.1 document](../api-reference/openapi.json), generated from the API
source, for client generators and API tools.

## Requests

* JSON in, JSON out. Send `Content-Type: application/json` on every request with
  a body.
* Authenticate with `Authorization: Bearer $JET_API_KEY`. See
  [Authentication](../getting-started/authentication.md).
* **Know whether a body is strict.** Some request bodies reject a field they do
  not recognise with `400`; most ignore it. The
  [API reference](../api-reference/README.md) says which, per endpoint. An
  ignored field is a silently missing one, so check the spelling of optional
  fields: a misspelled `quoteId` is a payment priced at execution time instead.
* Amounts are decimal strings on the way in. See
  [Amounts and currencies](../concepts/money.md).

## Responses

The payload is returned **directly**. There is no `{ "data": … }` envelope.

```json
{ "transactionId": "pay-2026-08-21-0001", "status": "accepted" }
```

Errors are a consistent shape:

```json
{ "error": "forbidden", "message": "your role may not initiate payments" }
```

Validation errors add `issues`, one entry per problem:

```json
{
  "error": "validation",
  "message": "invalid request …",
  "issues": [
    { "path": ["sendAmount", "amount"], "message": "must be a positive decimal string" }
  ]
}
```

Branch on `error`, never on `message`. Message text is written for humans and
may be reworded; the `error` code is the contract. Full list:
[Errors](../reference/errors.md).

## Status codes

| Code | When |
|---|---|
| `200` | Read succeeded, or a write completed synchronously |
| `201` | A resource was created (a recipient, an invitation, a beneficiary) |
| `202` | A durable operation **started**: a payment, batch, deposit, withdrawal or off-ramp |
| `400` | Your request is malformed or invalid |
| `401` | Missing or invalid credentials |
| `403` | Authenticated, but not allowed: role, KYB status, ownership, or compliance |
| `404` | No such resource, **or** it is not yours |
| `409` | Conflict: e.g. an idempotency key already used for something else |
| `422` | Understood but unsatisfiable right now: e.g. a corridor cannot be priced |
| `429` | Rate limited. See [Rate limits](../reference/rate-limits.md) |
| `501` | The feature is not wired up on this environment |
| `503` | A dependency of ours is unavailable. Retry with backoff |

## 202 means started, not finished

These return `202 Accepted`: `POST /transactions`, `POST /batches`,
`POST /treasury/onramp`, `POST /treasury/onramp/pix`,
`POST /treasury/withdrawals` and `POST /treasury/offramp`.

Each starts a durable operation that continues on our side. The response tells
you the ID; you learn the outcome by polling the matching `GET`, or by
[webhook](../webhooks/README.md).

## 404 also means "not yours"

Reading a payment, batch, deposit, withdrawal or off-ramp that belongs to
another organization returns exactly the same `404` as an ID that does not
exist.

This is deliberate. A distinct `403` would confirm that someone else's payment is
real, which is precisely the enumeration the check exists to prevent. Do not
interpret a `404` as proof that an ID was never valid.

## Ownership is derived, never declared

No endpoint accepts an organization ID in its body. Your organization comes from
your API key on every call. Where a request names a resource (a funding wallet,
for instance) we verify you own it before doing any work, and fail closed if we
cannot prove it.

## Pagination

The lists that grow without bound are paginated with an opaque **cursor**:
`GET /transactions`, `GET /batches` (and the items of `GET /batches/:batchId`),
`GET /treasury/onramps`, `GET /treasury/withdrawals` and `GET /treasury/offramps`.
Each accepts `?limit=` and `?cursor=` and returns a `nextCursor` alongside the
rows. The maximum `limit` is 500, or 200 for withdrawals and off-ramps:

```json
{
  "transactions": [ … ],
  "count": 50,
  "nextCursor": "eyJ0IjoiMjAyNi0wOC0yMVQxNDowMjowNy4wMDBaIn0",
  "hasMore": true
}
```

To walk a list, send `nextCursor` back as `?cursor=` and repeat until it comes
back `null`. That `null` is the end of the list: the only end-of-list signal.

```bash
curl "$JET_API/transactions?limit=50&cursor=$CURSOR" \
  -H "Authorization: Bearer $JET_API_KEY"
```

A few properties worth relying on:

- **`count` is the page, not the total.** It is the number of rows in the
  response you are holding. We do not return a total row count.
- **`hasMore` is authoritative.** A page that comes back exactly `limit` rows
  long is not evidence that another page exists; only `hasMore` / a non-null
  `nextCursor` is.
- **A cursor is opaque.** It encodes the sort position of the last row we sent.
  Echo it back unchanged; do not construct, parse, or modify one. A cursor we
  did not issue is rejected with `400 invalid_cursor`: never a silently empty
  page, which you would otherwise read as the end of your history.
- **Cursors are stable under writes.** These lists are newest-first over
  append-only records, so new rows arrive at the *top*. A cursor names a row
  rather than counting from the top, so a payment initiated (or a wire that
  lands) while you are paging cannot make a page repeat rows or skip them. This
  is why there is no `offset`.

Other list endpoints (`/recipients`, `/members`, `/wallets`,
`/issuers/accounts`, `/treasury/beneficiaries`) are bounded per organization and
return the full set with a `count`:

```json
{ "accounts": [ … ], "count": 3 }
```

## Timestamps

ISO 8601 with an offset, always UTC:

```
2026-08-21T14:03:11.000Z
```

Query parameters that take a time (`from` and `to` on `GET /statements`) expect
the same format.

## Rate limit headers

Every authenticated response carries them, not just a `429`:

```
ratelimit-limit: 30
ratelimit-remaining: 27
ratelimit-reset: 6
```

Back off before you are rejected, rather than discovering the limit by hitting
it. See [Rate limits](../reference/rate-limits.md).
