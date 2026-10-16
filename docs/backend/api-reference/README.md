---
description: "Every endpoint of the JET API, generated from the API source."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# API reference

Every endpoint the API serves, generated from its source code: the routes, the request schemas, the response bodies, the error codes, and who may call each one. The API's test suite fails when an endpoint changes without its generated contract being updated, and this site is regenerated from that same source.

For how the pieces fit together, start with the [guides](../api/conventions.md). The same contract is available as an [OpenAPI 3.1 document](openapi.json) for client generators and API tools.

## Reading an endpoint

- **Auth**: `API key` means `Authorization: Bearer <key>`. See [Authentication](../getting-started/authentication.md).
- **Permission**: the action your role must hold, and which roles hold it. See [Team and API keys](../api/team.md).
- **KYB**: the endpoint moves money, so your organization must have completed verification. Before that it answers `403 kyb_required`.
- **Rate limit**: which bucket the call draws from (below).
- **Unknown fields**: some request bodies reject a field they do not know with `400`; the rest ignore it. Each endpoint says which. Check spelling either way: an ignored field is a silently missing one.
- **Always present**: in a response table, `no` means the field is left out when it does not apply.

## Rate limits

Per organization and per bucket. Every authenticated response carries `ratelimit-limit`, `ratelimit-remaining` and `ratelimit-reset`. A batch costs one `initiate` token per item. Defaults:

| Bucket | Burst | Sustained, per second | Used by |
|---|---|---|---|
| `initiate` | 30 | 5 | `POST /transactions`, `POST /batches` |
| `write` | 60 | 10 | every other authenticated write |
| `read` | 300 | 50 | every authenticated read |

More: [Rate limits](../reference/rate-limits.md).

## Roles and permissions

Each team member has a role. An endpoint that names a permission is refused with `403 forbidden` for a role without it; every role may call an endpoint that names none. `owner` is the organization key itself and is never assigned.

