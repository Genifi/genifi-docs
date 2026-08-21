---
description: >-
  The web app for sending, tracking, and managing cross-border payments on
  Genifi — no code required.
---

# Genifi Dashboard

The Genifi Dashboard is where your team sends payments, watches them settle, and
manages the credentials your systems use.

Everything here is backed by the same [Genifi API](../backend/README.md) your
engineers integrate against. The dashboard is a client of that API — it holds no
separate ledger and no separate truth. If a payment shows a state here, that is
the state.

## What you can do

| | |
|---|---|
| **Send a payment** | Pick a corridor, enter an amount, name a recipient. Watch it land. |
| **Track everything** | Live status on every payment, with a full timeline. |
| **Manage recipients** | Save payees to an address book and reuse them. |
| **See your accounts** | Registered wallets, custody accounts, and issuer accounts. |
| **Hold your credentials** | View your API key, rotate it, register a webhook endpoint. |
| **Run your team** | Invite colleagues, set roles, revoke access. |
| **Export** | CSV for a spreadsheet, printable receipts for a customer. |

## Two ways in

**Sign in** with your work email — we send a magic link. Your email has to belong
to an active member of an organization, so somebody has to invite you first. See
[Signing in](getting-started/signing-in.md).

**Explore the demo** with no account at all. Every screen is fully interactive,
populated with sample data, and makes no network calls whatsoever. See
[Demo mode](getting-started/demo-mode.md).

## An honest interface

A few things about this dashboard are unusual, and they are all deliberate.

**Features that are not available say so.** Where a capability is not wired up on
your environment, the dashboard shows a panel explaining what it is and what it
needs — rather than a button that fails, or worse, a screen of plausible-looking
numbers that are not real. See [What's available](reference/feature-availability.md).

**Balances are labelled as reports.** Where you see a provider balance, it is
shown as what a provider said and when. Genifi's ledger is the source of truth
for value; a provider's figure is reconciled against it, not treated as fact.

**Demo data is unmistakable.** Every receipt, export, and screen generated in
demo mode is marked as such. No demo artefact can be mistaken for a real one.

**`Paid out` is not the last step.** A payment is finished when it reaches
`Reconciled`. The dashboard shows the difference rather than rounding it away —
see [Track transactions](guides/track-transactions.md).

## Where to start

* Just been invited? [Signing in](getting-started/signing-in.md).
* Want to look around first? [Demo mode](getting-started/demo-mode.md).
* Ready to send? [Send a payment](guides/send-a-payment.md).
