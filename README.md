# overlord skill

a [claude code](https://claude.com/claude-code) skill for [overlord](https://overlord.run) — ai task orchestration platform.

lets claude code create tasks, monitor execution, manage projects, and interact with your overlord server directly.

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

- **create tasks** — describe what you want done, overlord dispatches to a worker machine
- **monitor tasks** — check status, read logs, track progress
- **manage projects** — list projects, view members
- **check cluster** — see machine status, worker health
- **search** — find tasks, projects, machines
- **confirm stages** — approve pipeline stages for suspended tasks

## learn more

- [overlord documentation](https://overlord.run/docs)
- [api reference](https://overlord.run/docs/reference/api)
