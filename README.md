# genifi-docs

Public documentation for the JET cross-border payment rail — the API and the
dashboard.

**Live site:** https://docs.jet.money

Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and
deployed to GitHub Pages on every push to `main`.

## Layout

```
docs/
  README.md          Landing page
  SUMMARY.md         GitBook nav (not part of the MkDocs build)
  CNAME              Custom domain for GitHub Pages
  backend/           API guides and concepts, written by hand
  backend/api-reference/
                     The API reference — GENERATED, do not edit (see below)
  dashboard/         Dashboard guides
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
    curl "$JET_API/health"
    ```

=== "200 OK"

    ```json
    { "status": "ok" }
    ```
````

Link between pages with relative paths including the `.md` extension
(`../concepts/money.md`) so the strict build can verify them.

## Keeping it accurate

### The API reference is generated

`docs/backend/api-reference/` is generated from the backend's source code, along
with the nav block between the `BEGIN/END generated: api-reference` markers in
`mkdocs.yml` and `docs/SUMMARY.md`. **Do not edit those by hand**; the next run
overwrites them. From a checkout of the backend next to this repo:

```bash
cd ../backend
pnpm api:reference            # regenerate into ../genifi-docs
pnpm api:reference --check    # fail if anything is out of date
```

What it reads:

| In the reference | Comes from |
|---|---|
| Which endpoints exist | The API's live route table |
| Request fields and their rules | The Zod schemas the handlers parse |
| Response bodies, `error` codes per status | The TypeScript types of what each handler sends |
| Permission, roles, KYB gate | The gates each handler calls, and `can()` |
| Auth, rate-limit bucket and sizes | The API's auth and rate-limit code |
| Summaries, grouping, field notes | `apps/api/src/reference/catalog.ts` in the backend |
| Webhook event list | The same catalog, type-checked against the event type |

The backend's test suite fails when a route is added, removed or renamed without
a catalog entry, and when its committed `docs/api/openapi.json` is stale. So the
reference cannot silently fall behind the code; it can only be not yet
regenerated here. Regenerate when you update the docs for a release.

### The guides are written by hand

They explain; the reference specifies. Keep field lists and error tables out of
the guides where the reference already has them, and link to it instead. What
still drifts by hand:

| If this changes | Update |
|---|---|
| Currency scales | `docs/backend/concepts/money.md` |
| Payment states | `docs/backend/concepts/payment-lifecycle.md` |
| Webhook payload shapes | `docs/backend/webhooks/events.md` |
| Role descriptions | `docs/backend/getting-started/authentication.md`, `docs/dashboard/guides/team-and-roles.md` |

`validation: anchors` in `mkdocs.yml` makes a link to a heading that no longer
exists fail the build, including the reference's links into the guides.

## GitBook

`.gitbook.yaml` points at `docs/`, so the GitBook space can stay in sync off the
same files. It is a fallback, not the published site — `docs.jet.money` is
served by GitHub Pages.
