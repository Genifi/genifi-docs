# A tour of the dashboard

Every page, what it is for, and where to read more.

## Overview

`/dashboard`

Your landing page: whether the backend is reachable, what this environment can
do, and your most recent payments. If something is wrong with the connection,
this is where you find out first.

## Payments

`/dashboard/payments`

Where you send money. Pick a corridor, enter an amount, name a recipient, and
watch the payment run.

→ [Send a payment](../guides/send-a-payment.md)

## Transactions

`/dashboard/transactions`

Every payment your organization has initiated, with live status. Payments you
started in this browser appear immediately, merged with the full history from the
backend — one row per payment, no duplicates.

→ [Track transactions](../guides/track-transactions.md)

## Payouts

`/dashboard/payouts`

The same payments viewed from the settlement side: what has been paid out, what
is still settling, what needs attention.

## Wallets

`/dashboard/wallets`

The wallets registered to your organization — the pots payments are funded from.
Balances appear where a provider is wired on your environment.

→ [Wallets and funding](../guides/wallets-and-funding.md)

## Mint accounts

`/dashboard/mint-accounts`

Your registered issuer accounts, used to convert stablecoins and fiat at par.
Listing works; the mint and redeem actions need a provider connection.

## Deposits & withdrawals

`/dashboard/deposits`

Registered custody accounts — where fiat arrives and leaves. Balances and wire
actions depend on a banking connection being wired on your environment.

## Address book

`/dashboard/address-book`

Saved recipients, so you don't retype routing detail on every payment.

→ [The address book](../guides/address-book.md)

## API keys & webhooks

`/dashboard/api-keys`

Your organization's API key and its rotation, plus the one webhook endpoint your
systems receive events on.

→ [API keys and webhooks](../guides/api-keys-and-webhooks.md)

## Team

`/dashboard/team`

Invite colleagues, set and change roles, revoke access.

→ [Your team](../guides/team-and-roles.md)

## Settings

`/dashboard/settings`

Your own preferences — including which transaction events you want emailed.

→ [Email notifications](../guides/notifications.md)

## Reading the banner

A strip at the top of the app tells you what you are looking at.

| Banner | Meaning |
|---|---|
| **Demo mode** | Everything on screen is sample data. Nothing is real |
| **Connected** | The dashboard is talking to the backend normally |
| **Degraded** | The backend is reachable but cannot start new payments right now |
| **Unreachable** | The dashboard cannot reach the backend at all |

The connection state refreshes on its own — you don't need to reload to see it
recover.

## Pages that may be gated

Depending on what your environment has wired up, some pages show an explanation
instead of the feature. That is intentional, and the panel tells you what the
feature is and what it needs.

→ [What's available](../reference/feature-availability.md)
