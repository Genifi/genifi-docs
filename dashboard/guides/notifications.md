# Email notifications

Get an email when a payment reaches a state you care about. Preferences are
**per person**, not per organization — yours are yours.

Find them in **Settings**.

## What you can subscribe to

| Event | Fires when |
|---|---|
| **Quoted** | A price has been locked |
| **Funded** | Funds have left your wallet |
| **Paid out** | The recipient has been paid |
| **Reconciled** | Everything agrees — the payment is done |
| **Refunded** | An unwind completed and the money came back |
| **In doubt** | The outcome is ambiguous and has been parked |

If you have never changed your preferences, you are subscribed to the important
transitions only. The chatty intermediate ones are opt-in, so nobody gets flooded
on their first day.

## Choosing

Tick the events you want and save. Untick everything to stop all email.

### A sensible default

**Paid out**, **Refunded**, and **In doubt**.

Those are the three that need a person: the money landed, the money came back, or
something needs a human to look at it. Add **Reconciled** if you close books
daily.

## Verifying your address

Genifi only emails **verified** addresses.

If yours is not verified, Settings offers to send a confirmation link. Click the
link in that email and you are done.

You cannot enter an address to verify — it is always the address on your member
record. Otherwise this would be a way to send Genifi-branded mail to strangers.

If verification is not available on your environment, Settings says so rather
than offering a button that does nothing.

## If email stops arriving

Settings tells you when sending to your address has been **suppressed**, and why
— a hard bounce, for instance.

That is deliberate: "your address bounced" is something you can act on. Silence
is not.

Once the underlying problem is fixed, re-verify the address to resume.

## Notifications are for people

For **systems**, use webhooks instead: signed, retried, machine-readable
deliveries on the same six events. See
[API keys and webhooks](api-keys-and-webhooks.md).

A good split is webhooks driving your automation, and email telling your
operations team when something needs a human.

## Signed in with an API key?

Notification preferences are unavailable in that mode. An API key identifies your
**organization**, and an organization has no inbox.

Sign in with a [magic link](../getting-started/signing-in.md) to set them.
