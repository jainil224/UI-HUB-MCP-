# Keeping the MCP Server Awake

The MCP server is deployed to Render (free tier). Free instances **spin down after ~15 minutes of idle time** and take several minutes to boot again on the next request. That is why `/dashboard/mcp` and `/admin/mcp/*` pages can appear to hang after a quiet period.

This repo ships **three layers** of keep-alive so the server never enters sleep mode:

| Layer | Mechanism | Coverage |
| --- | --- | --- |
| 1 | GitHub Actions cron (`/.github/workflows/keep-alive.yml`) pings `/health` every 5 min | 24/7, no user traffic needed |
| 2 | [cron-job.org](https://cron-job.org) external monitor (set up below) | 24/7, independent of GitHub |
| 3 | Frontend `useMcpKeepAlive` hook pings `/health` every 2 min while the site is open | Active browsing |

The ping target is `GET https://api.ui-hub-design.com/health`. It is deliberately public (no auth, no secrets, no rate-limit impact).

---

## 1. GitHub Actions (already in this repo)

`.github/workflows/keep-alive.yml` runs on a `*/5 * * * *` schedule and curls the health endpoint with a 30s timeout. Nothing to configure — it activates as soon as this repo is on GitHub.

> Note: GitHub pauses scheduled workflows after 60 days with no repository activity. Any commit or PR wakes it back up.

## 2. cron-job.org (recommended external monitor)

1. Create a free account at https://cron-job.org → **Create Cron Job**.
2. Set **URL** to `https://api.ui-hub-design.com/health`.
3. Method: **GET**.
4. **Interval**: 1–2 minutes (plenty below Render's 15-minute idle limit). Every minute gives near-zero wake latency.
5. Attach your email for **Failure Notifications** (alerts when the ping fails or is slow).

### Alternative: UptimeRobot

1. Free account at https://uptimerobot.com → **+ New Monitor**.
2. Monitor Type: **HTTP(S)**, URL `https://api.ui-hub-design.com/health`.
3. Interval **5 minutes** (the free minimum), add your email for alerts.
4. Use this as a secondary monitor alongside cron-job.org if you want redundancy.

## 3. Frontend hook (already deployed with the website)

`frontend/src/hooks/useMcpKeepAlive.ts` fires `GET <VITE_MCP_API_URL>/health` every 2 minutes while the tab is visible and online, and is mounted on:

- `/dashboard/mcp` (`MCPPage.tsx`)
- all `/admin/mcp/*` pages (`AdminLayout.tsx`)

The API clients also retry once (~1.5s) on network errors / 502 / 504 so a rare cold boot degrades to a brief spinner instead of a dead page.

---

## Verifying it works

After deployment, confirm the endpoint responds:

```bash
curl -i https://api.ui-hub-design.com/health
```

Expected: `200` with `{"status":"ok","service":"ui-hub-mcp"}`. Wait >15 minutes with no pings active, then load `/admin/mcp/overview` in the app — it should respond within a few seconds thanks to the keep-alive.