| Permission | owner | admin | member | viewer | Endpoints |
|---|:---:|:---:|:---:|:---:|---|
| `beneficiary.manage` | yes | yes | yes | no | [`POST /treasury/beneficiaries`](withdrawals.md#post-treasurybeneficiaries) |
| `member.invite` | yes | yes | no | no | [`POST /members/invitations`](team.md#post-membersinvitations), [`POST /members/:id/invitations`](team.md#post-membersidinvitations) |
| `member.manage` | yes | yes | no | no | [`PATCH /members/:id`](team.md#patch-membersid), [`DELETE /members/:id`](team.md#delete-membersid) |
| `member.read` | yes | yes | no | no | [`GET /members`](team.md#get-members) |
| `payment.initiate` | yes | yes | yes | no | [`POST /quotes`](payments.md#post-quotes), [`POST /transactions`](payments.md#post-transactions), [`POST /batches`](batches.md#post-batches) |
| `recipient.manage` | yes | yes | yes | no | [`POST /recipients`](recipients.md#post-recipients), [`PATCH /recipients/:id`](recipients.md#patch-recipientsid), [`DELETE /recipients/:id`](recipients.md#delete-recipientsid) |
| `recipient.read` | yes | yes | yes | yes | [`GET /recipients`](recipients.md#get-recipients) |
| `treasury.move` | yes | yes | yes | no | [`POST /treasury/onramp`](deposits.md#post-treasuryonramp), [`POST /treasury/onramp/pix`](deposits.md#post-treasuryonramppix), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals), [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp), [`POST /treasury/onramp/:onRampId/simulate-payment`](sandbox.md#post-treasuryonramponrampidsimulate-payment), [`POST /treasury/onramps/simulate-wire`](sandbox.md#post-treasuryonrampssimulate-wire), [`POST /wallets/:walletId/topup-eurc`](sandbox.md#post-walletswalletidtopup-eurc) |
| `webhook.manage` | yes | yes | no | no | [`PUT /webhooks/endpoint`](webhooks.md#put-webhooksendpoint), [`DELETE /webhooks/endpoint`](webhooks.md#delete-webhooksendpoint) |

## All endpoints

### [Discovery](discovery.md)

| Endpoint | |
|---|---|
| [`GET /health`](discovery.md#get-health) | Liveness check |
| [`GET /capabilities`](discovery.md#get-capabilities) | What this environment can do |
| [`GET /me`](discovery.md#get-me) | Who this API key belongs to |

### [Quotes and payments](payments.md)

| Endpoint | |
|---|---|
| [`POST /quotes`](payments.md#post-quotes) | Get a firm, rate-locked quote |
| [`POST /transactions`](payments.md#post-transactions) | Send a payment |
| [`GET /transactions`](payments.md#get-transactions) | List your payments |
| [`GET /transactions/:id`](payments.md#get-transactionsid) | A payment's status and timeline |

### [Batch payouts](batches.md)

| Endpoint | |
|---|---|
| [`POST /batches`](batches.md#post-batches) | Submit a batch payout |
| [`GET /batches`](batches.md#get-batches) | List your batches |
| [`GET /batches/:batchId`](batches.md#get-batchesbatchid) | A batch, its item counts and a page of items |

### [Balance and wallets](balance.md)

| Endpoint | |
|---|---|
| [`GET /balance`](balance.md#get-balance) | Your USDC balance |
| [`GET /wallets`](balance.md#get-wallets) | Your wallets |
| [`GET /wallets/:walletId`](balance.md#get-walletswalletid) | One of your wallets |
| [`GET /wallets/:walletId/balances`](balance.md#get-walletswalletidbalances) | What the wallet provider reports a wallet holds |
| [`GET /issuers/accounts`](balance.md#get-issuersaccounts) | Issuer accounts held for you |
| [`GET /issuers`](balance.md#get-issuers) | The stablecoin issuers this environment supports |
| [`POST /wallets/:walletId/transfers`](balance.md#post-walletswalletidtransfers) | Move funds between your wallets *(not available)* |
| [`POST /issuers/:issuerId/mint`](balance.md#post-issuersissueridmint) | Mint a stablecoin at par *(not available)* |
| [`POST /banking/wire-transfers`](balance.md#post-bankingwire-transfers) | Send an outbound wire *(not available)* |

### [Deposits](deposits.md)

| Endpoint | |
|---|---|
| [`GET /treasury/deposit-instructions`](deposits.md#get-treasurydeposit-instructions) | Where to wire fiat, and the reference to quote |
| [`GET /treasury/onramp/quote`](deposits.md#get-treasuryonrampquote) | Price a wire deposit without starting one |
| [`POST /treasury/onramp`](deposits.md#post-treasuryonramp) | Open a wire deposit |
| [`GET /treasury/onramps`](deposits.md#get-treasuryonramps) | List your deposits |
| [`GET /treasury/onramp/:onRampId`](deposits.md#get-treasuryonramponrampid) | A deposit's status and timeline |
| [`GET /treasury/onramp/pix/quote`](deposits.md#get-treasuryonramppixquote) | Price a PIX deposit |
| [`POST /treasury/onramp/pix`](deposits.md#post-treasuryonramppix) | Open a PIX deposit |
| [`GET /treasury/crypto-deposit`](deposits.md#get-treasurycrypto-deposit) | Where to send USDC on chain, and what each network costs |

### [Withdrawals and off-ramps](withdrawals.md)

| Endpoint | |
|---|---|
| [`GET /treasury/withdrawals/quote`](withdrawals.md#get-treasurywithdrawalsquote) | Price a USDC withdrawal to your own wallet |
| [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals) | Withdraw USDC to your own wallet |
| [`GET /treasury/withdrawals`](withdrawals.md#get-treasurywithdrawals) | List your withdrawals |
| [`GET /treasury/withdrawals/:withdrawalId`](withdrawals.md#get-treasurywithdrawalswithdrawalid) | A withdrawal's status and timeline |
| [`POST /treasury/beneficiaries`](withdrawals.md#post-treasurybeneficiaries) | Register a bank account an off-ramp can pay |
| [`GET /treasury/beneficiaries`](withdrawals.md#get-treasurybeneficiaries) | Your registered beneficiaries |
| [`GET /treasury/offramp/quote`](withdrawals.md#get-treasuryofframpquote) | Price an off-ramp to a beneficiary |
| [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp) | Pay USDC out to a beneficiary's bank account |
| [`GET /treasury/offramps`](withdrawals.md#get-treasuryofframps) | List your off-ramps |
| [`GET /treasury/offramp/:offRampId`](withdrawals.md#get-treasuryofframpofframpid) | An off-ramp's status and timeline |

### [Recipients](recipients.md)

| Endpoint | |
|---|---|
| [`GET /recipients`](recipients.md#get-recipients) | Your saved recipients |
| [`POST /recipients`](recipients.md#post-recipients) | Save a recipient |
| [`PATCH /recipients/:id`](recipients.md#patch-recipientsid) | Edit a saved recipient |
| [`DELETE /recipients/:id`](recipients.md#delete-recipientsid) | Delete a saved recipient |

### [Team and API keys](team.md)

| Endpoint | |
|---|---|
| [`GET /members`](team.md#get-members) | Your team |
| [`POST /members/invitations`](team.md#post-membersinvitations) | Invite a team member |
| [`POST /members/:id/invitations`](team.md#post-membersidinvitations) | Re-send an invitation |
| [`POST /invitations/accept`](team.md#post-invitationsaccept) | Accept an invitation |
| [`PATCH /members/:id`](team.md#patch-membersid) | Change a member's role |
| [`DELETE /members/:id`](team.md#delete-membersid) | Revoke a member |
| [`POST /members/me/api-key`](team.md#post-membersmeapi-key) | Issue your own member API key |
| [`POST /api-keys/rotate`](team.md#post-api-keysrotate) | Rotate the organization API key |

### [Webhooks](webhooks.md)

| Endpoint | |
|---|---|
| [`GET /webhooks/endpoint`](webhooks.md#get-webhooksendpoint) | The registered webhook URL |
| [`PUT /webhooks/endpoint`](webhooks.md#put-webhooksendpoint) | Register or replace the webhook URL |
| [`DELETE /webhooks/endpoint`](webhooks.md#delete-webhooksendpoint) | Stop webhook delivery |

### [Email notifications](notifications.md)

| Endpoint | |
|---|---|
| [`GET /notifications/preferences`](notifications.md#get-notificationspreferences) | Which events you are emailed about |
| [`PUT /notifications/preferences`](notifications.md#put-notificationspreferences) | Choose which events you are emailed about |
| [`POST /notifications/verify/send`](notifications.md#post-notificationsverifysend) | Email yourself a verification link |

### [Statements](statements.md)

| Endpoint | |
|---|---|
| [`GET /statements`](statements.md#get-statements) | Your fee statement for a period |

### [Sandbox tools](sandbox.md)

| Endpoint | |
|---|---|
| [`POST /treasury/onramp/:onRampId/simulate-payment`](sandbox.md#post-treasuryonramponrampidsimulate-payment) | Make a provider-collected deposit's payment arrive *(sandbox)* |
| [`POST /treasury/onramps/simulate-wire`](sandbox.md#post-treasuryonrampssimulate-wire) | Make a wire arrive at our account, quoting your reference *(sandbox)* |
| [`GET /treasury/sandbox-prefill`](sandbox.md#get-treasurysandbox-prefill) | Inputs the sandbox providers are proven to settle *(sandbox)* |
| [`POST /wallets/:walletId/topup-eurc`](sandbox.md#post-walletswalletidtopup-eurc) | Top up a simulated EURC balance *(sandbox)* |
