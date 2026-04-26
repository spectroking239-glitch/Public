# YUMI MUSIC Workspace

## Overview

Pnpm workspace running the YUMI MUSIC Discord bot plus a live web dashboard for it.

## Layout

```
.
├── package.json              # Root: just `start` script
├── pnpm-workspace.yaml       # Workspace config (services/*, artifacts/*)
├── replit.md
├── services/
│   └── aerox-music/          # The Discord bot
│       ├── AeroX/            # Bot source (commands, events, helpers)
│       │   └── helpers/statusWriter.js   # writes database/live_stats.json every 5s
│       ├── database/         # Sequelize SQLite models + aerox_music.db + live_stats.json
│       ├── package.json      # name: @workspace/aerox-music
│       └── README.md
└── artifacts/
    └── dashboard/            # YUMI Music Dashboard (React + Vite)
        ├── server/apiPlugin.ts   # Vite middleware exposing GET /api/stats
        │                          # (reads aerox_music.db via better-sqlite3 + live_stats.json)
        ├── src/lib/api.ts         # typed fetcher + formatters used by the UI
        ├── src/pages/dashboard.tsx
        └── package.json      # name: @workspace/dashboard
```

The folder/code identifiers (`AeroX/`, `aerox_music.db`, `resumeKey: 'AeroXMusicBot'`) are intentionally left as-is — only user-facing strings shown in Discord are branded **YUMI MUSIC**.

## Stack

- Node.js 24, pnpm 10
- discord.js v14
- Poru (Lavalink v4 client)
- Sequelize + SQLite (`better-sqlite3` / `sqlite3`)
- @napi-rs/canvas (now playing card rendering)
- openai (AI chat command, optional)
- genius-lyrics (optional, needs `GENIUS_API_KEY`)

Native deps approved in `pnpm-workspace.yaml` `onlyBuiltDependencies`: `better-sqlite3`, `sqlite3`, `@napi-rs/canvas`.

## Running

- Workflow `AeroX Music Bot` runs `pnpm --filter @workspace/aerox-music run start` (console output, no port).
- Bot prefix: `,` (configurable in `services/aerox-music/AeroX/config.js`).

## Required Secrets

- `BOT_TOKEN` — Discord bot token
- `CLIENT_ID` — Discord application/client ID
- `OWNER_ID` — Discord user ID of the bot owner
- `LAVALINK_HOSTS`, `LAVALINK_PORTS`, `LAVALINK_PASSWORDS`, `LAVALINK_SECURES` — comma-separated lists, one entry per Lavalink node
- `SESSION_SECRET` — session secret (legacy, kept)

Optional: `GENIUS_API_KEY` (for full lyrics).

## Deployment (24/7 hosting)

Discord bots are long-running processes, so they must be deployed as a **Reserved VM**, not Autoscale.

When publishing:
1. Open the Publish tool.
2. Under deployment type, choose **Reserved VM** (always-running). Autoscale will NOT keep a Discord bot online.
3. Set the run command to: `pnpm --filter @workspace/aerox-music run start`
4. Make sure all required secrets above are set in the deployment environment too.
5. Publish.

After publishing, the bot stays online 24/7 independent of the Replit workspace tab.
