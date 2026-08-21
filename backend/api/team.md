# Team and API keys

Your organization is a team. Each member has a role that gates what they may do —
see [Authentication](../getting-started/authentication.md#roles) for the full
permission table.

## GET /members

Requires `member.read` (admin or owner).

{% tabs %}
{% tab title="Request" %}
```bash
curl "$GENIFI_API/members" -H "Authorization: Bearer $GENIFI_API_KEY"
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "members": [
    {
      "id": "mem_7c93",
      "email": "ana@northwind.example",
      "name": "Ana Bright",
      "role": "admin",
      "status": "active",
      "invitedAt": "2026-07-14T10:00:00.000Z",
      "acceptedAt": "2026-07-14T10:12:00.000Z",
      "createdAt": "2026-07-14T10:00:00.000Z"
    }
  ],
  "count": 1
}
```
{% endtab %}
{% endtabs %}

Member records never contain key or token material.

## POST /members/invitations

Invite someone. Requires `member.invite` (admin or owner).

{% tabs %}
{% tab title="Request" %}
```bash
curl -X POST "$GENIFI_API/members/invitations" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "sam@northwind.example",
    "name": "Sam Ortiz",
    "role": "member"
  }'
```
{% endtab %}

{% tab title="201 Created" %}
```json
{
  "member": {
    "id": "mem_a204",
    "email": "sam@northwind.example",
    "name": "Sam Ortiz",
    "role": "member",
    "status": "invited",
    "invitedAt": "2026-08-21T14:31:00.000Z",
    "acceptedAt": null,
    "createdAt": "2026-08-21T14:31:00.000Z"
  },
  "emailDelivered": true
}
```
{% endtab %}

{% tab title="Email failed" %}
```json
{
  "member": { "…": "…" },
  "emailDelivered": false,
  "emailError": "email delivery is not configured"
}
```
{% endtab %}
{% endtabs %}

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `email` | email, max 320 | yes | Where the invite goes |
| `name` | string(1–200) | no | |
| `role` | `"admin"`, `"member"`, `"viewer"` | yes | `owner` is never assignable |

### The invite token goes to the invitee, and to nobody else

The response tells you a member was created and whether the email went out. It
does **not** contain the invite token — not even for the person who sent the
invite. The token is a credential for someone else's account.

`emailDelivered: false` means the invitation stands but nobody was told. Use the
resend endpoint below.

### Privilege escalation is blocked

You may only grant a role you are allowed to manage. An `admin` cannot mint
another `admin` — only an `owner` can.

```json
{ "error": "forbidden", "message": "your role may not grant the role 'admin'" }
```

## POST /members/:id/invitations

Resend an outstanding invitation on a **fresh** token. The old token dies.

```bash
curl -X POST "$GENIFI_API/members/mem_a204/invitations" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

This is not just a convenience. Invite tokens are hashed at rest and shown once,
and each email may hold only one invitation. Without a resend path, an invitation
whose email bounced would leave a member who can never be onboarded and never be
re-invited under that address.

Returns `404` if there is no outstanding invitation for that member.

## POST /invitations/accept

The one write an un-keyed caller may make: the invite token *is* the credential.

{% tabs %}
{% tab title="Request" %}
```bash
curl -X POST "$GENIFI_API/invitations/accept" \
  -H "Content-Type: application/json" \
  -d '{ "token": "inv_…" }'
```
{% endtab %}

{% tab title="200 OK" %}
```json
{
  "apiKey": "gk_…",
  "member": {
    "id": "mem_a204",
    "email": "sam@northwind.example",
    "role": "member",
    "status": "active",
    "acceptedAt": "2026-08-21T15:02:00.000Z",
    "…": "…"
  }
}
```
{% endtab %}

{% tab title="400 Bad token" %}
```json
{ "error": "invalid_invite", "message": "invite token is invalid or already used" }
```
{% endtab %}
{% endtabs %}

The `apiKey` is returned **once**. It is a personal key scoped to that member's
role, distinct from the organization's root key.

## PATCH /members/:id

Change a member's role. Requires `member.manage`.

```bash
curl -X PATCH "$GENIFI_API/members/mem_a204" \
  -H "Authorization: Bearer $GENIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "role": "admin" }'
```

The same escalation rule applies: you cannot set a role you may not grant.

## DELETE /members/:id

Revoke a member, or rescind an invitation that was never accepted.

```bash
curl -X DELETE "$GENIFI_API/members/mem_a204" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "member": { "…": "…" }, "rescinded": true }
```

`rescinded: true` means the member had not yet accepted and the invitation was
withdrawn. `false` means an active member was revoked. Either way their
credential stops working.

`owner` cannot be revoked.

## POST /members/me/api-key

Rotate **your own** personal key. Available to any active member.

```bash
curl -X POST "$GENIFI_API/members/me/api-key" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "apiKey": "gk_new…", "member": { "…": "…" }, "rotatedAt": "2026-08-21T15:10:00.000Z" }
```

Calling it with the **organization root key** returns `409`, because the
organization has no personal key to rotate:

```json
{
  "error": "member_required",
  "message": "A member API key is per-user, and this request is authenticated as the ORGANIZATION…"
}
```

Use the endpoint below instead.

## POST /api-keys/rotate

Rotate the **organization's** root key. Owner only.

```bash
curl -X POST "$GENIFI_API/api-keys/rotate" \
  -H "Authorization: Bearer $GENIFI_API_KEY"
```

```json
{ "apiKey": "gk_new…", "rotatedAt": "2026-08-21T15:12:00.000Z" }
```

{% hint style="danger" %}
The old key — the one that authenticated this very request — is dead the moment
this responds. The new key is shown once and there is no way to read it back.
Store it before you do anything else, and deploy it before you rotate if you
cannot tolerate a gap.
{% endhint %}

## Two kinds of key, side by side

| | Organization root key | Member key |
|---|---|---|
| Identifies | The organization | One person |
| Minted by | Onboarding, then `POST /api-keys/rotate` | Accepting an invitation |
| Rotated by | `POST /api-keys/rotate` (owner only) | `POST /members/me/api-key` |
| Role | `owner` | The member's assigned role |

Use the root key for server-to-server integration. Use member keys where you want
per-person attribution and per-person revocation.
