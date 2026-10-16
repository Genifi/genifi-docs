# Notifications

Email notifications on transaction events, configured **per person**.

!!! info

    These are the only endpoints that need a *person*, not an organization. They are
    authenticated by a member credential: either a signed-in dashboard session or a
    personal member API key. Calling them with the organization root key returns
    `409 member_required`, because an organization has no inbox.

    For machine-consumable events, use [webhooks](../webhooks/README.md) instead.

## GET /notifications/preferences

=== "Request"

    ```bash
    curl "$JET_API/notifications/preferences" \
      -H "Authorization: Bearer $JET_MEMBER_KEY"
    ```

=== "200 OK"

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

=== "Suppressed"

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

| Field | Type | Notes |
|---|---|---|
| `events` | string[] | What this member has opted into |
| `availableEvents` | string[] | Everything that can be subscribed to |
| `emailVerified` | boolean | Unverified addresses are not emailed |
| `email` | string | The address mail would go to |
| `verificationAvailable` | boolean | Whether this environment can send a verification link |
| `suppressedAt` | ISO 8601 or absent | Present only when sending has been stopped |
| `suppressionReason` | string or absent | Why: e.g. a bounce |
| `updatedAt` | ISO 8601 | |

`suppressedAt` is surfaced so a client can say *why* mail stopped. "Your address
bounced" is actionable; silently sending nothing is not.

## PUT /notifications/preferences

Replaces the subscription set. This is a **full replacement**, not a patch: send
the complete list of events you want.

=== "Request"

    ```bash
    curl -X PUT "$JET_API/notifications/preferences" \
      -H "Authorization: Bearer $JET_MEMBER_KEY" \
      -H "Content-Type: application/json" \
      -d '{ "events": ["transaction.paid_out", "transaction.refunded", "transaction.in_doubt"] }'
    ```

=== "Unsubscribe from all"

    ```bash
    curl -X PUT "$JET_API/notifications/preferences" \
      -H "Authorization: Bearer $JET_MEMBER_KEY" \
      -H "Content-Type: application/json" \
      -d '{ "events": [] }'
    ```

Returns the updated preferences, in the same shape as the `GET`.

An empty array is valid and means "email me nothing".

## Available events

A subset of the [webhook events](../webhooks/events.md), the ones a person acts
on. `availableEvents` in the response is the authoritative list:

| Event | Fires when | On by default |
|---|---|:---:|
| `transaction.paid_out` | The recipient has been paid | yes |
| `transaction.reconciled` | Everything agrees: the payment is done | no |
| `transaction.refunded` | The payment was cancelled and the money returned | yes |
| `transaction.in_doubt` | The outcome is ambiguous and parked | yes |
| `onramp.delivered` | A deposit arrived in your wallet | yes |
| `onramp.refunded` | A deposit was returned to the payer | yes |
| `offramp.paid` | An off-ramp paid its beneficiary | yes |

A member who has never set preferences gets the defaults. `reconciled` is
opt-in, so nobody is told twice about every payment.

## POST /notifications/verify/send

Sends a confirmation link to the member's own address.

```bash
curl -X POST "$JET_API/notifications/verify/send" \
  -H "Authorization: Bearer $JET_MEMBER_KEY"
```

```json
{ "sent": true, "email": "ana@northwind.example", "expiresAt": "2026-08-21T16:31:00.000Z" }
```

`202` means the mail has been handed to the sending service, which is not the
same as delivered.

!!! warning

    You cannot name an address to verify. The address comes from the authenticated
    member's own record. Accepting one from the body would make this an open relay
    for sending JET-branded mail to strangers.

### Errors

| Code | Error | Meaning |
|---|---|---|
| `409` | `member_required` | Authenticated as the organization, not a person |
| `409` | `no_member_email` | This member has no address on file |
| `501` | `not_implemented` | Verification is not configured on this environment |
