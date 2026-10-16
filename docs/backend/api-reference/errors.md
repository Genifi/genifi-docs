---
description: "Every error code the JET API returns, and which endpoints return it."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Error codes

Every value of `error` an endpoint can send, read from the API source. Branch on `error`, never on `message`. What each status means and when to retry: [Errors](../reference/errors.md).

Some errors can come from any endpoint and are not repeated below: `500 internal`; a `503` when a dependency of ours is unavailable; and, for a body that is not valid JSON at all, `invalid_json`, `unsupported_media_type`, `payload_too_large` or `bad_request`.

| `error` | Status | Returned by |
|---|---|---|
| `ambiguous_home_wallet` | 422 | [`POST /transactions`](payments.md#post-transactions), [`POST /batches`](batches.md#post-batches), [`POST /treasury/onramp`](deposits.md#post-treasuryonramp), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals) |
| `beneficiary_incomplete` | 400 | [`POST /transactions`](payments.md#post-transactions) |
| `compliance_hold` | 403 | [`POST /transactions`](payments.md#post-transactions) |
| `conflict` | 409 | [`POST /members/invitations`](team.md#post-membersinvitations), [`POST /wallets/:walletId/topup-eurc`](sandbox.md#post-walletswalletidtopup-eurc) |
| `duplicate_item_ref` | 400 | [`POST /batches`](batches.md#post-batches) |
| `forbidden` | 403 | 23 endpoints |
| `idempotency_key_conflict` | 409 | [`POST /batches`](batches.md#post-batches), [`POST /treasury/onramp`](deposits.md#post-treasuryonramp), [`POST /treasury/onramp/pix`](deposits.md#post-treasuryonramppix), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals), [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp) |
| `idempotency_key_reused` | 409 | [`POST /treasury/onramp`](deposits.md#post-treasuryonramp), [`POST /treasury/onramp/pix`](deposits.md#post-treasuryonramppix), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals) |
| `insufficient_funds` | 422 | [`POST /transactions`](payments.md#post-transactions) |
| `invalid_cursor` | 400 | [`GET /transactions`](payments.md#get-transactions), [`GET /batches`](batches.md#get-batches), [`GET /batches/:batchId`](batches.md#get-batchesbatchid), [`GET /treasury/onramps`](deposits.md#get-treasuryonramps), [`GET /treasury/withdrawals`](withdrawals.md#get-treasurywithdrawals), [`GET /treasury/offramps`](withdrawals.md#get-treasuryofframps) |
| `invalid_invite` | 400 | [`POST /invitations/accept`](team.md#post-invitationsaccept) |
| `invalid_period` | 400 | [`GET /statements`](statements.md#get-statements) |
| `invalid_query` | 400 | [`GET /statements`](statements.md#get-statements) |
| `invalid_send_amount` | 400 | [`POST /batches`](batches.md#post-batches) |
| `kyb_required` | 403 | [`POST /transactions`](payments.md#post-transactions), [`POST /batches`](batches.md#post-batches), [`GET /treasury/deposit-instructions`](deposits.md#get-treasurydeposit-instructions), [`POST /treasury/onramp`](deposits.md#post-treasuryonramp), [`POST /treasury/onramp/pix`](deposits.md#post-treasuryonramppix), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals), [`POST /treasury/beneficiaries`](withdrawals.md#post-treasurybeneficiaries), [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp), [`POST /treasury/onramps/simulate-wire`](sandbox.md#post-treasuryonrampssimulate-wire), [`POST /wallets/:walletId/topup-eurc`](sandbox.md#post-walletswalletidtopup-eurc) |
| `member_required` | 409 | [`POST /members/me/api-key`](team.md#post-membersmeapi-key), [`GET /notifications/preferences`](notifications.md#get-notificationspreferences), [`PUT /notifications/preferences`](notifications.md#put-notificationspreferences), [`POST /notifications/verify/send`](notifications.md#post-notificationsverifysend) |
| `no_home_wallet` | 422 | [`POST /transactions`](payments.md#post-transactions), [`POST /batches`](batches.md#post-batches), [`POST /treasury/onramp`](deposits.md#post-treasuryonramp), [`GET /treasury/crypto-deposit`](deposits.md#get-treasurycrypto-deposit), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals) |
| `no_member_email` | 409 | [`POST /notifications/verify/send`](notifications.md#post-notificationsverifysend) |
| `no_route_for_chain` | 422 | [`POST /quotes`](payments.md#post-quotes) |
| `not_found` | 404 | 16 endpoints |
| `not_home_chain` | 422 | [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp) |
| `not_home_wallet` | 422 | [`POST /quotes`](payments.md#post-quotes), [`POST /transactions`](payments.md#post-transactions), [`POST /batches`](batches.md#post-batches) |
| `not_implemented` | 501 | 46 endpoints |
| `not_simulatable` | 409 | [`POST /treasury/onramp/:onRampId/simulate-payment`](sandbox.md#post-treasuryonramponrampidsimulate-payment) |
| `pix_onramp_unavailable` | 422 | [`POST /treasury/onramp/pix`](deposits.md#post-treasuryonramppix) |
| `quote_already_used` | 422 | [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp) |
| `quote_expired` | 422 | [`POST /transactions`](payments.md#post-transactions) |
| `quote_mismatch` | 422 | [`POST /transactions`](payments.md#post-transactions), [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp) |
| `quote_not_found` | 422 | [`POST /transactions`](payments.md#post-transactions), [`POST /treasury/offramp`](withdrawals.md#post-treasuryofframp) |
| `quote_unavailable` | 422 | [`POST /quotes`](payments.md#post-quotes), [`GET /treasury/withdrawals/quote`](withdrawals.md#get-treasurywithdrawalsquote), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals) |
| `rate_limited` | 429 | 57 endpoints |
| `unauthorized` | 401 | 57 endpoints |
| `unknown_currency` | 400 | [`POST /treasury/onramps/simulate-wire`](sandbox.md#post-treasuryonrampssimulate-wire) |
| `unknown_recipient` | 400 | [`POST /batches`](batches.md#post-batches) |
| `unsupported_network` | 422 | [`GET /treasury/withdrawals/quote`](withdrawals.md#get-treasurywithdrawalsquote), [`POST /treasury/withdrawals`](withdrawals.md#post-treasurywithdrawals) |
| `validation` | 400 | 28 endpoints |
| `wallet_chain_unknown` | 422 | [`POST /transactions`](payments.md#post-transactions), [`POST /batches`](batches.md#post-batches) |
