---
name: overlord
description: interact with an overlord server to create tasks, monitor execution, manage projects, and control ai coding agents across your machine fleet
---

# overlord skill

you are connected to an **overlord** server — an ai task orchestration platform that dispatches coding tasks to worker machines running ai agents (claude code, cursor, codex).

## configuration

before using this skill, ensure these environment variables are set:

- `OVERLORD_SERVER` — the overlord server url (e.g. `https://overlord.yourdomain.com`)
- `OVERLORD_TOKEN` — a personal access token (create one at `{server}/settings/tokens`)

if not set, ask the user to provide them.

## how to call the api

use the `WebFetch` tool (preferred) or `Bash` with `curl`. all requests go to `${OVERLORD_SERVER}/api/web/...` with header `Authorization: Bearer ${OVERLORD_TOKEN}`.

**standard pattern:**

```
WebFetch: GET ${OVERLORD_SERVER}/api/web/tasks
Headers: { "Authorization": "Bearer ${OVERLORD_TOKEN}" }
```

for POST/PUT/DELETE, set `Content-Type: application/json` and pass a JSON body.

**response format:** all endpoints return JSON. list endpoints support `?page=1&limit=20` pagination.

---

## api reference

### tasks

#### create a task
```
POST /api/web/tasks
{
  "description": "fix the login bug on the signup page",
  "projectKey": "my-project",
  "machineId": null
}
```
- `description` (required) — what the ai agent should do
- `projectKey` (required) — project key to create the task in
- `machineId` (optional) — target a specific machine, or null for auto-routing

#### list tasks
```
GET /api/web/tasks?status=running&projectKey=my-project&page=1&limit=20
```
- `status` (optional) — filter: `queued`, `assigned`, `running`, `suspended`, `completed`, `failed`, `cancelled`
- `projectKey` (optional) — filter by project

#### get task details
```
GET /api/web/tasks/{id}
```
returns full task info including status, machine assignment, timestamps, pipeline config.

#### get task logs
```
GET /api/web/tasks/{id}/logs
```
returns execution log lines from the task.

#### get task events
```
GET /api/web/tasks/{id}/events
```
returns structured event history (status changes, stage transitions).

#### cancel a task
```
POST /api/web/tasks/{id}/cancel
```

#### retry a failed task
```
POST /api/web/tasks/{id}/retry
```

#### confirm a pipeline stage
```
POST /api/web/tasks/{id}/confirm-stage
{ "stage": 1, "result": "confirm" }
```
- `stage` — stage index (0-based)
- `result` — `"confirm"` to approve, or a string value for choice/input stages

#### get pending confirmation info
```
GET /api/web/tasks/{id}/pending-confirm
```

---

### projects

#### list projects
```
GET /api/web/projects
```

#### get project details
```
GET /api/web/projects/{key}
```

#### update a project
```
PUT /api/web/projects/{key}
{ "name": "new name", "repoUrl": "https://...", "sshUrl": "git@..." }
```

#### list project members
```
GET /api/web/projects/{key}/members
```

#### add a project member
```
POST /api/web/projects/{key}/members
{ "developerId": 5, "role": "developer" }
```
- `role` — `"maintainer"` or `"developer"`

#### remove a project member
```
DELETE /api/web/projects/{key}/members/{developerId}
```

---

### machines

#### list machines
```
GET /api/web/machines
```

#### get machine details
```
GET /api/web/machines/{id}
```

---

### notifications

#### list notifications
```
GET /api/web/notifications?all=true
```
- `all` (optional) — include read notifications

#### mark notification as read
```
POST /api/web/notifications/{id}/read
```

---

### search

#### global search
```
GET /api/web/search?q=login+bug
```
returns matching tasks, projects, and machines.

---

### status & profile

#### current user
```
GET /api/web/profile
```

#### cluster dashboard stats
```
GET /api/web/dashboard/stats
```
returns active/queued task counts, online workers, completed today.

---

### pty terminal (advanced)

to attach to a running task's terminal:

1. get a pty token:
```
POST /api/web/tasks/{taskId}/pty-token
```

2. connect via websocket to `${OVERLORD_SERVER}` with the returned channel info.

> note: pty attachment is complex (websocket + xterm protocol). for most use cases, reading task logs via the REST api is sufficient.

---

## task statuses

| status | meaning |
|--------|---------|
| `queued` | waiting for a worker machine |
| `assigned` | assigned to a machine, starting up |
| `running` | ai agent is executing |
| `suspended` | paused — awaiting human confirmation or worker reconnection |
| `completed` | finished successfully |
| `failed` | execution failed |
| `cancelled` | cancelled by user |

---

## common workflows

### create and monitor a task
1. create a task with a description and project key
2. poll `GET /api/web/tasks/{id}` until status is `completed` or `failed`
3. if `suspended`, check `GET /api/web/tasks/{id}/pending-confirm` and confirm if appropriate

### check cluster health
1. call `GET /api/web/dashboard/stats` for overview
2. call `GET /api/web/machines` to see individual machine status

### find a task
1. use `GET /api/web/search?q=...` for broad search
2. or `GET /api/web/tasks?status=running` for filtered listing

---

## error handling

- **401** — token is invalid or expired. ask the user to create a new token.
- **403** — insufficient permissions. user may not have access to this project/resource.
- **404** — resource not found.
- **429** — rate limited. wait and retry.

## tips

- always check if `OVERLORD_SERVER` and `OVERLORD_TOKEN` are set before making requests
- use `--json` style: prefer structured data over rendering tables yourself
- when creating tasks, write clear descriptions — the ai agent on the other end will execute exactly what you describe
- for long-running tasks, poll status periodically rather than blocking
