---
description: Send your first payment end to end, in five steps.
---

# Quickstart

By the end of this page you will have priced a payment, sent it, and watched it
reach a terminal state.

## Before you start

You need three things, all issued during onboarding:

| | |
|---|---|
| **Base URL** | The address of your JET environment. Written `$JET_API` throughout these docs. |
| **API key** | Your organization's server-to-server credential. Written `$JET_API_KEY`. |
| **A funded balance** | USDC in your JET balance. You get a wallet at onboarding; fund it with a [deposit](../api/treasury.md). |

```bash
export JET_API="https://…"          # issued at onboarding
export JET_API_KEY="gk_…"           # issued at onboarding
```

!!! warning

    Your API key authenticates as your **whole organization** and can move money.
    Keep it server-side. Never ship it in a browser bundle, a mobile app, or a
    public repository. If it leaks, [rotate it](../api/team.md#post-api-keysrotate)
    immediately.

## 1. Check you are connected

```bash
curl "$JET_API/health"
```

```json
{ "status": "ok", "version": "1.4.2", "uptimeSeconds": 41822, "checks": { "temporal": "ok" } }
```

`/health` needs no credentials. A `503` with `"status": "degraded"` means we
cannot start new payments right now: do not treat it as a reason to retry a
send in a tight loop.

## 2. Confirm your key works

```bash
curl "$JET_API/me" -H "Authorization: Bearer $JET_API_KEY"
```

```json
{ "id": "int_4f21…", "name": "Northwind Payments", "role": "owner" }
```

This is the honest auth probe: it has no side effects and tells you *which*
organization the key belongs to. Useful when you hold keys for more than one
environment.

## 3. Check your balance

```bash
curl "$JET_API/balance" -H "Authorization: Bearer $JET_API_KEY"
```

```json
{
  "currency": "USDC",
  "scale": 6,
  "available": "2500000000",
  "settling": "0",
  "total": "2500000000",
  "complete": true,
  "reportedAt": "2026-08-21T14:01:30.000Z"
}
```

`available` is what a payment can spend now: 2,500 USDC here, in integer minor
units at `scale` 6. You never name a wallet when you pay: every payment is funded
from your **home wallet**, where this balance lives. See
[Balance and wallets](../api/wallets-and-accounts.md).

## 4. Price the payment

A quote is free, moves nothing, and locks a rate for about a minute.

```bash
curl -X POST "$JET_API/quotes" \
  -H "Authorization: Bearer $JET_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "corridor": { "to": "VND", "destinationCountry": "VN" },
    "sendAmount": { "amount": "1000.00", "currency": "USDC" }
  }'
```

```json
{
  "quoteId": "q_3d6e9932-61c7-49cb-9259-9703dabf3cdc",
  "sendAmount": { "amount": "1000000000", "currency": "USDC", "scale": 6 },
  "receiveAmount": { "amount": "24904406", "currency": "VND", "scale": 0 },
  "fee": { "amount": "0", "currency": "VND", "scale": 0 },
  "rate": { "from": "USDC", "to": "VND", "numerator": "…", "denominator": "…" },
  "provider": "tazapay",
  "routeKind": "orchestrator",
  "estimatedSeconds": 300,
  "expiresAt": "2026-08-21T14:03:11.000Z"
}
```

Show `receiveAmount` to your customer. That is what the recipient gets, floored
to whole units of the destination currency.

## 5. Send it

Pass the `quoteId` to execute at exactly that locked price.

```bash
curl -X POST "$JET_API/transactions" \
  -H "Authorization: Bearer $JET_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "idempotencyKey": "pay-2026-08-21-0001",
    "quoteId": "q_3d6e9932-61c7-49cb-9259-9703dabf3cdc",
    "corridor": { "to": "VND", "destinationCountry": "VN" },
    "sendAmount": { "amount": "1000.00", "currency": "USDC" },
    "recipientReference": "customer-4471",
    "beneficiary": {
      "name": "NGUYEN VAN A",
      "account": "0123456789",
      "bankName": "Vietcombank",
      "bic": "BFTVVNVX",
      "country": "VN"
    }
  }'
```

```json
{ "transactionId": "pay-2026-08-21-0001", "status": "accepted" }
```

## 6. Watch it finish

```bash
curl "$JET_API/transactions/pay-2026-08-21-0001" \
  -H "Authorization: Bearer $JET_API_KEY"
```

```json
{
  "transactionId": "pay-2026-08-21-0001",
  "running": false,
  "state": "RECONCILED",
  "provider": "tazapay",
  "payoutMode": "regular",
  "receiveAmount": { "amount": "24904406", "currency": "VND", "scale": 0 },
  "timeline": [
    { "state": "INITIATED" },
    { "state": "QUOTED", "detail": "locked q_3d6e9932…" },
    { "state": "FUNDED" },
    { "state": "ROUTING" },
    { "state": "SETTLING" },
    { "state": "PAID_OUT" },
    { "state": "RECONCILED" }
  ]
}
```

`RECONCILED` is the only state that means *done*. `PAID_OUT` means the recipient
has the money but our books, the chain, and the provider have not yet been proven
to agree. See [How a payment works](../concepts/payment-lifecycle.md).

## Stop polling: use webhooks

Polling is fine for a first integration. In production, [register a webhook
endpoint](../webhooks/README.md) and we will push every state change to you,
signed, with retries.

## Next steps

* [Idempotency](../concepts/idempotency.md): how to retry safely. Read this before going live.
* [Amounts and currencies](../concepts/money.md): the two amount formats and why.
* [Errors](../reference/errors.md): every error code and what to do about it.
* [API reference](../api-reference/README.md): every endpoint, generated from the API itself.
