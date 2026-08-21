---
description: One credential for machines, one for people, and what each may do.
---

# Authentication

Genifi authenticates two different kinds of caller, and it matters which one you
are.

| Subject | Credential | Used by |
|---|---|---|
| **An organization** | `Authorization: Bearer <apiKey>` | Your servers, calling this API |
| **A person** | A session cookie set by signing in | The Genifi Dashboard |

Everything on this site describes the first. Per-user features (notification
preferences) are the only endpoints that need the second, and they exist for the
dashboard's benefit.

## API keys

Your API key is issued at onboarding. It identifies your organization on every
call:

```bash
curl "$GENIFI_API/transactions" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

**The key determines who you are — always.** Nothing in a request body can
change it. There is no `integratorId` field on any endpoint, by design: a client
must not be able to claim to be a different organization by editing a field.

### Which routes need a key

Open, no credentials:

* `GET /health`
* `GET /capabilities`
* `GET /metrics`

Everything else requires a key, including endpoints that are not yet
implemented — we will not tell an anonymous caller what our feature surface
looks like.

One deliberate exception: `POST /invitations/accept` is unauthenticated, because
the invite token *is* the credential and the invitee has no key yet.

### Rotating a key

```bash
curl -X POST "$GENIFI_API/api-keys/rotate" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "apiKey": "gk_new…", "rotatedAt": "2026-08-21T14:09:00.000Z" }
```

The new key is shown **once**. The old key stops working immediately, so deploy
the new one before you rotate, or accept a gap. Rotation is an owner/admin
action.

!!! danger

    There is no endpoint that reads your key back. If you lose it, rotate. This is
    deliberate: a read path would turn any read-scoped credential leak into a
    money-moving one.

## Roles

Every member of your organization has a role, and the role gates what the
credential may do.

| Action | viewer | member | admin | owner |
|---|:---:|:---:|:---:|:---:|
| Read transactions | ✅ | ✅ | ✅ | ✅ |
| Read accounts | ✅ | ✅ | ✅ | ✅ |
| Read saved recipients | ✅ | ✅ | ✅ | ✅ |
| Request quotes | — | ✅ | ✅ | ✅ |
| Initiate payments | — | ✅ | ✅ | ✅ |
| Manage saved recipients | — | ✅ | ✅ | ✅ |
| Manage webhooks | — | — | ✅ | ✅ |
| View and invite team members | — | — | ✅ | ✅ |
| Change and revoke member roles | — | — | ✅ | ✅ |

`owner` is the organization itself and is never assignable — you cannot grant or
revoke it, and there is no "last owner" hazard among members. Assignable roles
are `admin`, `member`, and `viewer`.

A role failure is a `403`:

```json
{ "error": "forbidden", "message": "your role may not initiate payments" }
```

## The KYB gate

Before your organization may move any funds, Genifi must have verified it.
Until then, money-moving endpoints (`POST /transactions`, `POST /treasury/onramp`,
`POST /issuers/:issuerId/redeem`) refuse the request regardless of what you send.

This is checked *before* the request body is parsed, so an unverified
organization gets the same answer whatever it sends.

## Failure responses

Every authentication failure looks identical:

```json
{ "error": "unauthorized", "message": "missing or invalid credentials" }
```

Unknown key, malformed header, suspended organization, expired session — all
`401`, all the same body. We never reveal which, because the difference is
information an attacker can use.

## Handling a 401 in your client

Treat `401` as *this credential is dead*, not as *retry*. Stop, alert, and
replace the key. A retry loop against a rejected key will only get you rate
limited.
