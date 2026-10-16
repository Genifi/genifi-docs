# Recipients

A saved address book of payees, scoped to your organization. Save a recipient
once and reuse the routing detail instead of re-sending it on every payment.

Reading recipients needs `recipient.read` (every role has it). Creating,
updating, and deleting need `recipient.manage`: `member` and above, the same
floor as initiating a payment, because you save a payee in order to pay it.

## GET /recipients

=== "Request"

    ```bash
    curl "$JET_API/recipients" -H "Authorization: Bearer $JET_API_KEY"
    ```

=== "200 OK"

    ```json
    {
      "recipients": [
        {
          "id": "rcp_71fa",
          "name": "NGUYEN VAN A",
          "type": "bank",
          "account": "0123456789",
          "bankCode": null,
          "bic": null,
          "bankName": "Vietcombank",
          "country": "VN",
          "city": "Ho Chi Minh City",
          "addressLine": null,
          "state": null,
          "postalCode": null,
          "phoneCallingCode": null,
          "phoneNumber": null,
          "address": null,
          "networkId": null,
          "lastUsedAt": "2026-08-19T11:02:00.000Z",
          "createdAt": "2026-07-30T08:41:00.000Z",
          "updatedAt": "2026-07-30T08:41:00.000Z"
        }
      ],
      "count": 1
    }
    ```

Unset optional fields come back as `null`, not omitted.

## POST /recipients

=== "Bank recipient"

    ```bash
    curl -X POST "$JET_API/recipients" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "NGUYEN VAN A",
        "type": "bank",
        "account": "0123456789",
        "bankName": "Vietcombank",
        "country": "VN"
      }'
    ```

=== "Wallet recipient"

    ```bash
    curl -X POST "$JET_API/recipients" \
      -H "Authorization: Bearer $JET_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "name": "Treasury (Base)",
        "type": "wallet",
        "address": "0x…",
        "networkId": "base"
      }'
    ```

Returns `201` with the created recipient.

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `name` | string(1–200) | yes | |
| `type` | `"bank"` or `"wallet"` | yes | Fixed at creation; cannot be changed later |
| `account` | string(1–140) | for `bank` | The destination account |
| `address` | string(1–200) | for `wallet` | The destination address |
| `bankCode` | string(1–40) | no | |
| `bic` | string(8–11) | no | |
| `bankName` | string(1–200) | no | |
| `country` | string(2) | no | ISO country code |
| `city` | string(1–120) | no | |
| `addressLine` | string(1–280) | no | |
| `state` | string(1–100) | no | Some corridors need a full postal address: `addressLine`, `city`, `state`, `postalCode` |
| `postalCode` | string(1–20) | no | |
| `phoneCallingCode` | 1–4 digits | no | Country calling code. A leading `+` is accepted and removed |
| `phoneNumber` | 4–15 digits | no | No spaces or separators. Some corridors require a phone |
| `networkId` | string(1–60) | no | Chain identifier for a wallet recipient |

Every field, with its exact rules, is in the
[API reference](../api-reference/recipients.md#post-recipients). A saved
recipient can be used in a [batch](batches.md) by its `id`.

A `bank` recipient must have an `account`; a `wallet` recipient must have an
`address`. Missing the type's primary handle is a `400`.

## PATCH /recipients/:id

Patches only the fields you send. Pass `null` to clear a nullable field.

```bash
curl -X PATCH "$JET_API/recipients/rcp_71fa" \
  -H "Authorization: Bearer $JET_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "city": "Hanoi", "bankCode": null }'
```

`type` cannot be patched. A recipient that changes from a bank account to a
wallet is a different payee: create a new one.

## DELETE /recipients/:id

```bash
curl -X DELETE "$JET_API/recipients/rcp_71fa" \
  -H "Authorization: Bearer $JET_API_KEY"
```

Returns `200` with the deleted recipient, so you can log exactly what was
removed.

## Recipients and payments

Saved recipients are a convenience for your systems and for the dashboard. A
single payment (`POST /transactions`) does not take a recipient ID: it carries
its own `beneficiary` object. A [batch](batches.md) does: each item can name a
saved recipient with `recipientId`, resolved on our side.

Read the recipient, then copy its fields onto the payment:

```ts
// There is no GET /recipients/:id: read the list and pick the one you want.
const { recipients } = await listRecipients();
const recipient = recipients.find((r) => r.id === "rcp_71fa");

await initiatePayment({
  idempotencyKey: "pay-2026-08-21-0002",
  corridor: { to: "VND", destinationCountry: "VN" },
  sendAmount: { amount: "500.00", currency: "USDC" },
  recipientReference: recipient.id,
  beneficiary: {
    name: recipient.name,
    account: recipient.account,
    bankName: recipient.bankName,
    country: recipient.country,
  },
});
```

Keeping them separate is deliberate: a payment records the destination it
actually used, so editing a saved recipient afterwards never rewrites the history
of a payment that was already sent.
