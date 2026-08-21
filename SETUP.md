# Publishing to GitBook

This file is **not** part of the documentation — it is not listed in
`SUMMARY.md`, so GitBook will not publish it.

## Layout

One GitBook space, one sidebar, two top-level groups:

```
genifi-docs/
  .gitbook.yaml        Git Sync config — must be at the repo root
  SUMMARY.md           The sidebar. Two "## " groups = the two brackets
  README.md            Site landing page
  backend/             Group 1 — Backend API (24 pages)
  dashboard/           Group 2 — Dashboard (14 pages)
```

The two groups come from the `##` headings in `SUMMARY.md`. Adding a page means
adding the file **and** a line in `SUMMARY.md` — GitBook publishes only what the
summary lists.

## Option A — Git Sync (recommended)

Keeps the docs in version control and lets you edit either in GitBook or in the
repo; changes flow both ways.

1. Make this folder a repository and push it:

   ```bash
   cd genifi-docs
   git init -b main
   git add .
   git commit -m "docs: initial Genifi documentation"
   git remote add origin git@github.com:<org>/genifi-docs.git
   git push -u origin main
   ```

2. In your GitBook space: **Configure → Git Sync → GitHub/GitLab**.
3. Pick the repository and the `main` branch.
4. Leave the project directory blank — `.gitbook.yaml` is at the repo root.
5. Choose **GitBook ← repository** for the first sync, so the repo content is
   what lands. After that, either direction works.

## Option B — Import without a repository

If you would rather not set up Git Sync yet:

1. Zip this folder.
2. In the space: **⋯ → Import → Upload a file**.
3. GitBook reads `SUMMARY.md` and builds the sidebar from it.

You can add Git Sync later without redoing the content.

## After the first sync

* **Publish the site.** A space is private until you publish it, and a site can
  hold this space plus anything you add later.
* **Fill in the base URL.** Examples use `$GENIFI_API` as a placeholder. If you
  want a literal host in the docs, replace it in
  `backend/getting-started/quickstart.md` and `backend/api/conventions.md`.
* **Check the dashboard URL.** The dashboard pages describe routes
  (`/dashboard/payments`) but never name a host, so nothing needs changing unless
  you want to add one.

## Scope

These docs are **external/public**: written for integrators and dashboard users.
Deliberately excluded — repo layout, workflow engine internals, ledger
invariants, infrastructure and deploy, internal ops surfaces and ports, provider
vendor names, environment variables, and roadmap status.

Two things were left out as internal-only even though they exist in the API:

* `POST /wallets/:walletId/topup-eurc` — a simulated top-up for development.
* `GET /metrics` is mentioned only as "not part of the integration surface".

## Keeping it accurate

The API pages were written against the current request/response schemas. The
things most likely to drift:

| If this changes | Update |
|---|---|
| Currency scales | `backend/concepts/money.md` |
| Corridor list | `backend/concepts/corridors.md` (it already tells readers to use `/capabilities`) |
| Rate-limit defaults | `backend/reference/rate-limits.md` |
| Role permission table | `backend/getting-started/authentication.md`, `dashboard/guides/team-and-roles.md` |
| Webhook event list | `backend/webhooks/events.md`, `backend/api/notifications.md`, `dashboard/guides/api-keys-and-webhooks.md` |
| A `501` endpoint going live | `backend/api/wallets-and-accounts.md`, `dashboard/reference/feature-availability.md` |
