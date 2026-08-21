# The address book

Saved recipients, so nobody retypes bank details on every payment.

Everyone can **see** the address book. Adding, editing, and deleting need the
`member` role or higher — the same floor as sending a payment, since you save a
payee in order to pay it.

## Two kinds of recipient

**Bank** — paid into a bank account. Needs an account number, plus whatever
routing detail the destination requires.

**Wallet** — paid to an on-chain address. Needs the address and, usually, which
network it is on.

The type is fixed when you create the recipient. A payee that changes from a bank
account to a wallet is a different payee — create a new one, rather than editing
across.

## Adding one

Open **Address book** and add a recipient:

| Field | Notes |
|---|---|
| **Name** | As the destination institution holds it. Mismatches are a common cause of returned payments |
| **Type** | Bank or wallet |
| **Account** | For bank recipients |
| **Address** | For wallet recipients |
| **Bank name, code, BIC** | Routing detail, as the destination needs it |
| **Country, city, address** | |
| **Network** | For wallet recipients |

Which optional fields matter depends on where the money is going — a Vietnamese
bank transfer and a Nigerian one do not want the same routing detail.

## Using one

In the payment form, pick a saved recipient from the dropdown and the beneficiary
fields fill in. Review them, then send as usual.

You can still adjust the fields for that one payment. Editing them there does not
change the saved recipient.

## Editing and deleting

Edit changes the saved record from that point on. **It does not rewrite history**
— a payment already sent keeps the details it actually used.

That separation is deliberate. A record of where money went must not change
because somebody later corrected a typo in an address book entry.

Deleting removes the recipient from the book. Payments already sent to them are
untouched.

## Recently used

Recipients carry a "last used" timestamp, so the people you pay often are easy to
find without searching.

## A good recipient reference

Separate from the address book, every payment carries a **recipient reference** —
your own identifier for who is being paid.

Make it something your team recognises: a customer number, an order ID, an
internal payee ID. It appears on the transaction, in exports, and in receipts,
and it is usually the fastest way to find a payment later.
