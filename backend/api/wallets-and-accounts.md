# Wallets and accounts

Genifi keeps a registry of the accounts your organization owns — the wallets it
funds payments from, plus any custody and issuer accounts. Registration is what
lets you name a wallet in a payment and lets us verify it is yours before moving
anything.

## Why balances are separate from listings

Account listings deliberately carry **no balances**.

Genifi's ledger is the source of truth for value. A wallet's on-chain balance and
a provider's reported balance are *reports* — they are reconciled against the
ledger, never treated as authoritative. Mixing a report into a listing would
invite you to read it as a fact.

So balances live on their own endpoints, and every balance we return is stamped
with `reportedAt` and `source`, so you can never mistake a report for a
timeless truth.

## GET /wallets

Your registered wallets.

{% tabs %}
{% tab title="Request" %}
```bash
curl "$GENIFI_API/wallets" -H "Authorization: Bearer $GENIFI_API_KEY"
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "accounts": [
    {
      "id": "acc_9c1a",
      "kind": "wallet",
      "provider": "external",
      "externalId": "wal_your_wallet",
      "label": "USDC pay-in",
      "currencies": ["USDC"],
      "address": "0x…",
      "status": "active",
      "createdAt": "2026-07-02T09:14:00.000Z"
    }
  ],
  "count": 1
}
```
{% endtab %}
{% endtabs %}

| Field | Type | Notes |
|---|---|---|
| `id` | string | Genifi's internal ID for the registration |
| `kind` | `"wallet"`, `"bank_account"`, `"mint_account"` | |
| `provider` | string | Where the account lives. `external` means you manage it |
| `externalId` | string | **This is what you pass as `integratorWalletId`** |
| `label` | string | Your own label, typically the pay-in currency |
| `currencies` | string[] | Assets this account holds |
| `address` | string or null | On-chain address, where applicable |
| `status` | string | `active` when usable |
| `createdAt` | ISO 8601 | |

{% hint style="info" %}
`externalId`, not `id`, is the value a payment takes. `id` is our registration
record; `externalId` is the wallet itself.
{% endhint %}

## GET /wallets/:walletId

One wallet. Accepts either the `externalId` or the `id`. Same shape as a listing
entry.

A wallet you do not own returns `404`, not `403` — see
[Conventions](conventions.md#404-also-means-not-yours).

## POST /wallets

Register a wallet you own so it can fund payments. You may register several per
pay-in currency and choose between them at payment time.

{% tabs %}
{% tab title="Request" %}
```bash
curl -X POST "$GENIFI_API/wallets" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "externalId": "wal_treasury_eu",
    "label": "USDC pay-in (EU)",
    "currencies": ["USDC"],
    "address": "0x…"
  }'
```
{% endtab %}

{% tab title="201 Created" %}
```json
{
  "id": "acc_2f7b",
  "kind": "wallet",
  "provider": "external",
  "externalId": "wal_treasury_eu",
  "label": "USDC pay-in (EU)",
  "currencies": ["USDC"],
  "address": "0x…",
  "status": "active",
  "createdAt": "2026-08-21T14:20:00.000Z"
}
```
{% endtab %}

{% tab title="409 Conflict" %}
```json
{ "error": "conflict", "message": "this wallet is already registered" }
```
{% endtab %}
{% endtabs %}

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `externalId` | string(1–200) | yes | The wallet's ID at its provider |
| `label` | string(1–60) | yes | Your label |
| `currencies` | string[] | no | Defaults to `["USDC"]` |
| `provider` | string | no | Defaults to `"external"` — a wallet you manage |
| `address` | string(1–200) | no | On-chain address, so an on-ramp can deliver here |

A wallet already registered to another organization returns `409`. There is no
takeover path.

## GET /wallets/:walletId/balances

Provider-reported balances for a wallet you own.

{% tabs %}
{% tab title="Request" %}
```bash
curl "$GENIFI_API/wallets/wal_your_wallet/balances" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "walletId": "wal_your_wallet",
  "balances": [
    {
      "asset": "USDC",
      "amount": "1500000000",
      "scale": 6,
      "settled": true,
      "reportedAt": "2026-08-21T14:22:11.000Z"
    }
  ]
}
```
{% endtab %}

{% tab title="501 Not wired" %}
```json
{
  "error": "not_implemented",
  "feature": "wallets",
  "message": "no wallet provider is configured on this environment",
  "requires": "a wallet provider adapter"
}
```
{% endtab %}
{% endtabs %}

`amount` is in integer minor units at `scale` — see
[Amounts and currencies](../concepts/money.md).

A `501` means no provider can answer at all. It is **not** the same as an empty
balance list: "nothing is configured" and "this wallet holds nothing" are
different answers and must not render identically.

## Custody and issuer accounts

Same shape, different `kind`:

```bash
curl "$GENIFI_API/banking/accounts" -H "Authorization: Bearer $GENIFI_API_KEY"
curl "$GENIFI_API/issuers/accounts" -H "Authorization: Bearer $GENIFI_API_KEY"
```

Balances follow the same pattern and carry a `source`:

```bash
curl "$GENIFI_API/banking/accounts/acc_5d20/balances" \
  -H "Authorization: Bearer $GENIFI_API_KEY"

curl "$GENIFI_API/issuers/iss_a1b2/balances" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "accountId": "acc_5d20", "balances": [ … ], "source": "provider_report" }
```

`"source": "provider_report"` is the label doing the work. It is what the
provider says, at `reportedAt`. It is not your balance of record.

## Issuer directory

Distinct from the issuer *accounts* you own — this is the catalog of issuers and
the assets they support.

```bash
curl "$GENIFI_API/issuers" -H "Authorization: Bearer $GENIFI_API_KEY"
curl "$GENIFI_API/issuers/iss_…/assets" -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "issuers": [ … ], "count": 2 }
```

Returns `501` where no issuer is configured, for the same reason as balances.

## Endpoints that return 501 today

These exist in the contract and are gated until their adapters are wired. Check
[`/capabilities`](discovery.md#get-capabilities) rather than calling them
speculatively.

| Endpoint | Purpose |
|---|---|
| `POST /wallets/:walletId/transfers` | Move funds between wallets |
| `POST /issuers/:issuerId/mint` | Mint at par |
| `POST /banking/wire-transfers` | Send a wire from a custody account |

Each returns a typed reason and a `requires` field:

```json
{
  "error": "not_implemented",
  "feature": "banking",
  "message": "wire transfers need a custody bank adapter",
  "requires": "a production banking partner"
}
```
