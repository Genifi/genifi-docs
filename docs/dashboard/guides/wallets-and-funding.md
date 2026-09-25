# Wallets and funding

Payments are funded from a **wallet** registered to your organization. The
Wallets page is where you see them.

## What a registered wallet is

A wallet you control, recorded with Genifi so that:

* you can select it when sending a payment, and
* Genifi can verify it is yours before moving anything out of it.

You may register more than one — a common pattern is one per pay-in currency, or
one per business line — and choose between them at payment time.

Each wallet shows its label, the assets it holds, its on-chain address where it
has one, and its status.

## Why listings don't show balances

The wallet list deliberately shows **no balances**.

Genifi's ledger is the source of truth for value. A wallet's on-chain balance and
a provider's reported balance are *reports* — they are reconciled against the
ledger, not treated as authoritative. Putting a report in a list invites you to
read it as a fact.

Balances therefore appear on their own, labelled with **who reported them and
when**:

> USDC 1,500.00 · reported by provider at 14:22

That label is doing real work. It is what a provider said at a moment in time.

## When balances are unavailable

On some environments no provider is connected, and the dashboard says so rather
than showing a blank.

"No provider is connected" and "this wallet holds nothing" are different answers,
and the dashboard never renders them identically. If it cannot tell you a
balance, it tells you why.

## Funding a wallet

How value gets into a wallet depends on your arrangement:

* **Transfer in directly** — send the stablecoin to the wallet's on-chain
  address, shown on the wallet's detail.
* **On-ramp fiat** — wire USD or EUR and have it delivered as a stablecoin to
  that address. **Deposits & withdrawals → Your deposit details** shows where to
  send it: Genifi's bank details for each currency, and the **reference to
  quote**, with a copy button on every field. The same details come back when you
  start an on-ramp, through the dashboard or the API.

!!! warning "Always quote the reference"

    Every customer wires to the same Genifi account, so the reference is the only
    thing that says the money is yours. A wire without it — or with a mangled one
    — arrives unattributed and needs manual reconciliation; it is never matched on
    amount alone.

    One reference covers every currency: it identifies your organization, not the
    transfer. But the **bank accounts are per currency and are not
    interchangeable** — send USD to the USD account and EUR to the EUR one.

    Your deposit details appear once your business verification is approved.

## Not enough in the wallet?

When balances are available, the payment form warns you before you spend a quote:

> You're sending more than this wallet holds — balance is 800.00 USDC. Reduce the
> amount or top up the wallet.

Where balances are not available, the backend is still the authority and the
payment fails cleanly at the funding step, before value moves onward.

## Mint accounts, deposits and withdrawals

Two more pages appear in the sidebar:

**Mint accounts** (`/dashboard/mint-accounts`) — issuer accounts used to convert
between a stablecoin and fiat at par, rather than swapping on a market and paying
slippage. Listing works everywhere; the mint and redeem actions need a provider
connection.

**Deposits & withdrawals** (`/dashboard/deposits`) — your deposit details, and
every fiat deposit tracked through to delivered USDC. Withdrawals need a banking
connection.

!!! info "There is no bank account of your own to list here"

    You never link a bank account to Genifi or to any provider — a compliance
    decision, not a missing feature. Fiat reaches you through **Genifi's own
    account**, which is what "Your deposit details" describes. So this page shows
    where to send money and what has arrived, not a list of accounts you hold.

Where an action is not available, the dashboard shows what it is and what it
needs — see [What's available](../reference/feature-availability.md).

## Adding a wallet

Registering a wallet records its provider ID, a label, the assets it holds, and
its address.

A wallet already registered to another organization cannot be claimed. There is
no takeover path — if you believe a wallet is yours and it is registered
elsewhere, that is a support conversation, not a UI action.
