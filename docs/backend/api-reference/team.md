---
description: "Members, roles, invitations and keys."
---

<!-- GENERATED from the API source by `pnpm api:reference` in the backend repo. Do not edit by hand. -->

# Team and API keys

Members, roles, invitations and keys.

| Endpoint | |
|---|---|
| [`GET /members`](#get-members) | Your team |
| [`POST /members/invitations`](#post-membersinvitations) | Invite a team member |
| [`POST /members/:id/invitations`](#post-membersidinvitations) | Re-send an invitation |
| [`POST /invitations/accept`](#post-invitationsaccept) | Accept an invitation |
| [`PATCH /members/:id`](#patch-membersid) | Change a member's role |
| [`DELETE /members/:id`](#delete-membersid) | Revoke a member |
| [`POST /members/me/api-key`](#post-membersmeapi-key) | Issue your own member API key |
| [`POST /api-keys/rotate`](#post-api-keysrotate) | Rotate the organization API key |

## GET /members

Your team.

**Auth:** API key · **Permission:** `member.read` (owner, admin) · **Rate limit:** `read`

[Read the guide](../api/team.md#get-members)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `members` | object[] | yes |  |
| `members[].id` | string | yes |  |
| `members[].email` | string | yes |  |
| `members[].name` | string or null | yes |  |
| `members[].role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `members[].status` | string | yes |  |
| `members[].invitedAt` | string | yes |  |
| `members[].acceptedAt` | string or null | yes |  |
| `members[].createdAt` | string | yes |  |
| `count` | number | yes |  |

## POST /members/invitations

Invite a team member.

**Auth:** API key · **Permission:** `member.invite` (owner, admin) · **Rate limit:** `write`

You can only grant a role below your own. The invitation is emailed; `emailDelivered` says whether it was.

[Read the guide](../api/team.md#post-membersinvitations)

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `email` | string (email) | yes | at most 320 chars |  |
| `name` | string | no | 1–200 chars |  |
| `role` | string | yes |  |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `201` | Created |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `409` | Conflict | `conflict` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`201` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `emailDelivered` | boolean | yes |  |
| `emailError` | string | no |  |
| `member` | object | yes |  |
| `member.id` | string | yes |  |
| `member.email` | string | yes |  |
| `member.name` | string or null | yes |  |
| `member.role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `member.status` | string | yes |  |
| `member.invitedAt` | string | yes |  |
| `member.acceptedAt` | string or null | yes |  |
| `member.createdAt` | string | yes |  |

## POST /members/:id/invitations

Re-send an invitation.

**Auth:** API key · **Permission:** `member.invite` (owner, admin) · **Rate limit:** `write`

Issues a fresh link; the old one stops working.

[Read the guide](../api/team.md#post-membersidinvitations)

**Path parameters**

| Name | Type |
|---|---|
| `id` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `emailDelivered` | boolean | yes |  |
| `emailError` | string | no |  |
| `member` | object | yes |  |
| `member.id` | string | yes |  |
| `member.email` | string | yes |  |
| `member.name` | string or null | yes |  |
| `member.role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `member.status` | string | yes |  |
| `member.invitedAt` | string | yes |  |
| `member.acceptedAt` | string or null | yes |  |
| `member.createdAt` | string | yes |  |

## POST /invitations/accept

Accept an invitation.

**Auth:** none

Open: the invitation token is the credential. Returns the new member's API key **once**.

[Read the guide](../api/team.md#post-invitationsaccept)

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `token` | string | yes | 1–200 chars |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `invalid_invite`, `validation` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `apiKey` | string | yes |  |
| `member` | object | yes |  |
| `member.id` | string | yes |  |
| `member.email` | string | yes |  |
| `member.name` | string or null | yes |  |
| `member.role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `member.status` | string | yes |  |
| `member.invitedAt` | string | yes |  |
| `member.acceptedAt` | string or null | yes |  |
| `member.createdAt` | string | yes |  |

## PATCH /members/:id

Change a member's role.

**Auth:** API key · **Permission:** `member.manage` (owner, admin) · **Rate limit:** `write`

[Read the guide](../api/team.md#patch-membersid)

**Path parameters**

| Name | Type |
|---|---|
| `id` | string |

**Request body** · `application/json` · unknown fields are **rejected** with `400`

| Field | Type | Required | Rules | Notes |
|---|---|:---:|---|---|
| `role` | string | yes |  |  |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `400` | Invalid request | `validation` |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `member` | object | yes |  |
| `member.id` | string | yes |  |
| `member.email` | string | yes |  |
| `member.name` | string or null | yes |  |
| `member.role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `member.status` | string | yes |  |
| `member.invitedAt` | string | yes |  |
| `member.acceptedAt` | string or null | yes |  |
| `member.createdAt` | string | yes |  |

## DELETE /members/:id

Revoke a member.

**Auth:** API key · **Permission:** `member.manage` (owner, admin) · **Rate limit:** `write`

[Read the guide](../api/team.md#delete-membersid)

**Path parameters**

| Name | Type |
|---|---|
| `id` | string |

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `member` | object | yes |  |
| `member.id` | string | yes |  |
| `member.email` | string | yes |  |
| `member.name` | string or null | yes |  |
| `member.role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `member.status` | string | yes |  |
| `member.invitedAt` | string | yes |  |
| `member.acceptedAt` | string or null | yes |  |
| `member.createdAt` | string | yes |  |
| `rescinded` | boolean | yes |  |

## POST /members/me/api-key

Issue your own member API key.

**Auth:** API key · **Rate limit:** `write`

For a signed-in member: mints a key carrying your existing role and revokes your previous one. Shown **once**. Not for the organization key; that is `POST /api-keys/rotate`.

[Read the guide](../api/team.md#post-membersmeapi-key)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `404` | Not found, or not yours | `not_found` |
| `409` | Conflict | `member_required` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `apiKey` | string | yes |  |
| `member` | object | yes |  |
| `member.id` | string | yes |  |
| `member.email` | string | yes |  |
| `member.name` | string or null | yes |  |
| `member.role` | [IntegratorRole](types.md#integratorrole) | yes |  |
| `member.status` | string | yes |  |
| `member.invitedAt` | string | yes |  |
| `member.acceptedAt` | string or null | yes |  |
| `member.createdAt` | string | yes |  |
| `rotatedAt` | string | yes |  |

## POST /api-keys/rotate

Rotate the organization API key.

**Auth:** API key · **Rate limit:** `write`

Owner only, with the organization key. The new key is returned **once** and the key that made this request stops working immediately.

[Read the guide](../api/team.md#post-api-keysrotate)

**Responses**

| Status | Meaning | `error` codes |
|---|---|---|
| `200` | OK |  |
| `401` | Missing or invalid credentials | `unauthorized` |
| `403` | Not allowed | `forbidden` |
| `404` | Not found, or not yours | `not_found` |
| `429` | Rate limited | `rate_limited` |
| `501` | Not available on this environment | `not_implemented` |

**`200` body**

| Field | Type | Always present | Notes |
|---|---|:---:|---|
| `apiKey` | string | yes |  |
| `rotatedAt` | string | yes |  |
