---
description: One credential for machines, one for people, and what each may do.
---

# Authentication

JET authenticates two different kinds of caller, and it matters which one you
are.

| Subject | Credential | Used by |
|---|---|---|
| **An organization** | `Authorization: Bearer <apiKey>` | Your servers, calling this API |
| **A person** | A session cookie set by signing in | The JET Dashboard |

Everything on this site describes the first. Per-user features (notification
preferences) are the only endpoints that need the second, and they exist for the
dashboard's benefit.

## API keys

Your API key is issued at onboarding. It identifies your organization on every
call:

```bash
curl "$JET_API/transactions" \
  -H "Authorization: Bearer $JET_API_KEY"
```

**The key always determines who you are.** Nothing in a request body can
change it. There is no `integratorId` field on any endpoint, by design: a client
must not be able to claim to be a different organization by editing a field.

### Which routes need a key

Open, no credentials:

* `GET /health`
* `GET /capabilities`

Everything else requires a key, including endpoints that are not yet
implemented: we will not tell an anonymous caller what our feature surface
looks like.

One deliberate exception: `POST /invitations/accept` is unauthenticated, because
the invite token *is* the credential and the invitee has no key yet.

### Rotating a key

```bash
curl -X POST "$JET_API/api-keys/rotate" \
  -H "Authorization: Bearer $JET_API_KEY"
```

```json
{ "apiKey": "gk_new…", "rotatedAt": "2026-08-21T14:09:00.000Z" }
```

The new key is shown **once**. The old key stops working immediately, so deploy
the new one before you rotate, or accept a gap. Only the organization key itself
can rotate it; a member key cannot.

### Member keys

Each team member can also hold their own key, which carries their role rather
than the organization's. A member receives one when they
[accept an invitation](../api/team.md#post-invitationsaccept), and can issue
themselves a new one, revoking the old, with
[`POST /members/me/api-key`](../api/team.md#post-membersmeapi-key). Use member keys
when you want a credential limited to a role, or traceable to a person.

!!! danger

    There is no endpoint that reads your key back. If you lose it, rotate. This is
    deliberate: a read path would turn any read-scoped credential leak into a
    money-moving one.

## Roles

Every member of your organization has a role, and the role gates what the
credential may do.

| Action | viewer | member | admin | owner |
|---|:---:|:---:|:---:|:---:|
| Read payments, balances, wallets, deposits and recipients | yes | yes | yes | yes |
| Request quotes and send payments and batches | no | yes | yes | yes |
| Save and edit recipients | no | yes | yes | yes |
| Deposit, withdraw and off-ramp | no | yes | yes | yes |
| Register off-ramp beneficiaries | no | yes | yes | yes |
| Manage webhooks | no | no | yes | yes |
| View, invite and manage team members | no | no | yes | yes |

The exact permission each endpoint checks, and which roles hold it, is
[generated from the code](../api-reference/README.md#roles-and-permissions).
An admin can manage `member` and `viewer` only; managing admins takes the owner.

`owner` is the organization itself and is never assignable: you cannot grant or
revoke it, and there is no "last owner" hazard among members. Assignable roles
are `admin`, `member`, and `viewer`.

A role failure is a `403`:

```json
{ "error": "forbidden", "message": "your role may not initiate payments" }
```

## The KYB gate

Before your organization may move any funds, JET must have verified it.
Until then, money-moving endpoints refuse the request with `403 kyb_required`,
regardless of what you send. Each one is marked **KYB** in the
[API reference](../api-reference/README.md). Reads, quotes and account setup stay
open, so you can build while verification is in progress.

This is checked *before* the request body is parsed, so an unverified
organization gets the same answer whatever it sends.

## Failure responses

Every authentication failure looks identical:

```json
{ "error": "unauthorized", "message": "missing or invalid credentials" }
```

Unknown key, malformed header, suspended organization, expired session: all
`401`, all the same body. We never reveal which, because the difference is
information an attacker can use.

## Handling a 401 in your client

Treat `401` as *this credential is dead*, not as *retry*. Stop, alert, and
replace the key. A retry loop against a rejected key will only get you rate
limited.
