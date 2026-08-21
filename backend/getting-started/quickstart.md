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
| **Base URL** | The address of your Genifi environment. Written `$GENIFI_API` throughout these docs. |
| **API key** | Your organization's server-to-server credential. Written `$GENIFI_API_KEY`. |
| **A registered wallet** | The wallet Genifi funds the payment from. Its ID goes in `integratorWalletId`. |

```bash
export GENIFI_API="https://…"          # issued at onboarding
export GENIFI_API_KEY="gk_…"           # issued at onboarding
```

{% hint style="warning" %}
Your API key authenticates as your **whole organization** and can move money.
Keep it server-side. Never ship it in a browser bundle, a mobile app, or a
public repository. If it leaks, [rotate it](../api/team.md#post-api-keys-rotate)
immediately.
{% endhint %}

## 1. Check you are connected

```bash
curl "$GENIFI_API/health"
```

```json
{ "status": "ok", "version": "1.4.2", "uptimeSeconds": 41822, "checks": { "temporal": "ok" } }
```

`/health` needs no credentials. A `503` with `"status": "degraded"` means we
cannot start new payments right now — do not treat it as a reason to retry a
send in a tight loop.

## 2. Confirm your key works

```bash
curl "$GENIFI_API/me" -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "id": "int_4f21…", "name": "Northwind Payments", "role": "owner" }
```

This is the honest auth probe: it has no side effects and tells you *which*
organization the key belongs to. Useful when you hold keys for more than one
environment.

## 3. Find your wallet

```bash
curl "$GENIFI_API/wallets" -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{
  "accounts": [
    {
      "id": "acc_9c1a…",
      "kind": "wallet",
      "provider": "external",
      "externalId": "wal_your_wallet",
      "label": "USDC pay-in",
      "currencies": ["USDC"],
      "address": "0x…",
      "status": "active",
      "createdAt": "2026-07-02T09:14:00.000Z"
    }
  ],
  "count": 1
}
```

The value you pass as `integratorWalletId` is the **`externalId`**.

{% hint style="info" %}
Wallet listings deliberately carry **no balances**. Genifi's ledger is the
source of truth for value; a provider-reported balance is a report to be
reconciled against it, not an authority. Balances are exposed separately, where
a provider is wired — see [Wallets and accounts](../api/wallets-and-accounts.md).
{% endhint %}

## 4. Price the payment

A quote is free, moves nothing, and locks a rate for about a minute.

```bash
curl -X POST "$GENIFI_API/quotes" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "corridor": { "to": "VND", "destinationCountry": "VN" },
    "sendAmount": { "amount": "1000.00", "currency": "USDC" }
  }'
```

```json
{
  "quoteId": "qte_8b3d…",
  "sendAmount": { "amount": "1000000000", "currency": "USDC", "scale": 6 },
  "receiveAmount": { "amount": "24512750", "currency": "VND", "scale": 0 },
  "fee": { "amount": "4200000", "currency": "USDC", "scale": 6 },
  "rate": { "…": "…" },
  "provider": "owned-vn",
  "routeKind": "owned",
  "estimatedSeconds": 45,
  "expiresAt": "2026-08-21T14:03:11.000Z"
}
```

Show `receiveAmount` to your customer. That is what the recipient gets, floored
to whole units of the destination currency.

## 5. Send it

Pass the `quoteId` to execute at exactly that locked price.

```bash
curl -X POST "$GENIFI_API/transactions" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "idempotencyKey": "pay-2026-08-21-0001",
    "quoteId": "qte_8b3d…",
    "corridor": { "to": "VND", "destinationCountry": "VN" },
    "sendAmount": { "amount": "1000.00", "currency": "USDC" },
    "integratorWalletId": "wal_your_wallet",
    "recipientReference": "customer-4471",
    "beneficiary": {
      "name": "NGUYEN VAN A",
      "account": "0123456789",
      "bankName": "Vietcombank",
      "country": "VN"
    }
  }'
```

```json
{ "transactionId": "pay-2026-08-21-0001", "status": "accepted" }
```

## 6. Watch it finish

```bash
curl "$GENIFI_API/transactions/pay-2026-08-21-0001" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{
  "transactionId": "pay-2026-08-21-0001",
  "running": false,
  "state": "RECONCILED",
  "provider": "owned-vn",
  "payoutMode": "regular",
  "receiveAmount": { "amount": "24512750", "currency": "VND" },
  "timeline": [
    { "state": "INITIATED" },
    { "state": "QUOTED", "detail": "locked qte_8b3d…" },
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

* [Idempotency](../concepts/idempotency.md) — how to retry safely. Read this before going live.
* [Amounts and currencies](../concepts/money.md) — the two amount formats and why.
* [Errors](../reference/errors.md) — every error code and what to do about it.
