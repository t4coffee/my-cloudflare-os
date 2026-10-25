# Wrangler: essentials for this repo

Authoritative local reference: `~/.pi/agent/npm/node_modules/pi-cloudflare/skills/wrangler/SKILL.md`.
Raw docs: `sources/wrangler-commands.md`, `sources/wrangler-configuration.md`.
Always prefer the project's pinned version: `node_modules/wrangler/config-schema.json` and
`wrangler <command> --help`.

## Version / invocation

- Package manager is **pnpm** (never npm). Wrangler is a workspace devDependency (`catalog:`).
- Run it through the repo's scripts or `pnpm exec wrangler ...` so the pinned version wins.

## This repo's config model (important)

- Each Worker's `cloudflare.config.ts` (`@cloudflare/config` shape, built from
  `scripts/worker-config.ts`) is the source of truth.
- The `wrangler.jsonc` beside it is **generated** by `pnpm configs:generate`
  (`scripts/generate-worker-configs.ts`) and must not be hand-edited.
  `pnpm configs:check` (in `pnpm lint` and CI) fails on a stale or hand-written file.
- `pnpm dev-server` regenerates before starting.
- Settings `@cloudflare/config` has no field for (module `rules`, KV `preview_id`,
  `assets.directory`, `build`) go in the config's `wrangler` export; DO migrations stay in its
  `migrations` export.

## Commands seen in this repo

- `wrangler dev -c <config>...` drives the multi-worker local stack (see
  `scripts/run-dev-server.ts`). Local state under `.wrangler/`.
- `wrangler deploy` is run per package (e.g. via its `deploy` script).
- Release builds use wrangler dry-run (`scripts/release/build-release.ts`), not `deploy`.

## Local vars / secrets

- Root `.dev.vars` (gitignored) is the local var/secret file; `run-dev-server.ts` loads it and
  forwards selected keys to gatekeeper dev configs (see `cloudflare-os.md`).
- `.dev.vars` is not uploaded as a deployed secret — production secrets are separate.

## Caveats

- Config changes are not always inherited by environments; check each environment.
- `wrangler deploy`/`secret put` create a deployment immediately; not a test.
- The repo pins a wrangler version and uses generated configs — do not upgrade Wrangler to match
  upstream docs.
