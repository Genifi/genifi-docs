# Demo mode

Demo mode is a fully interactive Genifi Dashboard, populated with sample data,
that requires no account, no API key, and no backend.

It is the right way to evaluate the product, run a walkthrough, or explore the
interface before your environment is provisioned.

## Getting in

On the sign-in screen, click **Explore the demo — no API key needed**.

Some deployments — an evaluation link, for instance — are built as demo-only and
open straight into it.

## Getting out

If you opted in from the sign-in screen, the banner at the top offers **Exit
demo**. Clicking it returns you to the normal sign-in screen.

A demo-only build has no exit, because there is nothing behind it to return to.

## What is real in demo mode

Nothing.

More precisely: every screen is populated from sample data held in your browser,
and **the app makes no network calls at all**. No payment is initiated, no
recipient is stored anywhere, no key is valid. Closing the tab discards
everything.

This is not a "sandbox pointed at test servers". It is a self-contained
demonstration with nothing behind it.

## What you can do

All of it. Every page is fully interactive:

* Run the payment wizard end to end, including multi-recipient and bulk flows
* Watch a payment move through its states on a timeline
* Add, edit, and delete recipients
* Browse wallets, mint accounts, and custody accounts with sample balances
* Register a webhook endpoint and see the flow
* Invite team members and change roles
* Export CSVs and generate receipts

Pages that are gated in a live environment are **fully available in demo mode**,
because there is no real provider to be missing.

## Demo artefacts are marked

Anything you can carry out of demo mode says so on its face. A receipt generated
in demo mode is unmistakably a demo receipt; an export is unmistakably a demo
export.

This is deliberate and not adjustable. A document that could be mistaken for a
record of a real payment is a document that will eventually be mistaken for one.

## Using it for a walkthrough

Demo mode is genuinely good for this:

* Nothing you click can cost money or touch a customer
* No credentials to hand out or claw back
* Sample data is stable and sensible, so the story is the same every time
* It works with no network access to a backend

Reset by reloading the page — state lives in the browser session.

## Demo mode and live mode never mix

A given page load is entirely one mode or the other. Switching always does a full
reload, so there is no window where half the screen is real and half is sample
data.

A live session cannot be pushed into demo mode by a URL parameter, and a demo
session never falls back to live data when something fails. The separation runs
in both directions on purpose.

## Ready for the real thing?

You need an invitation to an organization. See
[Signing in](signing-in.md).
