# Send a payment

Sending is a **two-step** flow: get a firm quote, then accept it. The price you
see is the price that executes.

You need the `member` role or higher.

## Step 1 — fill in the payment

Open **Payments** and start a new payment.

### Destination

Pick the corridor — the country and currency your recipient is paid in. Only
corridors your environment can actually serve appear in the list.

### Funding wallet

Choose which of your registered wallets funds the payment. If you have exactly
one, it is selected for you.

Where balances are available, the dashboard shows the wallet's balance and warns
you **before** spending a quote if the amount exceeds it.

### Amount

A positive decimal, e.g. `1000.00`, in the send currency.

Don't add more decimal places than the currency has. Over-precise amounts are
rejected rather than rounded — rounding is how you send an amount nobody asked
for.

### Recipient reference

Your own reference for this recipient — an ID from your system, a customer
number, anything you will recognise later. It appears on the transaction and in
exports.

### Beneficiary — where the money actually lands

The recipient's bank details. **On live corridors this is required**, and the
form tells you exactly which fields that corridor needs.

Different destinations need different routing detail, so the required set changes
with the corridor. The form validates formats — account length, BIC shape — before
the round-trip, so you find out about a typo immediately rather than after a
failed payment.

!!! success

    Saved a recipient in the [address book](address-book.md)? Choose it from the
    dropdown and the beneficiary fields fill in.

### Payout mode

On corridors that support it you can choose **instant**: the recipient is paid
immediately from pre-positioned local liquidity, while the settling legs catch up
behind.

Instant is a request, not a guarantee. If local liquidity does not cover it, the
payment proceeds the regular way and nothing breaks. Check the transaction
afterwards to see which mode it actually used.

Corridors without instant support always send the regular way.

## Step 2 — review the quote

Click through and you get a firm quote:

* **What the recipient receives**, in their currency — the number that matters
* The fee and effective rate
* An estimated time to payout
* A countdown to expiry

**Nothing has moved yet.** A quote is a price, not a payment.

The quote lock lasts about a minute. Let it expire and the dashboard refuses to
send, and asks you to refresh for a current price. That is the intended
behaviour: it is better to show a customer a new number than to silently charge
them at a price they never saw.

## Step 3 — confirm

Confirm, and the payment starts. You get a transaction ID immediately, and the
payment moves through its states on screen.

The payment executes at **exactly** the quoted price. The quote and the payment
are bound together, so what you approved is what runs.

## Watching it land

The payment appears straight away in [Transactions](track-transactions.md) with a
live timeline:

```
Initiated → Quoted → Funded → Routing → Settling → Paid out → Reconciled
```

**Paid out** means the recipient has the money. **Reconciled** means everything
agrees and the payment is genuinely finished. Tell your customer at *paid out*;
close your books at *reconciled*.

## If something goes wrong

| Message | What it means |
|---|---|
| "Enter a positive amount" | The amount is zero, negative, or not a number |
| "Choose the wallet to fund this payment from" | No funding wallet selected |
| "…needs a complete beneficiary — missing: …" | The corridor requires those fields. The message names them |
| "You're sending more than this wallet holds" | Reduce the amount, or fund the wallet |
| "This quote has expired" | Refresh for a current price |
| "Could not get a quote" | The corridor could not be priced right now. Try again shortly |

More in [Troubleshooting](../reference/troubleshooting.md).

## Sending won't work if…

* **Your role is `viewer`.** Ask an admin for `member` or above.
* **Your organization is not yet verified.** Money movement is gated on
  verification. Contact your Genifi account team.
* **The banner says degraded or unreachable.** New payments cannot start while
  the backend is degraded. Existing ones are unaffected.

## Duplicate payments

Every payment carries an idempotency key generated when you request the quote and
reused when you confirm. Clicking confirm twice, or a network retry, cannot
produce two payments.

If you are unsure whether a payment went through, **check
[Transactions](track-transactions.md) rather than sending again**.
