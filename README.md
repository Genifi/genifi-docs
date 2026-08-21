# genifi-docs

Public documentation for the Genifi cross-border payment rail — the API and the
dashboard.

**Live site:** https://docs.genifi.com

Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and
deployed to GitHub Pages on every push to `main`.

## Layout

```
docs/
  README.md          Landing page
  SUMMARY.md         GitBook nav (not part of the MkDocs build)
  CNAME              Custom domain for GitHub Pages
  backend/           API documentation — 24 pages
  dashboard/         Dashboard guides — 14 pages
mkdocs.yml           Site config and navigation
requirements.txt     Pinned build dependencies
```

Navigation lives in the `nav:` block of `mkdocs.yml`. **Adding a page means
adding the file and a `nav:` entry** — the build runs with `strict: true`, so an
orphaned page or a broken internal link fails CI rather than shipping quietly.

## Working on the docs

```bash
pip install -r requirements.txt
mkdocs serve            # http://127.0.0.1:8000, live reload
```

Before opening a PR:

```bash
mkdocs build            # same strict build CI runs
```

Every pull request builds the site as a check. Merging to `main` deploys.

## Writing conventions

Callouts use Material admonitions:

```markdown
!!! warning

    Rotating a key invalidates the old one immediately.
```

Request/response pairs use content tabs:

````markdown
=== "Request"

    ```bash
    curl "$GENIFI_API/health"
    ```

=== "200 OK"

    ```json
    { "status": "ok" }
    ```
````

Link between pages with relative paths including the `.md` extension
(`../concepts/money.md`) so the strict build can verify them.

## Keeping it accurate

These pages are written against the real API contract, not summarised from
prose. The things most likely to drift:

| If this changes | Update |
|---|---|
| Currency scales | `docs/backend/concepts/money.md` |
| Corridor list | `docs/backend/concepts/corridors.md` |
| Rate-limit defaults | `docs/backend/reference/rate-limits.md` |
| Role permissions | `docs/backend/getting-started/authentication.md`, `docs/dashboard/guides/team-and-roles.md` |
| Webhook events | `docs/backend/webhooks/events.md`, `docs/backend/api/notifications.md` |
| An endpoint leaving `501` | `docs/backend/api/wallets-and-accounts.md`, `docs/dashboard/reference/feature-availability.md` |

## GitBook

`.gitbook.yaml` points at `docs/`, so the GitBook space can stay in sync off the
same files. It is a fallback, not the published site — `docs.genifi.com` is
served by GitHub Pages.
