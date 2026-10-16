---
description: "Which events each team member is emailed about."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Email notifications

Which events each team member is emailed about.

| Endpoint | |
|---|---|
| [`GET /notifications/preferences`](#get-notificationspreferences) | Which events you are emailed about |
| [`PUT /notifications/preferences`](#put-notificationspreferences) | Choose which events you are emailed about |
| [`POST /notifications/verify/send`](#post-notificationsverifysend) | Email yourself a verification link |

## GET /notifications/preferences

Which events you are emailed about.

**Auth:** API key · **Rate limit:** `read`

Per member, so it needs a member key or a dashboard session, not the organization key.

[Read the guide](../api/notifications.md)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `409` | Conflict | `member_required` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `verificationAvailable` | boolean | yes |  |
| `email` | string | no |  |
| `updatedAt` | string | yes |  |
| `suppressionReason` | string | no |  |
| `suppressedAt` | string | no |  |
| `events` | [NotificationEventType](types.md#notificationeventtype)[] | yes |  |
| `emailVerified` | boolean | yes |  |
| `availableEvents` | [NotificationEventType](types.md#notificationeventtype)[] | yes |  |

## PUT /notifications/preferences

Choose which events you are emailed about.

**Auth:** API key · **Rate limit:** `write`

Send the full set you want; an empty list turns every email off.

[Read the guide](../api/notifications.md)

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `events` | `"transaction.paid_out"`, `"transaction.reconciled"`, `"transaction.refunded"`, `"transaction.in_doubt"`, `"onramp.delivered"`, `"onramp.refunded"`, `"offramp.paid"`[] | yes | at most 7 items |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `409` | Conflict | `member_required` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `verificationAvailable` | boolean | yes |  |
| `email` | string | no |  |
| `updatedAt` | string | yes |  |
| `suppressionReason` | string | no |  |
| `suppressedAt` | string | no |  |
| `events` | [NotificationEventType](types.md#notificationeventtype)[] | yes |  |
| `emailVerified` | boolean | yes |  |
| `availableEvents` | [NotificationEventType](types.md#notificationeventtype)[] | yes |  |

## POST /notifications/verify/send

Email yourself a verification link.

**Auth:** API key · **Rate limit:** `write`

Always to the address on your own member record. Nothing is emailed until it is verified.

[Read the guide](../api/notifications.md)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `202` | Accepted: started, not finished |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `409` | Conflict | `member_required`, `no_member_email` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`202` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `sent` | boolean | yes |  |
| `email` | string | yes |  |
| `expiresAt` | string | yes |  |
