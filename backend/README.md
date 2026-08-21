---
description: >-
  The Genifi API moves value across borders for your customers and delivers
  local fiat to the recipient.
---

# Genifi API

Genifi is a cross-border payment rail for regulated businesses. You send a
stablecoin amount; your recipient is paid in their **local currency**, through a
licensed local payout channel in the destination country.

The pattern is sometimes called a *stablecoin sandwich*: value moves in as fiat
or stablecoin, travels across borders as a stablecoin, and lands as local fiat.
The recipient never holds, receives, or needs to know about a stablecoin.

{% hint style="info" %}
**Genifi is a business-to-business rail.** You keep the relationship with your
end users and you own their KYC. Genifi verifies **you** (KYB) and provides the
corridor.
{% endhint %}

## What you can do with it

| | |
|---|---|
| **Price a payment** | Get a firm, rate-locked quote before committing to anything. |
| **Send a payment** | One authenticated call starts a durable payment that runs to completion on its own. |
| **Track it** | Poll a status endpoint, or receive signed webhooks on every state change. |
| **Manage recipients** | Save payees once, reuse them across payments. |
| **Manage your team** | Invite colleagues, assign roles, rotate your API key. |
| **Reconcile** | Pull a per-period statement of what you sent and what it cost. |

## The shortest possible example

```bash
curl -X POST "$GENIFI_API/transactions" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "idempotencyKey": "pay-2026-08-21-0001",
    "corridor": { "to": "VND", "destinationCountry": "VN" },
    "sendAmount": { "amount": "1000.00", "currency": "USDC" },
    "integratorWalletId": "wal_your_wallet",
    "recipientReference": "customer-4471"
  }'
```

```json
{ "transactionId": "pay-2026-08-21-0001", "status": "accepted" }
```

That `202 Accepted` means the payment is now running as a durable process on our
side. It will keep going — retrying providers, posting to our ledger, and, if
something goes wrong, unwinding itself — whether or not your process is still
alive. You find out how it ended by [polling](api/transactions.md#get-transactions-id)
or by [webhook](webhooks/README.md).

## Design principles you will notice

These are worth knowing up front, because they explain choices that might
otherwise look strange.

**Amounts are never JSON numbers.** You send decimal *strings*; we return integer
*minor units*. A JSON number cannot carry a 6-decimal stablecoin amount without
losing precision, and losing precision on money is not an acceptable failure
mode. See [Amounts and currencies](concepts/money.md).

**Every money-moving call is idempotent, and the key is the ID.** Your
`idempotencyKey` *becomes* the transaction ID. Retry the same call as often as
you like; you get one payment. See [Idempotency](concepts/idempotency.md).

**We fail closed and we never fake success.** If a feature is not wired up on
your environment, the endpoint returns `501` with a machine-readable reason —
not an empty list, not a plausible-looking stub. If we cannot tell whether funds
moved, the payment goes to `IN_DOUBT` and is reconciled rather than optimistically
marked complete.

**The API tells you what it can do.** [`GET /capabilities`](api/discovery.md)
returns the live corridors and the status of every feature, so you can build
against what is actually available instead of hard-coding assumptions.

## Where to go next

* New here? Start with the [Quickstart](getting-started/quickstart.md).
* Integrating properly? Read [How a payment works](concepts/payment-lifecycle.md)
  and [Idempotency](concepts/idempotency.md) before writing code.
* Looking for a specific endpoint? Jump to the [API reference](api/conventions.md).
