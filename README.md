---
description: >-
  Genifi is a cross-border payment rail. You send a stablecoin amount; your
  recipient is paid in their local currency.
---

# Genifi Documentation

Genifi moves value across borders for regulated businesses. Value travels as a
stablecoin and lands as **local fiat**, disbursed by a licensed partner in the
destination country. The recipient never holds, receives, or needs to know about
a stablecoin.

You keep the relationship with your end users and you own their KYC. Genifi
verifies you, and provides the corridor.

## Two ways to use it

<table data-card-size="large" data-view="cards">
  <thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead>
  <tbody>
    <tr>
      <td><strong>Backend API</strong></td>
      <td>Integrate Genifi into your own systems. Quotes, payments, webhooks, and the full endpoint reference.</td>
      <td><a href="backend/README.md">backend/README.md</a></td>
    </tr>
    <tr>
      <td><strong>Dashboard</strong></td>
      <td>Send and track payments from a web app. No code required.</td>
      <td><a href="dashboard/README.md">dashboard/README.md</a></td>
    </tr>
  </tbody>
</table>

## Start here

**Building an integration?** [Quickstart](backend/getting-started/quickstart.md)
takes you from an API key to a settled payment in six steps. Then read
[Idempotency](backend/concepts/idempotency.md) and
[How a payment works](backend/concepts/payment-lifecycle.md) before you write
production code — they cover the two things integrations most often get wrong.

**Using the dashboard?** [Signing in](dashboard/getting-started/signing-in.md),
then [Send a payment](dashboard/guides/send-a-payment.md). Want to look around
first, with no account? [Demo mode](dashboard/getting-started/demo-mode.md).

## What to expect

A few properties run through everything Genifi does. They explain choices that
might otherwise look strange.

**Money is never a JSON number.** You send decimal strings; we return integer
minor units. A float cannot carry a six-decimal stablecoin amount without losing
precision, and losing precision on money is not an acceptable failure mode.
→ [Amounts and currencies](backend/concepts/money.md)

**Retries are always safe.** Your idempotency key *is* the transaction ID. Send
the same request ten times and there is one payment.
→ [Idempotency](backend/concepts/idempotency.md)

**We fail closed and never fake success.** If we cannot tell whether funds moved,
the payment is parked for reconciliation rather than optimistically marked
complete. If a feature is not wired up on your environment, you get a `501` with
a reason — not an empty list, not a plausible stub.
→ [Environments](backend/getting-started/environments.md)

**A payment is done when it is reconciled.** `Paid out` means the recipient has
the money. `Reconciled` means the books, the chain, and the provider all agree.
Tell your customer at the first; close your ledger at the second.
→ [How a payment works](backend/concepts/payment-lifecycle.md)

**The recipient always receives local fiat.** In several destination markets,
using a stablecoin as a means of payment is prohibited. The stablecoin leg stays
offshore, always.
→ [Corridors](backend/concepts/corridors.md)
