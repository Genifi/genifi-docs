---
description: Two amount formats, one rule — never a JSON number.
---

# Amounts and currencies

Genifi never represents money as a JSON number, and neither should you.

A JSON number is an IEEE 754 double. It cannot hold a 6-decimal stablecoin
amount without silent precision loss, and a fraction of a cent that disappears
in transit is exactly the class of bug that money systems exist to prevent.

## The two shapes

**In a request, you send a decimal string:**

```json
"sendAmount": { "amount": "1000.00", "currency": "USDC" }
```

**In a response, you receive integer minor units:**

```json
"receiveAmount": { "amount": "24512750", "currency": "VND", "scale": 0 }
```

Both `amount` values are **strings**. The response adds `scale` — the number of
decimal places the currency has.

## Reading a response amount

Divide by `10^scale`:

| Response | Scale | Means |
|---|---|---|
| `{ "amount": "24512750", "currency": "VND" }` | 0 | 24,512,750 ₫ |
| `{ "amount": "1500000", "currency": "USDC" }` | 6 | 1.50 USDC |
| `{ "amount": "129900", "currency": "EUR" }` | 2 | €1,299.00 |

```ts
const SCALES: Record<string, number> = {
  USD: 2, EUR: 2, GBP: 2, CZK: 2, NGN: 2, PHP: 2, INR: 2,
  VND: 0,
  USDC: 6, EURC: 6,
};

function format(amount: string, currency: string): string {
  const scale = SCALES[currency] ?? 2;
  if (scale === 0) return amount;
  const negative = amount.startsWith("-");
  const digits = (negative ? amount.slice(1) : amount).padStart(scale + 1, "0");
  const whole = digits.slice(0, -scale);
  const frac = digits.slice(-scale);
  return `${negative ? "-" : ""}${whole}.${frac}`;
}
```

Do the arithmetic on integers, or on a decimal library. Do not route the string
through `parseFloat` on the way to a display, and never through a float on the
way to a total.

## Supported currencies and their scales

| Currency | Scale | Role |
|---|:---:|---|
| `USDC` | 6 | Stablecoin — the send currency |
| `EURC` | 6 | Euro stablecoin — held after a EUR top-up |
| `USD` | 2 | Settlement |
| `EUR` | 2 | Pay-in |
| `GBP` | 2 | Pay-in |
| `CZK` | 2 | Pay-in |
| `VND` | 0 | Payout — Vietnam |
| `NGN` | 2 | Payout — Nigeria |
| `PHP` | 2 | Payout — Philippines |
| `INR` | 2 | Payout — India |

Vietnamese đồng has no subunit in practice, which is why its scale is `0`. A VND
amount of `"24512750"` is twenty-four and a half million đồng, not two hundred
and forty-five thousand.

## Over-precision is rejected, not rounded

Send more decimal places than the currency has and the request fails with a
`400`:

```json
{ "sendAmount": { "amount": "1.0000001", "currency": "USDC" } }
```

```json
{ "error": "validation", "message": "Invalid sendAmount" }
```

We do not round. Rounding is how you send an amount nobody asked for.

## Rounding on the payout side

`receiveAmount` is **floored** to whole units of the destination currency. Your
customer will never be quoted more than the recipient receives.

## No implicit conversion

Every currency conversion in Genifi is an explicit, logged operation. There is no
endpoint that silently converts one currency into another as a side effect of
doing something else. If a payment involves FX, the rate is in the quote and the
conversion is a step in the [lifecycle](payment-lifecycle.md).
