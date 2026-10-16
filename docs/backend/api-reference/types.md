---
description: "Types shared across the JET API reference."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Shared types

Named types that appear in more than one response, as the API source declares them.

## AccountKindName

One of: `"wallet"`, `"mint_account"`, `"bank_account"`

## BackendCapabilities

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `version` | string | yes |  |
| `environment` | `"simulated"`, `"sandbox"`, `"production"` | yes |  |
| `corridors` | [CorridorCapability](types.md#corridorcapability)[] | yes |  |
| `features` | [FeatureCapability](types.md#featurecapability)[] | yes |  |

## BeneficiaryField

One of: `"name"`, `"account"`, `"country"`, `"bic"`, `"bankCode"`, `"bankName"`, `"city"`, `"addressLine"`, `"state"`, `"postalCode"`, `"phone"`

## BeneficiaryView

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `name` | string | yes |  |
| `accountId` | string | yes |  |
| `bic` | string | yes |  |
| `bankName` | string | yes |  |
| `city` | string | yes |  |
| `addressLine` | string | yes |  |
| `country` | string | yes |  |
| `currency` | string | yes |  |
| `createdAt` | string | yes |  |

## BridgeStatusView

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `bridgeId` | string | no |  |
| `stage` | string | yes |  |
| `detail` | string | no |  |
| `quote` | object | no |  |
| `quote.netAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `quote.speed` | string | yes |  |
| `net` | [MoneyJSON](types.md#moneyjson) | no |  |
| `txHash` | string | no |  |

## CorridorCapability

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `from` | string | yes |  |
| `to` | string | yes |  |
| `destinationCountry` | string | yes |  |
| `ownedRail` | boolean | yes |  |
| `instant` | boolean | yes |  |

## DepositInstructions

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `reference` | string | yes |  |
| `accounts` | object[] | yes |  |
| `accounts[].currency` | string | yes |  |
| `accounts[].accountName` | string | yes |  |
| `accounts[].bankName` | string | no |  |
| `accounts[].iban` | string | no |  |
| `accounts[].accountNumber` | string | no |  |
| `accounts[].bic` | string | no |  |
| `accounts[].bankAddress` | string | no |  |

## FeatureCapability

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `feature` | string | yes |  |
| `status` | `"simulated"`, `"live"`, `"partial"`, `"planned"` | yes |  |
| `reason` | string | yes |  |
| `requires` | string | no |  |

## IntegratorAction

One of: `"transaction.read"`, `"account.read"`, `"payment.initiate"`, `"webhook.manage"`, `"member.read"`, `"member.invite"`, `"member.manage"`, `"recipient.read"`, `"recipient.manage"`, `"treasury.move"`, `"beneficiary.manage"`

## IntegratorRole

One of: `"owner"`, `"admin"`, `"member"`, `"viewer"`

## IssuerCatalogEntry

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `id` | string | yes |  |
| `name` | string | yes |  |
| `redeems` | string[] | yes |  |
| `canRedeem` | boolean | yes |  |
| `environment` | string | yes |  |

## MoneyJSON

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `amount` | string | yes |  |
| `currency` | string | yes |  |
| `scale` | number | yes |  |

## NotificationEventType

One of: `"transaction.paid_out"`, `"transaction.reconciled"`, `"transaction.refunded"`, `"transaction.in_doubt"`, `"onramp.delivered"`, `"onramp.refunded"`, `"offramp.paid"`

## OffRampQuote

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `pulled` | object | yes |  |
| `pulled.amount` | string | yes |  |
| `pulled.currency` | string | yes |  |
| `pulled.scale` | number | yes |  |
| `settlementCurrency` | string | yes |  |
| `atPar` | boolean | yes |  |
| `feeBps` | number | yes |  |
| `fee` | object | no |  |
| `fee.amount` | string | yes |  |
| `fee.currency` | string | yes |  |
| `fee.scale` | number | yes |  |
| `deliveryFee` | object | yes |  |
| `deliveryFee.amount` | string | yes |  |
| `deliveryFee.currency` | string | yes |  |
| `deliveryFee.scale` | number | yes |  |
| `net` | object | no |  |
| `net.amount` | string | yes |  |
| `net.currency` | string | yes |  |
| `net.scale` | number | yes |  |
| `fxBufferBps` | number | no |  |
| `minimum` | object | yes |  |
| `minimum.amount` | string | yes |  |
| `minimum.currency` | string | yes |  |
| `minimum.scale` | number | yes |  |
| `quoteId` | string | no |  |
| `expiresAt` | string | no |  |

## OffRampSummary

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `offRampId` | string | yes |  |
| `amount` | object | yes |  |
| `amount.amount` | string | yes |  |
| `amount.currency` | string | yes |  |
| `settlementCurrency` | string | yes |  |
| `beneficiaryId` | string | yes |  |
| `createdAt` | string | yes |  |

## OnRampEvent

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `stage` | [OnRampStage](types.md#onrampstage) | yes |  |
| `at` | string | yes |  |
| `detail` | string | no |  |

## OnRampInstructions

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `reference` | string | yes |  |
| `paymentUrl` | string | no |  |
| `iban` | string | no |  |
| `bic` | string | no |  |
| `bankName` | string | no |  |
| `bankAddress` | string | no |  |
| `accountNumber` | string | no |  |
| `accountName` | string | no |  |
| `expiresAt` | string | no |  |
| `brCode` | string | no |  |
| `pay` | [MoneyJSON](types.md#moneyjson) | no |  |
| `receive` | [MoneyJSON](types.md#moneyjson) | no |  |
| `provider` | string | no |  |

## OnRampStage

One of: `"FAILED"`, `"IN_DOUBT"`, `"REFUNDED"`, `"DELIVERED"`, `"EXPIRED"`, `"OPENING_COLLECTION"`, `"AWAITING_PAYMENT"`, `"PAYMENT_RECEIVED"`, `"FORWARDING"`, `"CONVERTING"`, `"DELIVERING"`

## OnRampSummary

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `onRampId` | string | yes |  |
| `origin` | `"api"`, `"deposit"`, `"pix"` | yes |  |
| `amount` | object | yes |  |
| `amount.amount` | string | yes |  |
| `amount.currency` | string | yes |  |
| `targetStablecoin` | string | yes |  |
| `destinationAddress` | string | yes |  |
| `createdAt` | string | yes |  |
| `fundingProvider` | string | no |  |
| `fundingDepositId` | string | no |  |

## OwnedMarginJSON

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `currency` | string | yes |  |
| `scale` | number | yes |  |
| `amountMinorUnits` | string | yes |  |
| `bps` | number | yes |  |
| `realized` | boolean | yes |  |
| `ownedProvider` | string | yes |  |
| `orchestratorProvider` | string | yes |  |

## PayoutDelta

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `quoted` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `paid` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `deltaMinorUnits` | string | yes |  |
| `deltaBps` | number | no |  |
| `direction` | `"short"`, `"over"`, `"currency_mismatch"` | yes |  |

## ProviderGap

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `provider` | string | yes |  |
| `missing` | [BeneficiaryField](types.md#beneficiaryfield)[] | yes |  |

## QuotePreviewResult

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `quoteId` | string | yes |  |
| `sendAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `receiveAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `fee` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `rate` | [RateJSON](types.md#ratejson) | yes |  |
| `provider` | string | yes |  |
| `routeKind` | `"orchestrator"`, `"owned"` | yes |  |
| `estimatedSeconds` | number | yes |  |
| `expiresAt` | string | yes |  |
| `margin` | [OwnedMarginJSON](types.md#ownedmarginjson) | no |  |
| `networkFee` | [MoneyJSON](types.md#moneyjson) | no |  |
| `routed` | boolean | no |  |

## RateJSON

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `from` | string | yes |  |
| `to` | string | yes |  |
| `numerator` | string | yes |  |
| `denominator` | string | yes |  |

## RedeemState

One of: `"SETTLED"`, `"FAILED"`, `"IN_DOUBT"`, `"REFUNDED"`

## ReportedBalance

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `asset` | string | yes |  |
| `amount` | string | yes |  |
| `scale` | number | yes |  |
| `settled` | boolean | yes |  |
| `reportedAt` | string | yes |  |

## SandboxBeneficiary

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `name` | string | yes |  |
| `account` | string | yes |  |
| `bankCode` | string | no |  |
| `bic` | string | no |  |
| `bankName` | string | no |  |
| `country` | string | no |  |
| `city` | string | no |  |
| `addressLine` | string | no |  |
| `state` | string | no |  |
| `postalCode` | string | no |  |
| `phone` | object | no |  |
| `phone.callingCode` | string | yes |  |
| `phone.number` | string | yes |  |

## SandboxOffRampBeneficiary

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `name` | string | yes |  |
| `accountId` | string | yes |  |
| `bic` | string | yes |  |
| `bankName` | string | yes |  |
| `city` | string | yes |  |
| `addressLine` | string | yes |  |
| `country` | string | yes |  |
| `currency` | string | yes |  |

## SandboxOutcomeBand

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `provider` | `"tazapay"` | yes |  |
| `fromMinor` | string | yes |  |
| `toMinor` | string | yes |  |
| `outcome` | string | yes |  |

## SavedRecipientType

One of: `"wallet"`, `"bank"`

## Statement

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `integratorId` | string | yes |  |
| `period` | object | yes |  |
| `period.from` | string | yes |  |
| `period.to` | string | yes |  |
| `lines` | [StatementLine](types.md#statementline)[] | yes |  |
| `totals` | [StatementTotals](types.md#statementtotals) | yes |  |
| `generatedAt` | string | yes |  |

## StatementCorridor

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `from` | string | yes |  |
| `to` | string | yes |  |
| `destinationCountry` | string | yes |  |

## StatementLine

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactionId` | string | yes |  |
| `corridor` | [StatementCorridor](types.md#statementcorridor) | yes |  |
| `sendAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `recipientReference` | string | yes |  |
| `fee` | [MoneyJSON](types.md#moneyjson)[] | yes |  |
| `initiatedAt` | string | yes |  |

## StatementTotals

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactionCount` | number | yes |  |
| `initiatedVolume` | [MoneyJSON](types.md#moneyjson)[] | yes |  |
| `feesEarned` | [MoneyJSON](types.md#moneyjson)[] | yes |  |

## TransactionListItem

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactionId` | string | yes |  |
| `corridor` | object | yes |  |
| `corridor.from` | string | yes |  |
| `corridor.to` | string | yes |  |
| `corridor.destinationCountry` | string | yes |  |
| `sendAmount` | [MoneyJSON](types.md#moneyjson) | yes |  |
| `recipientReference` | string | yes |  |
| `createdAt` | string | yes |  |

## TransactionStatus

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `transactionId` | string | yes |  |
| `running` | boolean | yes |  |
| `failure` | string | no |  |
| `state` | `"FAILED"`, `"IN_DOUBT"`, `"REFUNDED"`, `"INITIATED"`, `"QUOTED"`, `"FUNDED"`, `"ROUTING"`, `"SETTLING"`, `"PAID_OUT"`, `"RECONCILED"`, `"REFUNDING"`, `"PENDING_RETRY"`, `"NEEDS_ATTENTION"` | no |  |
| `provider` | string | no |  |
| `payoutMode` | `"regular"`, `"instant"` | no |  |
| `receiveAmount` | object | no |  |
| `receiveAmount.amount` | string | yes |  |
| `receiveAmount.currency` | string | yes |  |
| `receiveAmount.scale` | number | yes |  |
| `payoutDelta` | [PayoutDelta](types.md#payoutdelta) | no |  |
| `margin` | [OwnedMarginJSON](types.md#ownedmarginjson) | no |  |
| `timeline` | object[] | no |  |
| `timeline[].state` | string | yes |  |
| `timeline[].detail` | string | no |  |

## WithdrawalSummary

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `withdrawalId` | string | yes |  |
| `amount` | object | yes |  |
| `amount.amount` | string | yes |  |
| `amount.currency` | string | yes |  |
| `network` | string | yes |  |
| `toAddress` | string | yes |  |
| `createdAt` | string | yes |  |
