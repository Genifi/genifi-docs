# Your team

The **Team** page is where you invite colleagues, set what they can do, and
revoke access.

Viewing and inviting need the `admin` role or higher.

## Roles

| | Viewer | Member | Admin | Owner |
|---|:---:|:---:|:---:|:---:|
| See payments and accounts | ✅ | ✅ | ✅ | ✅ |
| See the address book | ✅ | ✅ | ✅ | ✅ |
| Request quotes and send payments | — | ✅ | ✅ | ✅ |
| Add and edit recipients | — | ✅ | ✅ | ✅ |
| Manage webhooks | — | — | ✅ | ✅ |
| View and invite team members | — | — | ✅ | ✅ |
| Change and revoke roles | — | — | ✅ | ✅ |
| Rotate the organization API key | — | — | — | ✅ |

**Owner** is the organization itself. It is never assignable — you cannot grant
it, and you cannot lose it by removing people. That means there is no way to lock
your organization out of its own account by revoking the wrong member.

Pick the smallest role that lets someone do their job. `viewer` is genuinely
useful for finance and support people who need to see payments but should never
be able to start one.

## Inviting someone

Invite by email address and choose their role.

They get an email with an accept link. Once they accept, they can sign in with a
[magic link](../getting-started/signing-in.md) at that address, and they receive
a personal API key.

### The invite link goes to them, and only them

You will not see the invite token — not even as the person who sent it. It is a
credential for someone else's account, so it goes to their inbox and nowhere
else.

### You cannot grant a role above your own reach

An `admin` cannot create another `admin`; only the `owner` can. Attempting it
gives you a clear refusal rather than quietly downgrading the invitation.

## Resending an invitation

If an invitation never arrived — a typo, a spam filter, a bounced address — use
**Resend**. It issues a **fresh** token and kills the old one.

This is not just a convenience. Invite tokens are stored hashed and shown once,
and each address may hold only one invitation. Without a resend path, a bounced
invitation would leave a member who can never be onboarded and never re-invited
at that address.

## Changing a role

Change it in place. It takes effect on their next request.

The same rule applies: you cannot set a role you are not allowed to grant.

## Removing someone

Removing a member who has **accepted** revokes their access — their session and
their personal API key stop working.

Removing a member who has **not yet accepted** rescinds the invitation. The link
they were sent stops working.

The dashboard tells you which of the two happened.

!!! info

    Removing a member does not affect the **organization's** API key. If the person
    leaving had access to it, [rotate it](api-keys-and-webhooks.md#rotating) as well.
    That is a separate, deliberate step — it is the credential your integration runs
    on.

## Member statuses

| Status | Meaning |
|---|---|
| **Invited** | Invitation sent, not yet accepted |
| **Active** | Accepted and able to sign in |

Each member also shows when they were invited and when they accepted, which is
usually enough to answer "did this ever go out?" without asking.
