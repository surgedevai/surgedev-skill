---
name: surgedev
version: 0.0.2
description: SurgeDev BaaS — manage database, storage, auth, AI, and app deploy from one CLI. OpenAPI backend. Works with Cursor, Claude Code, Windsurf, and any agent that can run shell commands.
homepage: https://github.com/surgedevai/surgedev-skill
cli_package: surgedev-cli
agent_package: '@surgedev/agent'
api_base_hint: Set SURGEDEV_API_URL to your backend (e.g. http://localhost:7130 or https://back.surgedev.ai)
last_updated: 2026-04-09
---

# SurgeDev for AI Coding Agents

**One API key. One CLI (`surgedev`).** Manage tables, storage, users, and probe your SurgeDev backend without MCP — the agent runs terminal commands.

**Canonical skill URL (raw):** `https://raw.githubusercontent.com/surgedevai/surgedev-skill/main/skill.md`

---

## Onboarding (2 steps)

### 1) Tell your AI agent

Copy and paste into **Cursor** (or any AI coding assistant):

```text
set up SurgeDev: follow the skill at:
https://raw.githubusercontent.com/surgedevai/surgedev-skill/main/skill.md

Install the CLI: npm install -g surgedev-cli
Configure SURGEDEV_API_URL and SURGEDEV_API_KEY (see skill.md), then use `surgedev ping` to verify.
```

### 2) Your API key

For direct backend access (CLI and HTTP `Authorization: Bearer`):

- Get an **API Key** from the SurgeDev dashboard (format usually `ik_...`).
- Export or add to project `.env`:

```bash
export SURGEDEV_API_URL=http://localhost:7130   # or your deployed API origin, no trailing /api
export SURGEDEV_API_KEY=ik_your_key_here
```

```env
# .env (project root)
SURGEDEV_API_URL=http://localhost:7130
SURGEDEV_API_KEY=ik_your_key_here
```

Then verify:

```bash
surgedev ping
```

---

## Why SurgeDev CLI?

| Pattern | What | For |
|--------|------|-----|
| **CLI** | `surgedev <command>` | Database, storage, auth, deploy, health checks — agent runs in terminal |
| **REST API** | `GET/POST /api/...` | Same backend; use when building app code, not for agent ops |

**Flow:**

```text
AI Agent → terminal → surgedev CLI → SurgeDev Backend API
```

---

## Install the CLI

```bash
npm install -g surgedev-cli
```

Or use `npx` without global install:

```bash
npx surgedev-cli ping
```

(If your package name on npm differs, adjust the command to match `surgedev-cli`’s published name.)

---

## Agent decision flow

When the user asks you to do something, map to CLI commands:

### Step 1 — Identify the task

| User wants to… | Use |
|----------------|-----|
| List / inspect tables, run SQL, CRUD rows | `surgedev db ...` |
| Buckets and files | `surgedev storage ...` |
| List users, view auth config | `surgedev auth ...` |
| Check backend reachability | `surgedev ping` |
| **Deploy** app to a registered VPS (build + compose) | `surgedev deploy ...` (see **Deploy** below) |
| AI models / chat in **application code** | HTTP `/api/ai/...` or dashboard; see [AI integration](docs/agent-docs/skills/ai-integration.md) (main monorepo) |

### Step 2 — Prefer safe, read-only first

1. `surgedev ping`
2. `surgedev db list` before destructive `drop` / `delete`
3. Confirm destructive operations with the user when data loss is possible

### Step 3 — Configuration

- **Never** commit real API keys. Use `.env` and `.gitignore`.
- `SURGEDEV_API_URL` is the **origin** of the API server (e.g. `http://localhost:7130`), not the path `/api` (the CLI adds paths internally).

---

## Command reference (CLI)

### Database

```bash
surgedev db list
surgedev db schema <table>
surgedev db create <table> --columns '[...]'
surgedev db drop <table>
surgedev db query <table> [--filter] [--select] [--limit]
surgedev db insert <table> --data '[...]'
surgedev db update <table> --filter '{}' --data '{}'
surgedev db delete <table> --filter '{}'
surgedev db sql "SELECT * FROM users LIMIT 10"
```

Example — filtered query (PostgREST-style filters as JSON):

```bash
surgedev db query users --filter '{"status":"eq.active"}' --select id,email --limit 20
```

### Storage

```bash
surgedev storage ls
surgedev storage create <bucket>
surgedev storage create <bucket> --private
surgedev storage drop <bucket>
surgedev storage objects <bucket>
```

### Auth

```bash
surgedev auth users
surgedev auth users --search "alice"
surgedev auth config
```

### Utility

```bash
surgedev ping
```

### Deploy

Requires the **SurgeDev control plane** plus at least one **deployment host** (VPS) where the **SurgeDev Agent** runs. Admins register hosts and install `@surgedev/agent` on the server; developers use the CLI with a normal **project API key**.

```bash
# From your app repo: pack source, upload, build on platform, run on assigned host
surgedev deploy create [--dir .] [--domain api.example.com] [--follow]

# Or pin an existing image (skip upload)
surgedev deploy create --artifact registry.example.com/myapp:tag

surgedev deploy list
surgedev deploy status <deploymentId>
surgedev deploy logs <deploymentId>
surgedev deploy rollback
surgedev deploy cancel <deploymentId>
```

**Build conventions (Node / auto-detected projects without a root `Dockerfile`):**

- The platform uses **`npm ci`** → commit **`package-lock.json`**, or add a root **`Dockerfile`** and the agent will use it instead of generating one.
- **Next.js**: include **`build`** and **`start`** scripts (container listens on **3000** by default).
- If you must upload without a lockfile: `surgedev deploy create --skip-checks` (build may still fail server-side).

**VPS side (operators):** install Docker + Node 18+, then run **`@surgedev/agent`** with `SURGEDEV_API_URL`, `SURGEDEV_HOST_ID`, and `SURGEDEV_AGENT_TOKEN` from host registration. The agent polls the API and runs `docker build` / `docker compose` locally — the control plane does not SSH into your box.

---

## Environment variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SURGEDEV_API_URL` | Backend base URL | `http://localhost:7130` |
| `SURGEDEV_API_KEY` | API Key (`ik_...`) | Required |

Some setups also accept `API_BASE_URL` / `API_KEY` as aliases (see CLI source).

---

## Deeper docs (main SurgeDev monorepo)

Full skill guides live in the open-source monorepo:

| Topic | Link |
|-------|------|
| Authentication | [auth-patterns.md](docs/agent-docs/skills/auth-patterns.md) |
| Database | [database-guide.md](docs/agent-docs/skills/database-guide.md) |
| Schema patterns | [schema-patterns.md](docs/agent-docs/skills/schema-patterns.md) |
| Storage | [storage-guide.md](docs/agent-docs/skills/storage-guide.md) |
| AI integration | [ai-integration.md](docs/agent-docs/skills/ai-integration.md) |
| Deploy — VPS Agent install (ZH) | [surgedev-vps-agent-install.zh.md](docs/deployment/surgedev-vps-agent-install.zh.md) |
| Deploy — cloud guides | [docs/deployment/](docs/deployment/) |

CLI package source: [surgedev-cli](surgedev-cli/README.md) · Agent: [`@surgedev/agent`](https://www.npmjs.com/package/@surgedev/agent) (install on target VPS)

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|--------|----------------|-----|
| `未找到 API Key` / missing API key | `SURGEDEV_API_KEY` unset | Set env or `.env` in project root |
| Connection refused | Wrong host/port or backend not running | Check `SURGEDEV_API_URL`, start backend |
| 401 / 403 | Invalid or expired key | Regenerate key in dashboard |
| Wrong URL in OAuth or redirects | `API_BASE_URL` doubled with `/api` | Use origin only for CLI; avoid `.../api` as base for env that expects origin |
| Deploy build fails (`npm ci`) | Missing `package-lock.json` | Add lockfile or supply a root `Dockerfile` |
| Deploy running but wrong port / 502 | App not listening on expected port | Match `EXPOSE` / compose mapping (e.g. Node defaults to 3000); check `surgedev deploy logs` |
| No host picked / 503 | No active agent or pool mismatch | Register host, keep agent running, check tier/pool |

---

## Maintainer notes

- Default branch is assumed **`main`**. If you use another branch, adjust the raw URL accordingly.
- Update `last_updated` in the YAML frontmatter when this file changes.

---

**One key. One CLI. Full backend control from your AI agent.**

Repository: [surgedevai/surgedev-skill](https://github.com/surgedevai/surgedev-skill)
