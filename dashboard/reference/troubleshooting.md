# Troubleshooting

## Signing in

**No email arrives.**
Your address has to belong to an active member of an organization. Check spam,
then ask an admin whether you were actually invited — and at which address. If
the invitation bounced, they can resend it, which issues a fresh link.

**"Invalid or expired link".**
Magic links are single-use and time-limited. Request a new one. If you clicked an
older link from a previous email, only the most recent one works.

**Signed in, but everything is read-only.**
Your role is `viewer`. Ask an admin to change it — see
[Your team](../guides/team-and-roles.md).

**Signed out unexpectedly.**
The session expired, or you signed out in another tab. Sign in again.

## The banner

**Degraded.**
The backend is reachable but cannot start new payments right now. Payments
already running are unaffected and will continue. Wait; the banner clears on its
own.

**Unreachable.**
The dashboard cannot reach the backend at all. Check your own network first. If
it persists, contact support — and note that payments already running are still
running, whatever the dashboard can see.

## Sending a payment

**"Enter a positive amount."**
The amount is zero, negative, or not a valid number. Use a plain decimal like
`1000.00`.

**Over-precise amount rejected.**
More decimal places than the currency has. Amounts are rejected rather than
rounded — rounding would send an amount nobody asked for.

**"Choose the wallet to fund this payment from."**
No funding wallet is selected. If you have none registered, see
[Wallets and funding](../guides/wallets-and-funding.md).

**"…needs a complete beneficiary — missing: …"**
That corridor requires those specific fields. The message names them. Different
destinations need different routing detail.

**"You're sending more than this wallet holds."**
Reduce the amount or fund the wallet. This check only appears where balances are
available; elsewhere the payment fails cleanly at the funding step instead.

**"This quote has expired."**
Quotes lock for about a minute. Refresh for a current price. This is deliberate —
better to show a new number than to charge at a price nobody saw.

**"Could not get a quote."**
The corridor could not be priced at that moment. Try again shortly. If it
persists, check whether the corridor is available on your environment.

**Sending is blocked entirely.**
Either your role is `viewer`, or your organization is not yet verified. Money
movement is gated on verification; contact your Genifi account team.

## Payments

**Stuck on "Paid out".**
Not stuck. The recipient has the money; reconciliation is still running.
**Reconciled** is the finished state. See
[Track transactions](../guides/track-transactions.md).

**A payment says "In doubt".**
Genifi cannot yet tell whether funds moved, so it has parked the payment rather
than guessing. It is re-queried automatically and will reach a terminal state on
its own.

{% hint style="danger" %}
Do not send a replacement. That is how an ambiguous payment becomes a confirmed
double payment. Escalate to operations and wait.
{% endhint %}

**A payment refunded.**
A step failed after funds moved, so the earlier steps were reversed. Your money
is back. The timeline shows where it stopped.

**The payout mode isn't the one I asked for.**
Instant is a request, not a guarantee. If local liquidity did not cover it, the
payment went the regular way. The transaction shows what actually happened.

**I'm not sure whether a payment went through.**
Check the Transactions list. Do not send again — search by your recipient
reference or the transaction ID.

## Balances and accounts

**A balance is missing.**
Either no provider is connected on your environment, or that account type does
not report balances. The dashboard says which. It never shows a blank in place of
an answer.

**A balance looks out of date.**
Provider balances are labelled with when they were reported. They are reports
reconciled against Genifi's ledger, not live figures.

**A page shows "not available".**
See [What's available](feature-availability.md).

## Webhooks

**Deliveries stopped after re-registering.**
Re-registering mints a fresh signing secret and kills the old one. Update the
secret in your receiver.

**Signature verification fails.**
Usually one of: verifying the *parsed* body instead of the raw bytes; a stale
secret; or a clock skew large enough to fail the freshness check. See
[Verifying signatures](../../backend/webhooks/verifying.md).

**I lost the signing secret.**
It cannot be recovered. Register the URL again for a new one.

## Team

**An invitation never arrived.**
Resend it. That issues a fresh token and kills the old one — the only way back
from a bounced invitation.

**I can't grant the admin role.**
Only the owner can create another admin.

**I removed someone — is that enough?**
Their session and personal key are dead. If they also had the **organization**
API key, [rotate it](../guides/api-keys-and-webhooks.md#rotating) too. That is a
separate step.

## Still stuck?

Contact your Genifi account contact with:

* the **transaction ID** or recipient reference, if it's about a payment
* the **time** it happened, with your timezone
* what you saw, and what you expected

Those three make almost any issue traceable on the first reply.
