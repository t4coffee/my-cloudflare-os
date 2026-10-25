# Site: t4coffee (Cloudflare account)

Site-specific facts gathered 2026-10-13. No secrets here; the API token lives in 1Password at
`op://Personal/Cloudflare Account Token` (`pi-macbook-air-read-only`).

- Account ID: `000f5a0c6d473d0c42a415cea255cb54`
- AI Gateway: **`commandcentre-aigw`** — `authentication: true` (needs a token for inference),
  `byok_only: false`, unified Workers AI billing. Account has no BYOK keys recorded here.
- MCP server portal: id/name **`commancentre-mcp`** (sic), hostname `mcp.t4coffee.com`,
  `code_mode: default_on`, `secure_web_gateway: true`. Endpoint:
  `https://mcp.t4coffee.com/mcp?codemode=off`.
- Upstream MCP servers: `slack-mcp` (https://mcp.slack.com/mcp), `gmail-mcp`
  (https://gmailmcp.googleapis.com/mcp/v1), `drive-mcp` (https://drivemcp.googleapis.com/mcp/v1),
  `calendar-mcp` (https://calendarmcp.googleapis.com/mcp/v1) — all `auth_type: oauth` with the
  shared Cloudflare OAuth callback enabled.

## Token scopes

`pi-macbook-air-read-only` has AI Gateway **Read**, MCP Portals Read, Workers AI Read, Workers
Scripts Read, etc. — **no AI Gateway Run**, so it cannot serve as `CF_AI_GATEWAY_API_TOKEN`.
`wrangler whoami` reports not authenticated.

## Inspect it (read-only)

```sh
CF_TOKEN="$(op read 'op://Personal/Cloudflare Account Token/credential')"
A=000f5a0c6d473d0c42a415cea255cb54
curl -s -H "Authorization: Bearer $CF_TOKEN" "https://api.cloudflare.com/client/v4/accounts/$A/ai-gateway/gateways"
curl -s -H "Authorization: Bearer $CF_TOKEN" "https://api.cloudflare.com/client/v4/accounts/$A/access/ai-controls/mcp/portals"
curl -s -H "Authorization: Bearer $CF_TOKEN" "https://api.cloudflare.com/client/v4/accounts/$A/access/ai-controls/mcp/servers" \
  | jq -c '.result[] | {id, name, hostname, auth_type}'
```
