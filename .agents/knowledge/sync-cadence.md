# Syncing with upstream (cloudflare/cloudflare-os)

## Remotes

- `origin` → `https://github.com/cloudflare/cloudflare-os.git` — upstream, **fetch only, never push**.
- `personal` → `https://github.com/t4coffee/my-cloudflare-os.git` — push target for `main`.

## Cadence

- **Weekly**: fetch upstream and merge. The upstream moves fast (~70 commits in two months at one point); monthly syncs get painful.
- **Monthly** (or after a big feature lands): also run the test suite.

## Weekly sync

```sh
git fetch origin
git merge origin/main        # or: git rebase origin/main to keep local commits on top
pnpm install                 # lockfile usually changes upstream
pnpm build                   # type check + codegen; all tasks must pass
```

Notes:

- `git merge --ff-only origin/main` works only when local `main` has no commits ahead; with local commits (the normal case) it refuses — use a plain `git merge` or `git rebase origin/main`.
- Prefer rebase if you want a linear history of local-only commits on top of upstream.

## Likely conflict points

Local customizations live in these files — re-check their diffs on every merge:

- `.gitignore` (offline agent sources, `google_client.json`)
- `AGENTS.md` (knowledge-notes reference)
- `package.json` / `pnpm-lock.yaml` (pnpm 12 bump)

## Never commit

- `google_client.json` — contains a Google OAuth client secret (ignored, verify before any manual `git add`).