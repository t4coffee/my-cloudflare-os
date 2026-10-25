# Agent knowledge base

Local, offline notes so an agent does not have to re-fetch Cloudflare docs for routine work on
this repo. Each file is a short summary plus links to the authoritative source. Raw fetched docs
live in `sources/` (gitignored; regenerate from the links when missing).

| File | Covers |
| --- | --- |
| [cloudflare-os.md](./cloudflare-os.md) | How this repo runs locally, `.dev.vars`, gatekeeper discovery, MCP portal connection |
| [mcp-portals.md](./mcp-portals.md) | Cloudflare MCP server portals: OAuth, namespacing, built-in tools, Code Mode, service tokens |
| [wrangler.md](./wrangler.md) | Wrangler essentials + where this repo generates its configs |
| [ai-gateway.md](./ai-gateway.md) | AI Gateway essentials (and why MCP portals are not AI Gateway) |
| [site-t4coffee.md](./site-t4coffee.md) | This machine's account/gateway/portal facts (no secrets) |

Authoritative sources, already installed on this machine (no network needed):

- Cloudflare skill: `~/.pi/agent/npm/node_modules/pi-cloudflare/skills/cloudflare/SKILL.md`
- Wrangler skill: `~/.pi/agent/npm/node_modules/pi-cloudflare/skills/wrangler/SKILL.md`
- AI Gateway references: `~/.pi/agent/npm/node_modules/pi-cloudflare/skills/cloudflare/references/ai-gateway/`
- Cloudflare One skill: `~/.pi/agent/npm/node_modules/pi-cloudflare/skills/cloudflare-one/SKILL.md`

Capture notes on: 2026-10-13. Cloudflare docs change; re-fetch anything load-bearing.
