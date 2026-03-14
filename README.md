# overlord skill

an ai coding agent skill for [overlord](https://overlord.run) — ai agent fleet management.

works with any ai coding agent that supports skills: claude code, cursor, windsurf, codex, and more.

lets your ai assistant create tasks, monitor execution, manage projects, and interact with your overlord server directly.

## install

```bash
npx skills add overlord-run/skill
```

## setup

set these environment variables before using:

```bash
export OVERLORD_SERVER=https://overlord.yourdomain.com
export OVERLORD_TOKEN=your-personal-access-token
```

create a token at `{your-server}/settings/tokens`.

## what it can do

- **create tasks** — describe what you want done, overlord dispatches to a worker
- **monitor tasks** — check status, read logs, track progress
- **manage projects** — list projects, view members, manage git tokens
- **manage workers** — check status, drain/undrain workers
- **workspace tunnels** — start/stop remote workspace connections
- **search** — find tasks, projects, workers
- **confirm stages** — approve pipeline stages for suspended tasks
- **profile & tokens** — manage your profile and api tokens

## learn more

- [overlord documentation](https://overlord.run/docs)
- [api reference](https://overlord.run/docs/reference/api)
