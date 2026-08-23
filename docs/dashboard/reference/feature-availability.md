# What's available

Some pages in the dashboard show an explanation instead of the feature. This page
explains why, and how to read what you see.

## Why a feature might not be available

Genifi's rail is assembled from several connections — wallet providers, issuers,
banking partners, local payout partners. Not every environment has every one
connected.

Where a connection is missing, the dashboard has two options: pretend, or say so.
It says so.

## What that looks like

Instead of the feature, you get a panel with:

* **what the feature is**, in plain language
* **why it isn't available here**
* **what it needs** in order to work

No dead buttons, no spinner that never resolves, and — importantly — no screen of
plausible-looking numbers that are not real.

## The availability levels

| Level | What you can do |
|---|---|
| **Live** | Works now, fully |
| **Partial** | You can **see** things — the accounts registered to you. The actions that move value need a provider connection |
| **Local** | Handled in your browser; no backend involved |
| **Planned** | Not built yet. Shown as unavailable rather than hidden, so you know it's coming |

## Typically live

* Sending and tracking payments
* Transaction history and timelines
* The address book
* API keys and webhooks
* Team management
* Exports and receipts

## Typically partial

**Wallets** — the list of your registered wallets always works. Balances need a
wallet provider connection.

**Mint accounts** — your registered issuer accounts always list. Mint and redeem
need an issuer connection.

**Deposits & withdrawals** — your registered custody accounts always list.
Balances and wires need a banking connection.

The pattern is consistent: **reads of Genifi's own records work everywhere;
actions that reach out to a provider need that provider connected.**

## Why listings work without a provider

Wallets, mint accounts, and custody accounts are held in Genifi's own account
registry. That is what makes them list with nothing else connected — and also why
they carry no balances. A balance is a report from a provider, and without the
provider there is no report.

## "Nothing configured" versus "nothing there"

The dashboard never renders those two the same way.

An empty list because no provider is connected, and an empty list because you
genuinely hold nothing, are different facts. Conflating them is how somebody
concludes their money has vanished.

If a balance cannot be shown, the dashboard tells you it cannot be shown, and
why.

## The connection banner

Separate from feature availability, the banner at the top reports the connection:

| Banner | Meaning |
|---|---|
| **Connected** | Normal |
| **Degraded** | Reachable, but new payments cannot start right now. Existing payments are unaffected |
| **Unreachable** | The dashboard cannot reach the backend |

It refreshes on its own — no need to reload to see it recover.

## In demo mode, everything works

[Demo mode](../getting-started/demo-mode.md) has no gating, because there is no
real provider to be missing. Every page is fully interactive with sample data.

It is the right way to see what a feature does before it is available on your
environment.

## Getting a feature enabled

Availability follows your environment and your commercial arrangement — it is not
a toggle in the dashboard. Talk to your Genifi account contact about what you
need, and it will appear when the connection behind it is live for you.
