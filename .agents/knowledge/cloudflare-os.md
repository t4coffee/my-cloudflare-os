# Cloudflare OS: running locally, and connecting an MCP portal

## `pnpm run-local`

Entry point `scripts/run-local.ts`:

1. `pnpm install`
2. builds `@gadgets/typed-storage` (backend imports its `dist`) and the frontend bundle
   (`@gadgets/workshop-frontend#build:assets`)
3. spawns `scripts/run-dev-server.ts --serve-frontend-assets`

Result: the whole stack on `http://localhost:8787`. Data lives under `.wrangler/`.
Sign in with the built-in username/password (enabled unless `DISABLE_PASSWORD_AUTH=true`; dev also
sets `ADMINS=["admin"]`). See `docs/public-server.md`.

Normal dev (Vite HMR + backend split) is `pnpm dev-server`; the frontend is on :3000.

## How the dev server wires gatekeepers

`scripts/run-dev-server.ts`:

- regenerates every `wrangler.jsonc` from each `cloudflare.config.ts`
  (`scripts/generate-worker-configs.ts`), then writes gitignored `*.dev.jsonc` files
  (root `wrangler.dev.jsonc`, backend, and every gatekeeper) that add service bindings.
- discovers gatekeepers purely by scanning `packages/gatekeeper-*` for a `wrangler.jsonc`
  (`findGatekeepers`). A library package with no `wrangler.jsonc` is not started.
- sets each gatekeeper's `BASE_URL` to `http://localhost:8787/gatekeeper/<name-without-prefix>`.

## `.dev.vars`

Root `.dev.vars` (gitignored) is loaded by `run-dev-server.ts` (`loadDevVars`); shell env wins.
`PASSTHROUGH_GATEKEEPER_VARS` forwards listed vars into the matching gatekeeper's dev config.
Notably `gatekeeper-mcp-portal` passes through:
`MCP_PORTAL_URL`, `MCP_PORTAL_NAME`, `MCP_PORTAL_AUTH`, `MCP_PORTAL_TOKEN`,
`MCP_PORTAL_TRUST_ANNOTATIONS`, `MCP_PORTAL_HIDDEN_SERVER_IDS`, `MCP_ALLOW_INSECURE`.

So pointing the local OS at a portal needs **no code change**, only root `.dev.vars`:

```
MCP_PORTAL_URL=https://mcp.t4coffee.com/mcp
# MCP_PORTAL_AUTH defaults to "oauth"; use "token" + MCP_PORTAL_TOKEN for a service-token portal.
```

If the portal's `code_mode` is `default_on`, append `?codemode=off` (the connector needs the
upstream tools exposed directly; see `mcp-portals.md`). This machine's portal is `default_on`,
so its URL is `https://mcp.t4coffee.com/mcp?codemode=off`.

## Connecting to an MCP server portal (local)

1. Put `MCP_PORTAL_URL` in `.dev.vars` (above). Default auth is `oauth`.
2. `pnpm run-local`, open `http://localhost:8787`, sign in.
3. Connectors page → connect **MCP Server Portal** → completes the portal's Cloudflare Access
   login in a browser popup, then lists the upstream servers.
4. Grant access to a Gadget/agent by picking **one server** behind the portal (optionally a subset
   of its tools). A portal grant never spans the whole portal.

Key files:

- `packages/gatekeeper-mcp-portal/README.md` — full connector behavior, config, repoint semantics.
- `packages/gatekeeper-mcp-portal/src/config.ts` — reads/validates `MCP_PORTAL_*`.
- `packages/mcp-shared/src/account.ts` — OAuth; redirect URI is `${BASE_URL}/oauth`, i.e.
  `http://localhost:8787/gatekeeper/mcp-portal/oauth`. DCR registers it dynamically.
- `packages/gatekeeper-mcp/README.md` — the connector for user-supplied (non-portal) endpoints.

Notes:

- The portal must expose upstream tools directly: do not append `optimize_context`/Code Mode, and
  do not add `?codemode=...` (the connector README forbids it).
- `MCP_ALLOW_INSECURE` is only for `http://` / private hosts; a public `https://` portal does not
  need it (and the committed default is `"false"`).
- Leave `MCP_PORTAL_TRUST_ANNOTATIONS` off (the default): a portal aggregates servers the admin
  did not review, so their tool annotations must not drive auto-approval.
- Changing `MCP_PORTAL_URL` on a deployment with connected accounts is a *repoint*: old bindings
  fail closed and users reconnect.
