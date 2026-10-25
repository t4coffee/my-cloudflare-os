# AI Gateway: essentials

Local reference: `~/.pi/agent/npm/node_modules/pi-cloudflare/skills/cloudflare/references/ai-gateway/`
(README, configuration, features, dynamic-routing, sdk-integration, troubleshooting).
Raw landing page: `sources/ai-gateway.md`.

AI Gateway observes and controls requests to AI providers: analytics, logging, caching, rate
limiting, retries, model fallback, dynamic routing. It is configured per gateway; auth to the
gateway and the upstream provider credentials/billing are separate concerns.

In this repo, Cloudflare OS uses AI Gateway as its model transport when `CF_AI_GATEWAY*` is set
(see `docs/public-server.md` and `docs/ai-gateway-billing.md`; `packages/workshop-backend/src/ai-gateway.ts`
merges admin model settings with the catalog).

## Not to be confused with MCP

**MCP server portals are not an AI Gateway feature.** They live in **Zero Trust > Access
controls > MCP Portals** and use Cloudflare Access + OAuth. Reach a portal from this repo with the
`gatekeeper-mcp-portal` connector (`MCP_PORTAL_URL`), not via AI Gateway. See
[mcp-portals.md](./mcp-portals.md).
