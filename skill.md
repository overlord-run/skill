---
name: overlord
description: create and monitor tasks, manage projects and workers, open remote workspaces, and search across your overlord ai agent fleet
---

# overlord skill

you are connected to an **overlord** server — an ai agent fleet management platform that dispatches coding tasks to worker machines running ai agents (claude code, cursor, codex).

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

**response format:** all endpoints return JSON. list endpoints support `?limit=20&cursor=xxx` cursor-based pagination.

---

## api reference

### tasks

#### create a task
```
POST /api/web/tasks
{
  "description": "fix the login bug on the signup page",
  "projectKey": "my-project",
  "workerId": null
}
```
- `description` (required) — what the ai agent should do
- `projectKey` (required) — project key to create the task in
- `workerId` (optional) — target a specific worker (UUID), or null for auto-routing
- `developerId` (optional) — assign to a specific developer

#### list tasks
```
GET /api/web/tasks?status=running&projectKey=my-project&limit=20&cursor=xxx
```
- `status` (optional) — filter: `queued`, `assigned`, `running`, `suspended`, `completed`, `failed`, `cancelled`
- `projectKey` (optional) — filter by project
- admins see all tasks; regular users see only their own

#### get task details
```
GET /api/web/tasks/{id}
```
returns full task info including status, worker assignment, agent type, timestamps, pipeline config.

#### get task logs
```
GET /api/web/tasks/{id}/logs?limit=100&offset=0
```
- `limit` (optional) — max lines to return (max 1000)
- `offset` (optional) — skip N lines from the start

