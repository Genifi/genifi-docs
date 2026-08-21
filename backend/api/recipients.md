# Recipients

A saved address book of payees, scoped to your organization. Save a recipient
once and reuse the routing detail instead of re-sending it on every payment.

Reading recipients needs `recipient.read` (every role has it). Creating,
updating, and deleting need `recipient.manage` — `member` and above, the same
floor as initiating a payment, because you save a payee in order to pay it.

## GET /recipients

{% tabs %}
{% tab title="Request" %}
```bash
curl "$GENIFI_API/recipients" -H "Authorization: Bearer $GENIFI_API_KEY"
```
{% endtab %}

{% tab title="200 OK" %}
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
{% endtab %}
{% endtabs %}

Unset optional fields come back as `null`, not omitted.

## POST /recipients

{% tabs %}
{% tab title="Bank recipient" %}
```bash
curl -X POST "$GENIFI_API/recipients" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "NGUYEN VAN A",
    "type": "bank",
    "account": "0123456789",
    "bankName": "Vietcombank",
    "country": "VN"
  }'
```
{% endtab %}

{% tab title="Wallet recipient" %}
```bash
curl -X POST "$GENIFI_API/recipients" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Treasury (Base)",
    "type": "wallet",
    "address": "0x…",
    "networkId": "base"
  }'
```
{% endtab %}
{% endtabs %}

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
| `networkId` | string(1–60) | no | Chain identifier for a wallet recipient |

A `bank` recipient must have an `account`; a `wallet` recipient must have an
`address`. Missing the type's primary handle is a `400`.

## PATCH /recipients/:id

Patches only the fields you send. Pass `null` to clear a nullable field.

```bash
curl -X PATCH "$GENIFI_API/recipients/rcp_71fa" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "city": "Hanoi", "bankCode": null }'
```

`type` cannot be patched. A recipient that changes from a bank account to a
wallet is a different payee — create a new one.

## DELETE /recipients/:id

```bash
curl -X DELETE "$GENIFI_API/recipients/rcp_71fa" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

Returns `200` with the deleted recipient, so you can log exactly what was
removed.

## Recipients and payments

Saved recipients are a convenience for your systems and for the dashboard. They
are **not** wired into `POST /transactions` — a payment still carries its own
`beneficiary` object.

Read the recipient, then copy its fields onto the payment:

```ts
const recipient = await getRecipient("rcp_71fa");

await initiatePayment({
  idempotencyKey: "pay-2026-08-21-0002",
  corridor: { to: "VND", destinationCountry: "VN" },
  sendAmount: { amount: "500.00", currency: "USDC" },
  integratorWalletId: "wal_your_wallet",
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
