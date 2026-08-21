# Statements

A read-only report of what your organization sent in a period, and the fee Genifi
realized on it.

## GET /statements

{% tabs %}
{% tab title="Request" %}
```bash
curl "$GENIFI_API/statements?from=2026-08-01T00:00:00Z&to=2026-09-01T00:00:00Z" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "integratorId": "int_4f21",
  "period": { "from": "2026-08-01T00:00:00.000Z", "to": "2026-09-01T00:00:00.000Z" },
  "lines": [
    {
      "transactionId": "pay-2026-08-21-0001",
      "corridor": { "from": "USDC", "to": "VND", "destinationCountry": "VN" },
      "sendAmount": { "amount": "1000000000", "currency": "USDC", "scale": 6 },
      "recipientReference": "customer-4471",
      "fee": [ { "amount": "4200000", "currency": "USDC", "scale": 6 } ],
      "initiatedAt": "2026-08-21T14:02:07.000Z"
    }
  ],
  "totals": {
    "transactionCount": 1,
    "initiatedVolume": [ { "amount": "1000000000", "currency": "USDC", "scale": 6 } ],
    "feesEarned": [ { "amount": "4200000", "currency": "USDC", "scale": 6 } ]
  },
  "generatedAt": "2026-09-01T06:00:00.000Z"
}
```
{% endtab %}
{% endtabs %}

### Query parameters

| Parameter | Type | Notes |
|---|---|---|
| `from` | ISO 8601 with offset | Defaults to 30 days before `to` |
| `to` | ISO 8601 with offset | Defaults to now |

The interval is half-open: `[from, to)`. A transaction initiated exactly at `to`
belongs to the next period, not this one.

`from` must be strictly before `to`, or you get a `400`:

```json
{ "error": "invalid_period", "message": "from must be strictly before to" }
```

### Response

| Field | Type | Notes |
|---|---|---|
| `period` | object | The window actually used, after defaults were applied |
| `lines[]` | array | One entry per transaction initiated in the period |
| `lines[].fee` | Money[] | The **realized** fee, per currency. Empty when nothing was realized |
| `totals.transactionCount` | number | |
| `totals.initiatedVolume` | Money[] | Send notional initiated, per currency |
| `totals.feesEarned` | Money[] | Realized fee, per currency |
| `generatedAt` | ISO 8601 | When this snapshot was computed |

All figures are arrays keyed by currency, because a corridor mixes send and fee
currencies and a single number would have to pick one.

## What a statement is — and is not

**It is not an invoice.** Genifi takes its margin in-rate: the price you were
quoted already includes it. A statement reports what was earned; it is not a bill
and it does not represent a balance owed.

**It reports realized fees, not quoted ones.** A line's `fee` is what was
actually booked, not the hypothetical margin from quote time. Those two numbers
diverge whenever a payment refunds, fails, or settles at a different amount than
quoted — and billing off the quote-time figure would mean billing for revenue
that never existed.

**An empty `fee` is normal.** A refunded payment realizes nothing. So does a
payment on a zero-margin arrangement. An empty array means exactly that: nothing
was realized on this line.

**It is a snapshot, not a stored document.** `generatedAt` tells you when it was
computed. Requesting the same period again recomputes it, and the figures can
legitimately change if a payment in that window later reconciles or refunds.

## Availability

Where statements are not configured on your environment, the endpoint returns
`501`:

```json
{ "error": "not_implemented", "message": "statements are not configured" }
```