#### get task events
```
GET /api/web/tasks/{id}/events?type=stage_change&search=deploy
```
- `type` (optional) — filter by event type
- `search` (optional) — search within events

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
{ "stageName": "deploy", "approved": true }
```
- `stageName` (required) — name of the stage to confirm
- `approved` (required) — `true` to approve, `false` to reject

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
returns projects with metadata: task count, member count, members preview, last activity.

#### get project details
```
GET /api/web/projects/{key}
```

#### update a project
```
PUT /api/web/projects/{key}
{ "name": "new name", "repoUrl": "https://...", "sshUrl": "git@..." }
```
requires maintainer role on the project.

#### list project members
```
GET /api/web/projects/{key}/members
```
requires maintainer role.

#### add a project member
```
POST /api/web/projects/{key}/members
{ "developerId": 5, "role": "member" }
```
- `role` (optional) — `"maintainer"` or `"member"` (default: `"member"`)

#### remove a project member
```
DELETE /api/web/projects/{key}/members?developerId=5
```
- `developerId` (query param) — the developer id to remove

#### list all developers
```
GET /api/web/projects/-/developers
```
returns all developers in the system. useful when adding members to a project.

#### set project git token
```
PUT /api/web/projects/{key}/git-token
{ "token": "glpat-xxx" }
```
requires maintainer role. token is stored encrypted.

#### delete project git token
```
DELETE /api/web/projects/{key}/git-token
```

---

### workers

#### list workers
```
GET /api/web/workers?status=online
```
- `status` (optional) — filter by worker status

#### get worker details
```
GET /api/web/workers/{id}
```
worker ids are UUIDs.

#### get worker tasks
```
GET /api/web/workers/{id}/tasks
```
returns tasks assigned to this worker.

#### drain a worker
```
POST /api/web/workers/{id}/drain
```
prevents new task assignment. requires lead role or above.

#### undrain a worker
```
POST /api/web/workers/{id}/undrain
```
restores worker to online. requires lead role or above.

---

### workspace tunnels

remote workspace access via cursor remote tunnel.

#### get workspace info
```
GET /api/web/workspaces/{taskId}
```

#### start tunnel
```
POST /api/web/workspace-tunnel/{taskId}/start
```
starts a cursor remote tunnel for the task's workspace. only task owner or admin can start.

#### stop tunnel
```
POST /api/web/workspace-tunnel/{taskId}/stop
```

#### get tunnel status
```
GET /api/web/workspace-tunnel/{taskId}/status
```
returns current tunnel state.

**tunnel states:** `IDLE` → `STARTING` → `CONNECTED` → `EXPIRED` / `CLOSING` → `CLOSED`

---

### notifications

#### list notifications
```
GET /api/web/notifications?limit=20&cursor=xxx
```
- `limit` (optional) — max results per page
- `cursor` (optional) — pagination cursor from previous response

#### mark notification as read
```
POST /api/web/notifications/{id}/read
```

#### mark all notifications as read
```
POST /api/web/notifications/read-all
```
returns count of notifications marked as read.

---

### search

#### global search
```
GET /api/web/search?q=login+bug&limit=10
```
- `q` — search query
- `limit` (optional) — max results

returns matching tasks, projects, and workers.

---

### dashboard

#### cluster dashboard stats
```
GET /api/web/dashboard/stats
```
returns active/queued task counts, online workers, completed today.

#### recent tasks
```
GET /api/web/dashboard/recent-tasks
```
returns recent tasks visible to the current user.

#### recent activity
```
GET /api/web/dashboard/recent-activity
```
returns recent activity log across visible projects.

---

### profile

#### get current user
```
GET /api/web/profile
```
returns name, git info, role, totp status, platform bindings.

#### update profile
```
PUT /api/web/profile
{ "gitName": "Jane Doe", "gitEmail": "jane@example.com" }
```

#### bind platform account
```
POST /api/web/profile/bind-platform
{ "token": "bind-token-from-bot" }
```
links your overlord account to a chat platform (slack, lark, etc.) using a token from the bot's bind command.

#### list api tokens
```
GET /api/web/profile/tokens
```
returns metadata for all personal access tokens (no secrets).

#### create api token
```
POST /api/web/profile/tokens
{ "label": "my-token", "expiresAt": "2025-12-31" }
```
- `label` (required) — token display name
- `expiresAt` (optional) — expiration date (ISO 8601)

returns the token value once — store it securely.

#### revoke api token
```
POST /api/web/profile/tokens/{id}/revoke
```

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
| `queued` | waiting for a worker |
| `assigned` | assigned to a worker, starting up |
| `running` | ai agent is executing |
| `suspended` | paused — awaiting human confirmation or worker reconnection |
| `completed` | finished successfully |
| `failed` | execution failed |
| `cancelled` | cancelled by user |

## worker statuses

| status | meaning |
|--------|---------|
| `online` | available for task assignment |
| `offline` | disconnected |
| `draining` | not accepting new tasks, finishing current work |
| `decommissioned` | permanently removed |

## agent types

tasks can run on different ai agents: `claude`, `cursor`, `codex`, `custom`.

---

## common workflows

### create and monitor a task
1. create a task with a description and project key
2. poll `GET /api/web/tasks/{id}` until status is `completed` or `failed`
3. if `suspended`, check `GET /api/web/tasks/{id}/pending-confirm` and confirm if appropriate

### check cluster health
1. call `GET /api/web/dashboard/stats` for overview
2. call `GET /api/web/workers` to see individual worker status

### find a task
1. use `GET /api/web/search?q=...` for broad search
2. or `GET /api/web/tasks?status=running` for filtered listing

### open a remote workspace
1. call `POST /api/web/workspace-tunnel/{taskId}/start` to start the tunnel
2. poll `GET /api/web/workspace-tunnel/{taskId}/status` until `CONNECTED`
3. use the returned connection info to open the workspace in cursor
4. call `POST /api/web/workspace-tunnel/{taskId}/stop` when done

### manage workers
1. call `POST /api/web/workers/{id}/drain` to stop new tasks on a worker
2. call `POST /api/web/workers/{id}/undrain` to resume task assignment

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
- worker ids are UUIDs — validate format before making requests
