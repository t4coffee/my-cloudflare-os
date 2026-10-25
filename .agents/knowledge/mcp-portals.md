# Cloudflare MCP server portals (essentials)

Product name is **MCP server portals** (formerly "Agents Gateway"; API/Terraform still use
`agents_gateway` / `agw`). Dashboard: **Zero Trust > Access controls > MCP Portals**. It is a
Cloudflare One / Access feature, **not** part of AI Gateway.

Raw doc: `sources/cloudflare-one-mcp-portals.md`.

## How a client connects

1. Client connects to the portal URL `https://<subdomain>.<domain>/mcp`.
2. Gets `401` with `WWW-Authenticate: Bearer ... resource_metadata=".../.well-known/oauth-protected-resource/mcp"`.
3. OAuth discovery → the portal's own Access-managed authorization server; supports **DCR**
   (`registration_endpoint`). Public clients (`token_endpoint_auth_method: none`) work, and
   `http://localhost:...` redirect URIs are accepted.
4. User authenticates through Cloudflare Access (idP) in a browser. Device auth is not supported.
5. `tools/list` returns the enabled upstream servers' tools; calls are proxied with the right
   credential (per-user OAuth grant, or the admin credential).

Transport: stateless MCP `2026-07-28` and earlier 2025 Streamable HTTP at `/mcp`. Protocol is
selected per request.

## Tool naming

- Every upstream tool/prompt is namespaced `{server_id}_{original_name}`; split on the **first**
  underscore only. Server IDs therefore cannot contain underscores (use hyphens).
- Aliases replace the original name but keep the server-id prefix.
- Built-in tools use the `portal_` prefix and are present only on the portal itself:
  `portal_list_servers`, `portal_toggle_servers`, `portal_toggle_single_server`.
- Context optimization can replace upstream tools with `portal_query_tools` / `portal_execute`.
- **Do not** enable Code Mode (or `optimize_context`) for `gatekeeper-mcp-portal`; it requires
  upstream tools exposed directly.

## Access / OAuth for upstream servers

- **Automatic (DCR)** is the recommended upstream OAuth mode; the admin logs in once and that
  account becomes the server's **admin credential**.
- Per-user upstream OAuth: each user authenticates individually; the portal stores it against the
  user's portal grant. "Require user auth" (`on_behalf: true` in the API) forces this.
- Background tool/prompt sync runs ~every 2 hours using admin credentials (DCR servers).

## Service tokens (machine-to-machine)

- Service tokens (`CF-Access-Client-Id` / `CF-Access-Client-Secret`) bypass the browser OAuth flow.
- The token must match a **Service Auth** policy on the portal app *and* on each linked server app.
- **Require user auth** must be **off** for each server the token should reach; `on_behalf: true`
  servers are hidden from service-token sessions.
- The portal then uses the admin credential for every upstream call.

## Context optimization (why the gatekeeper avoids it)

`?optimize_context=...` on the URL reduces token usage but changes the tool surface
(`minimize_tools`, `search_and_execute` add `portal_query_tools` / `portal_execute`). The
gatekeeper needs the plain, flattened `tools/list`, so leave it off.

## Gateway routing

Portal traffic can be routed through Cloudflare Gateway for HTTP logging / DLP (`Gateway > ...`),
either for the whole portal or per server. Optional.

## Practical checks

```sh
curl -i https://<host>/mcp                     # expect 401 + resource_metadata
curl -s https://<host>/.well-known/oauth-protected-resource/mcp
curl -s https://<host>/.well-known/oauth-authorization-server   # issuer, endpoints, DCR
```

MCP clients: Workers AI Playground, MCP Inspector, `npx mcp-remote@latest <url>`. Prefer the
`npx` form over a `serverURL` parameter.
