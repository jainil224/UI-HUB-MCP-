# UI HUB MCP Server

Independent Model Context Protocol (MCP) server for the [UI HUB](https://ui-hub-design.vercel.app) AI-accessible UI component platform.

This server lets AI coding assistants (Cursor, Claude Code, VS Code/Copilot, and other MCP clients) search, discover, and retrieve UI HUB components, source code, dependencies, templates, and animations.

## Quick Start (local)

```bash
npm install
# copy .env.example to .env and fill in Firebase credentials
npm run build
npm start
```

Or in development:

```bash
npm run dev
```

## Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /health` | Health check (`{"status":"ok","service":"ui-hub-mcp"}`) |
| `POST /mcp` | MCP Streamable HTTP endpoint (JSON-RPC) |
| `GET /mcp` | MCP discovery / tool list |
| `GET /api/dashboard/mcp/*` | Dashboard API-key management routes (Firebase token auth) |

## Environment Variables

See [.env.example](./.env.example). Key variables:

- `MCP_SERVER_URL` — public base URL of this service (e.g. `https://ui-hub-mcp.onrender.com`)
- `MCP_API_KEY_PREFIX` — default `uh_live_`
- `MCP_RATE_LIMIT_FREE` / `MCP_RATE_LIMIT_PRO` — daily request limits per plan
- `FIREBASE_PROJECT_ID` / `FIREBASE_CLIENT_EMAIL` / `FIREBASE_PRIVATE_KEY` — Firebase Admin SDK credentials
- `REDIS_URL` — (optional) Upstash Redis for distributed rate limiting
- `MCP_ADMIN_EMAILS` — comma-separated admin email overrides
- `MCP_ALLOWED_ORIGINS` — comma-separated CORS origins

Port: defaults to `PORT` (Render sets this automatically), falling back to `MCP_PORT` then `3001`.

## Deploy on Render

This repo is designed to be deployed independently on Render as a long-running Node.js web service.

1. In Render, create a **New Web Service** and connect this repository.
2. Render auto-detects the Node runtime.
3. Set the **Build Command**: `npm install && npm run build`
4. Set the **Start Command**: `node dist/index.js`
5. Set the **Health Check Path**: `/health`
6. Add the environment variables listed above.
7. Optionally, attach the included `render.yaml` blueprint.

Or use Render's **Blueprint** (Infrastructure as Code) with the included `render.yaml`:

```bash
render blueprint create --file render.yaml
```

## MCP Tools

- `search_components`
- `get_component`
- `get_component_code`
- `search_templates`
- `get_template`
- `search_animations`
- `get_animation_code`
- `list_categories`
- `get_dependencies`

Full documentation: see `docs/mcp.md` in the main UI HUB repository, or the in-repo `docs` structure.

## Authentication

API keys use the format `uh_live_xxxxxxxxxxxxxxxxxxxxxxxxx`, sent as:

```
Authorization: Bearer uh_live_xxxxxxxxxxxxxxxxxxxxxxxxx
```

Keys are stored as SHA-256 hashes in the Firestore collection `mcp_api_keys`.

## Testing

```bash
npm test
```

40 tests covering auth, permissions, API-key handling, rate limiting, and all MCP tools.
