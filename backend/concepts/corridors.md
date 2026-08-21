---
description: Where you can send, how routes are chosen, and why the recipient always gets fiat.
---

# Corridors

A **corridor** is a route from a send currency to a destination country and its
local currency.

```json
{ "corridor": { "to": "VND", "destinationCountry": "VN" } }
```

The send currency is not part of the `corridor` object — it comes from
`sendAmount.currency`. Genifi assembles the full corridor (`from` → `to` →
`destinationCountry`) from the two.

## Destination country is required, and it is not decorative

`destinationCountry` is a two-letter ISO country code and it is mandatory. The
same currency can be payable in more than one place under different rules, and
the payout partner, the compliance treatment, and the legality of the route all
depend on the country — not just the currency.

## What is available

Ask your environment rather than hard-coding a list:

```bash
curl "$GENIFI_API/capabilities"
```

```json
"corridors": [
  { "from": "USDC", "to": "VND", "destinationCountry": "VN", "ownedRail": true,  "instant": true  },
  { "from": "USDC", "to": "NGN", "destinationCountry": "NG", "ownedRail": false, "instant": false },
  { "from": "USDC", "to": "PHP", "destinationCountry": "PH", "ownedRail": false, "instant": false },
  { "from": "USDC", "to": "INR", "destinationCountry": "IN", "ownedRail": false, "instant": false }
]
```

| Field | Meaning for you |
|---|---|
| `ownedRail` | Genifi operates this corridor end to end, rather than handing the crossing to a partner. Usually the better price. |
| `instant` | The corridor can pay the recipient immediately from pre-positioned local liquidity. Ask for it with `"payoutMode": "instant"`. |

An unsupported corridor fails cleanly, before any money moves.

## How a route is chosen

You do not pick a route. For each payment, Genifi prices every route that can
serve the corridor — its own rail where one exists, and partner rails otherwise —
and takes the best one for you.

The [quote](../api/quotes.md) tells you which one won:

```json
{ "provider": "owned-vn", "routeKind": "owned", "estimatedSeconds": 45 }
```

`routeKind` is `owned` or `orchestrator`. Treat `provider` as an opaque label for
support and reconciliation, not as something to branch on — the set of routes
changes as corridors mature.

## The recipient always receives local fiat

This is a hard constraint, not a product preference.

In several destination markets — Vietnam most sharply — holding a stablecoin is
legal but *using one as a means of payment* is not. So the stablecoin leg of every
payment stays offshore, and the last mile into the destination is always local
fiat, disbursed by a licensed local partner.

There is no mode, flag, or endpoint that delivers a stablecoin to a recipient in
a destination country. If your product needs that, Genifi is not the right rail
for it.

## Adding a corridor

Corridors are opened by Genifi, not self-served: each one needs a licensed local
payout partner and a compliance review of the route. Talk to your account contact
about a destination you need, and it will appear in `/capabilities` when it is
live for you.
