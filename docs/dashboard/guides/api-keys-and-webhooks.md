# API keys and webhooks

The **API keys & webhooks** page holds the two credentials your systems use.

## Your API key

One key per organization. It is the server-to-server credential your backend
sends on every API call, and it authenticates as your **whole organization**.

!!! danger

    An API key can move money. Keep it on your servers. Never put it in a browser
    bundle, a mobile app, a public repository, or a support ticket.

### Rotating

Only the **owner** can rotate the organization key.

Rotation is atomic: a new key is minted and the old one dies at the same moment.

!!! warning

    The new key is shown **once**, and there is no way to read it back later.

    The old key — including the one your integration is using right now — stops
    working immediately. There is no overlap window. Have somewhere to put the new
    key before you click, and expect a gap unless you can deploy it instantly.

Rotate when someone with access leaves, when a key may have been exposed, or on
whatever schedule your security policy sets.

### Personal keys

If you signed in as a person, you can mint and rotate a key scoped to **you**,
carrying your own role. Useful when you want per-person attribution and
per-person revocation rather than one shared organization key.

The organization root key and personal keys are rotated separately, from different
places.

## Webhooks

Rather than having your systems poll for payment status, register one endpoint
and Genifi pushes every state change to you.

Managing webhooks needs the `admin` role or higher.

### Registering

Enter the HTTPS URL your systems listen on and register it.

You get back a **signing secret**, shown once.

!!! danger

    Store the signing secret immediately. It is never displayed again, and there is
    no way to read it back — a read path would turn a read-only credential leak into
    the ability to forge deliveries to your own system.

### One endpoint at a time

Your organization has exactly one webhook endpoint. Registering a new URL
**replaces** the previous one and mints a **fresh** secret — the old secret stops
working straight away.

### URL rules

| Rule | Why |
|---|---|
| Must be `https` | Deliveries carry transaction data and a signature. Over plain HTTP an observer sees both |
| No private or internal addresses | `localhost`, `127.*`, `10.*`, `192.168.*`, `169.254.*`, `.internal`, `.local` are refused |

The second rule exists because a webhook target is chosen by the caller. Without
it, a URL could be used to make Genifi issue requests inside its own network, on
a schedule, with retries.

### What gets delivered

A signed `POST` on every transaction transition:

| Event | Fires when |
|---|---|
| `transaction.quoted` | A price is locked |
| `transaction.funded` | Funds have left your wallet |
| `transaction.paid_out` | The recipient has been paid |
| `transaction.reconciled` | Everything agrees — the payment is done |
| `transaction.refunded` | An unwind completed |
| `transaction.in_doubt` | The outcome is ambiguous and parked |

Deliveries are **at-least-once**, retried with backoff, and dead-lettered rather
than dropped if your endpoint stays unreachable. Each delivery carries a stable
delivery ID, so a retry can be recognised and ignored.

Your engineers should verify the signature on every delivery before acting on it.
[Verifying signatures](../../backend/webhooks/verifying.md) covers the exact
scheme with working code.

### Checking what is registered

The page shows the URL currently registered, or that none is. It never shows the
secret.

### Turning it off

Deleting the endpoint stops delivery and destroys the signing secret. Deliveries
already queued are not cancelled — they fail and dead-letter rather than
disappearing, so nothing is silently lost.

Deleting is confirmed behind a dialog, because it destroys a secret that cannot
be recovered.

## Rotating a signing secret

Register the same URL again. A fresh secret comes back and the old one dies
immediately.

There is no overlap window, so if you cannot tolerate rejected deliveries, deploy
a receiver that accepts either secret first, rotate, then drop the old one.
