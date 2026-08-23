# Signing in

## The front door is a magic link

1. Go to the sign-in screen.
2. Enter your **work email**.
3. Open the email we send and click the link.

That's it — no password to choose, forget, or reuse.

Where your organization has it enabled, **Continue with Google** works too, for
the same email address.

## You have to be invited first

Your email must belong to an **active member** of an organization. Signing in
with an address nobody has invited will not create an account.

If you are new, ask an admin or the owner of your organization to invite you from
the **Team** page. You will get an invitation email with an accept link; once you
accept, magic-link sign-in works for that address.

See [Your team](../guides/team-and-roles.md) if you are the one doing the
inviting.

## Your session

Signing in sets a secure session cookie in your browser. It is scoped to you as a
person — which is what makes per-user features like
[email notifications](../guides/notifications.md) possible.

Signing out ends it. Signing in on another device gives you a separate session;
they do not interfere.

## What your role lets you do

Every member has a role, set when they were invited and changeable by an admin.

| | Viewer | Member | Admin | Owner |
|---|:---:|:---:|:---:|:---:|
| See payments and accounts | ✅ | ✅ | ✅ | ✅ |
| See the address book | ✅ | ✅ | ✅ | ✅ |
| Send payments | — | ✅ | ✅ | ✅ |
| Add and edit recipients | — | ✅ | ✅ | ✅ |
| Manage webhooks | — | — | ✅ | ✅ |
| Invite and manage team members | — | — | ✅ | ✅ |
| Rotate the organization API key | — | — | — | ✅ |

Actions your role does not allow are not hidden — they are visible and disabled,
so you can see what exists and know who to ask.

## Signing in with an API key

Under **Advanced** on the sign-in screen there is an option to sign in with an
API key instead.

This is for integrations and local development. An API key identifies your
**organization**, not you — so per-user features (notification preferences) are
unavailable in that mode, because there is no person to attach them to.

!!! warning

    An API key can move money. Treat it like a production credential: don't paste it
    into a shared browser, and rotate it from
    [API keys and webhooks](../guides/api-keys-and-webhooks.md) if you think it has
    been seen.

## Just want to look around?

You do not need an account. Click **Explore the demo — no API key needed** on the
sign-in screen. See [Demo mode](demo-mode.md).

## Trouble signing in?

| Symptom | Likely cause |
|---|---|
| No email arrives | The address is not an active member of any organization, or the mail is in spam |
| "Invalid or expired link" | Magic links are single-use and time-limited. Request a fresh one |
| Signed in, but everything is read-only | Your role is `viewer`. Ask an admin to change it |
| Signed out unexpectedly | The session expired, or you signed out in another tab |

More in [Troubleshooting](../reference/troubleshooting.md).
