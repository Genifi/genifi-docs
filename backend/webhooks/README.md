---
description: Signed, retried, at-least-once notifications on every transaction transition.
---

# Webhooks

Instead of polling every payment, register one endpoint and Genifi pushes each
state change to you — signed, with retries and a dead-letter path.

One endpoint per organization. Registering a new URL **replaces** the old one.

## Register an endpoint

Requires `webhook.manage` (admin or owner).

{% tabs %}
{% tab title="Request" %}
```bash
curl -X PUT "$GENIFI_API/webhooks/endpoint" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "url": "https://hooks.northwind.example/genifi" }'
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "url": "https://hooks.northwind.example/genifi",
  "secret": "whsec_kR3n…"
}
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**The `secret` is shown once.** There is no read path back to it — a `GET` that
returned it would turn a read-scoped credential leak into a forgeable-delivery
leak. Store it before you do anything else.

Registering again mints a **fresh** secret and invalidates the old one.
{% endhint %}

### URL requirements

| Rule | Why |
|---|---|
| `https` required | Deliveries carry transaction data and an HMAC. Over plain HTTP, an observer sees the data and a replayer has everything needed to resend it |
| No loopback, link-local, or private addresses | `localhost`, `127.*`, `169.254.*`, `10.*`, `192.168.*`, `172.16–31.*`, `.internal`, `.local` are refused |

The private-address rule exists because the webhook target is attacker-chosen:
any authenticated caller could otherwise name a URL that makes us issue requests
from inside our own network, on a schedule, with retries.

```json
{
  "error": "validation",
  "issues": [{ "message": "webhook url must not target a loopback, link-local, or private address" }]
}
```

## Read the registered endpoint

```bash
curl "$GENIFI_API/webhooks/endpoint" -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "url": "https://hooks.northwind.example/genifi" }
```

`{ "url": null }` when nothing is registered — that is a normal state of a
resource that exists, not a `404`. The secret is never returned.

## Delete the endpoint

```bash
curl -X DELETE "$GENIFI_API/webhooks/endpoint" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "url": null, "deleted": true }
```

Delivery stops and the signing secret is destroyed. The call is idempotent:
deleting when nothing is registered returns `200` with `deleted: false`.

{% hint style="info" %}
Deliveries **already queued** are not cancelled. They stay durable and fail with
"no endpoint configured" until they dead-letter. A delete that silently dropped
queued work would be the one way this system could lose a notification.
{% endhint %}

## What a delivery looks like

```http
POST /genifi HTTP/1.1
Host: hooks.northwind.example
Content-Type: application/json
X-Genifi-Signature: sha256=9f2c…
X-Genifi-Timestamp: 1787320991
X-Genifi-Delivery-Id: dlv_6ac1…
X-Genifi-Event: transaction.reconciled
```

```json
{
  "deliveryId": "dlv_6ac1…",
  "event": "transaction.reconciled",
  "transactionId": "pay-2026-08-21-0001",
  "data": {
    "state": "RECONCILED",
    "receiveAmount": { "amount": "24512750", "currency": "VND" }
  }
}
```

| Header | Meaning |
|---|---|
| `X-Genifi-Signature` | `sha256=<hex>` HMAC over `{timestamp}.{rawBody}` |
| `X-Genifi-Timestamp` | Unix **seconds**, bound into the signature |
| `X-Genifi-Delivery-Id` | Stable across retries of the same delivery |
| `X-Genifi-Event` | The event type, also present in the body |

**Always verify the signature before trusting a delivery.** See
[Verifying signatures](verifying.md).

## Responding

Return any `2xx` and we mark the delivery done. Anything else — or a timeout — is
a failed attempt.

Respond fast and do your work asynchronously. A slow endpoint turns into retries,
and retries turn into duplicate processing on your side.

## Delivery guarantees

**At-least-once.** A delivery may arrive more than once — after a network blip,
or a retry of a request we never saw succeed.

**De-duplicate on `deliveryId`.** A retry carries the *same* `deliveryId`, so
recording IDs you have processed turns at-least-once into effectively
exactly-once on your side.

**Retries back off exponentially** — roughly 1s, 2s, 4s, 8s, and so on — up to a
small number of attempts, then the delivery **dead-letters** rather than being
dropped. Nothing is silently lost.

**Each retry is signed afresh** with a new timestamp, so your staleness window
never rejects our own legitimate retries.

**Ordering is not guaranteed.** Two events for the same payment can arrive out of
order. Use the `state` in the payload as the truth, and if you need certainty,
read `GET /transactions/:id`.

## A robust receiver

```
1. Read the raw body — bytes, before any parsing
2. Verify the HMAC in constant time
3. Check the timestamp is within your tolerance (5 minutes is typical)
4. Have I seen this deliveryId? → return 200, do nothing
5. Record the deliveryId
6. Return 200
7. Process asynchronously
```

Steps 1–3 are non-negotiable. Steps 4–7 are what keep a retry storm from
becoming a double-processing incident.

## Next

* [Event types](events.md) — every event and when it fires
* [Verifying signatures](verifying.md) — with working code
