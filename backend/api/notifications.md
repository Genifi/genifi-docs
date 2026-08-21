# Notifications

Email notifications on transaction events, configured **per person**.

{% hint style="info" %}
These are the only endpoints that need a *person*, not an organization. They are
authenticated by a member credential — either a signed-in dashboard session or a
personal member API key. Calling them with the organization root key returns
`409 member_required`, because an organization has no inbox.

For machine-consumable events, use [webhooks](../webhooks/README.md) instead.
{% endhint %}

## GET /notifications/preferences

{% tabs %}
{% tab title="Request" %}
```bash
curl "$GENIFI_API/notifications/preferences" \
  -H "Authorization: Bearer $GENIFI_MEMBER_KEY"
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "events": ["transaction.paid_out", "transaction.in_doubt"],
  "availableEvents": [
    "transaction.quoted",
    "transaction.funded",
    "transaction.paid_out",
    "transaction.reconciled",
    "transaction.refunded",
    "transaction.in_doubt"
  ],
  "emailVerified": true,
  "email": "ana@northwind.example",
  "verificationAvailable": true,
  "updatedAt": "2026-08-12T09:30:00.000Z"
}
```
{% endtab %}

{% tab title="Suppressed" %}
```json
{
  "events": ["transaction.paid_out"],
  "availableEvents": [ "…" ],
  "emailVerified": true,
  "suppressedAt": "2026-08-18T22:14:00.000Z",
  "suppressionReason": "hard_bounce",
  "updatedAt": "2026-08-12T09:30:00.000Z"
}
```
{% endtab %}
{% endtabs %}

| Field | Type | Notes |
|---|---|---|
| `events` | string[] | What this member has opted into |
| `availableEvents` | string[] | Everything that can be subscribed to |
| `emailVerified` | boolean | Unverified addresses are not emailed |
| `email` | string | The address mail would go to |
| `verificationAvailable` | boolean | Whether this environment can send a verification link |
| `suppressedAt` | ISO 8601 or absent | Present only when sending has been stopped |
| `suppressionReason` | string or absent | Why — e.g. a bounce |
| `updatedAt` | ISO 8601 | |

`suppressedAt` is surfaced so a client can say *why* mail stopped. "Your address
bounced" is actionable; silently sending nothing is not.

## PUT /notifications/preferences

Replaces the subscription set. This is a **full replacement**, not a patch — send
the complete list of events you want.

{% tabs %}
{% tab title="Request" %}
```bash
curl -X PUT "$GENIFI_API/notifications/preferences" \
  -H "Authorization: Bearer $GENIFI_MEMBER_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "events": ["transaction.paid_out", "transaction.refunded", "transaction.in_doubt"] }'
```
{% endtab %}

{% tab title="Unsubscribe from all" %}
```bash
curl -X PUT "$GENIFI_API/notifications/preferences" \
  -H "Authorization: Bearer $GENIFI_MEMBER_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "events": [] }'
```
{% endtab %}
{% endtabs %}

Returns the updated preferences, in the same shape as the `GET`.

An empty array is valid and means "email me nothing".

## Available events

The same six transitions webhooks carry:

| Event | Fires when |
|---|---|
| `transaction.quoted` | A price is locked |
| `transaction.funded` | Funds have left your wallet |
| `transaction.paid_out` | The recipient has been paid |
| `transaction.reconciled` | Everything agrees — the payment is done |
| `transaction.refunded` | An unwind completed |
| `transaction.in_doubt` | The outcome is ambiguous and parked |

A member who has never set preferences is subscribed to the important
transitions only. The chatty intermediate ones are opt-in, so a new colleague is
not flooded on their first day.

## POST /notifications/verify/send

Sends a confirmation link to the member's own address.

```bash
curl -X POST "$GENIFI_API/notifications/verify/send" \
  -H "Authorization: Bearer $GENIFI_MEMBER_KEY"
```

```json
{ "sent": true, "email": "ana@northwind.example", "expiresAt": "2026-08-21T16:31:00.000Z" }
```

`202` means the mail has been handed to the sending service — which is not the
same as delivered.

{% hint style="warning" %}
You cannot name an address to verify. The address comes from the authenticated
member's own record. Accepting one from the body would make this an open relay
for sending Genifi-branded mail to strangers.
{% endhint %}

### Errors

| Code | Error | Meaning |
|---|---|---|
| `409` | `member_required` | Authenticated as the organization, not a person |
| `409` | `no_member_email` | This member has no address on file |
| `501` | `not_implemented` | Verification is not configured on this environment |
