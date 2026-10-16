---
description: "The fees you were charged over a period."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Statements

The fees you were charged over a period.

| Endpoint | |
|---|---|
| [`GET /statements`](#get-statements) | Your fee statement for a period |

## GET /statements

Your fee statement for a period.

**Auth:** API key · **Rate limit:** `read`

The fees actually booked against your payments in `[from, to)`, defaulting to the last 30 days. A report, not an invoice.

[Read the guide](../api/statements.md)

**Query parameters**

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `from` | string (date-time) | no |  |  |
| `to` | string (date-time) | no |  |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `invalid_period`, `invalid_query`, `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body** · [Statement](types.md#statement)

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `integratorId` | string | yes |  |
| `period` | object | yes |  |
| `period.from` | string | yes |  |
| `period.to` | string | yes |  |
| `lines` | [StatementLine](types.md#statementline)[] | yes |  |
| `totals` | [StatementTotals](types.md#statementtotals) | yes |  |
| `generatedAt` | string | yes |  |
