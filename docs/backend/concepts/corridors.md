---
description: Where you can send, how routes are chosen, and why the recipient always gets fiat.
---

# Corridors

A **corridor** is a route from a send currency to a destination country and its
local currency.

```json
{ "corridor": { "to": "VND", "destinationCountry": "VN" } }
```

The send currency is not part of the `corridor` object: it comes from
`sendAmount.currency`. JET assembles the full corridor (`from` → `to` →
`destinationCountry`) from the two.

## Destination country is required, and it is not decorative

`destinationCountry` is a two-letter ISO country code and it is mandatory. The
same currency can be payable in more than one place under different rules, and
the payout partner, the compliance treatment, and the legality of the route all
depend on the country, not just the currency.

## What is available

Ask your environment rather than hard-coding a list:

```bash
curl "$JET_API/capabilities"
```

```json
"corridors": [
  { "from": "USDC", "to": "VND", "destinationCountry": "VN", "ownedRail": false, "instant": false },
  { "from": "USDC", "to": "NGN", "destinationCountry": "NG", "ownedRail": false, "instant": false },
  { "from": "USDC", "to": "IDR", "destinationCountry": "ID", "ownedRail": false, "instant": false },
  { "from": "USDC", "to": "INR", "destinationCountry": "IN", "ownedRail": false, "instant": false }
]
```

| Field | Meaning for you |
|---|---|
| `ownedRail` | JET operates this corridor end to end, rather than handing the crossing to a partner. Usually the better price. |
| `instant` | Always `false` today. Instant payout would need money pre-positioned in the destination country, and JET holds none. |

An unsupported corridor fails cleanly, before any money moves.

## How a route is chosen

You do not pick a route. For each payment, JET prices every rail that can serve
the corridor and can be funded from your balance. Where a corridor has a
preferred rail and that rail can serve your payment, it is used first; otherwise
the best price wins. The others stand behind it as fallbacks.

A [batch payout](../api/batches.md) is the exception: you name its rail, because
the whole batch is funded at one provider.

The [quote](../api/quotes.md) tells you which one won:

```json
{ "provider": "tazapay", "routeKind": "orchestrator", "estimatedSeconds": 300 }
```

`routeKind` is `owned` or `orchestrator`. Treat `provider` as an opaque label for
support and reconciliation, not as something to branch on: the set of routes
changes as corridors mature.

## The recipient always receives local fiat

This is a hard constraint, not a product preference.

In several destination markets, Vietnam most sharply, holding a stablecoin is
legal but *using one as a means of payment* is not. So the stablecoin leg of every
payment stays offshore, and the last mile into the destination is always local
fiat, disbursed by a licensed local partner.

There is no mode, flag, or endpoint that delivers a stablecoin to a recipient in
a destination country. If your product needs that, JET is not the right rail
for it.

## Adding a corridor

Corridors are opened by JET, not self-served: each one needs a licensed local
payout partner and a compliance review of the route. Talk to your account contact
about a destination you need, and it will appear in `/capabilities` when it is
live for you.
