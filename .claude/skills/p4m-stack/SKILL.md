---
name: p4m-stack
description: Projects4Me local stack reference - p4m CLI, local URLs, containers, Postgres access, migrations and seeds, the issue-number sequence, repo run commands, Windows/Git Bash gotchas and browser checks. Use before running docker/psql/php/npm commands, migrating or seeding a database, calling the local app/API, or checking UI in a browser.
---

# Projects4Me local stack

## p4m and URLs
- CLI: `dev-tools/installer/p4m` (docs `dev-tools/installer/P4M-CLI.md`). App https://app.local-projects4.me, API https://api.local-projects4.me, test API https://api-test.local-projects4.me.
- Quick API calls with login handled: `dev-tools/ai/ai call GET <path> [--target dev] [--profile <api-tester profile>]`.
- Tracker: the `gaia` MCP server (`dev-tools/gaia-mcp`, user Agent Odysseus) or `dev-tools/ai/ai issue|comment|status|time`. If MCP tools fail to sign in after a config change, the MCP process is stale: restart Claude Code.

## Database
- Postgres: `docker exec postgres-dev psql -U nouman -d pr4m` (test DB `pr4m_test`). Columns are camelCase: quote them (`"userId"`); `settings.deleted` is boolean, most other tables use 0/1.
- Migrate (declarative, DROPS columns removed from metadata): `dev-tools/installer/p4m db migrate`. Only against the local DB.
- Seeds (idempotent, run once per environment after migrating): `db/seeds/settings.sql`, `db/seeds/issue_statuses.sql` (`docker exec -i postgres-dev psql -U nouman -d pr4m -v ON_ERROR_STOP=1 < gaia/db/seeds/<file>`). `p4m db prepare-test` seeds `pr4m_test`.
- Issue numbers come from sequence `issues_issueNumber_seq` (moved to the real tracker's numbering, 3955+, on 2026-10-13; a DB reset re-imports the old dump and needs `setval` again).

## Repo run commands (outside p4m)
- gaia: `docker network create backend` (once), `docker compose up -d` → API :8080, test API :8081, Postgres :5432. Migrate in the container: `docker exec gaia-dev php cli.php migration migrate`.
- prometheus: `npm install`; `API_HOST_DEV=http://localhost:8080 HERMES_URL_DEV=http://localhost:9000 npm start` (:4200). Lint: `npm run lint:js`, `npx ember-template-lint .` (locally `node_modules` exist only in the `prometheus-dev` container: prefix with `docker exec prometheus-dev`).
- hermes: `npm install`; `npm start` (:9000). `docker compose up` → hermes :9000 (→ gaia :8080), hermes-test :9001 (→ gaia-test :8081).
- ares: `pip install -r requirements.txt`; `ANTHROPIC_API_KEY=... uvicorn app.main:app --port 8000 --reload`.

## Containers and checkouts
- The shared containers serve the folder `p4m up` was last run from, which may be another workspace on this
  machine. Check with `docker inspect gaia-test --format '{{range .Mounts}}{{.Source}} {{end}}'`. Tests handle
  this themselves (`p4m test` / `ai test`); for anything else (migrations, debug scripts) make sure the container
  serves this checkout before trusting the result.

## Windows / Git Bash gotchas
- Use `python`, never `python3`: on Windows `python3` can be the Microsoft Store stub, which hangs until the timeout.
- `docker exec -w /var/www/html …` from Git Bash needs `MSYS_NO_PATHCONV=1`, or the path gets mangled.
- Add `-e XDEBUG_MODE=off` to one-off `php` commands in `gaia-dev` to silence Xdebug noise.
- Don't `cat > file` without input (it waits on stdin until timeout).
- Debug scripts that bootstrap Gaia need the config loaded (`new \Gaia\Libraries\Config($di)` with fileHandler, modelsManager, relationshipFactory, metaManager in the DI, `Di::setDefault`) or `$settings` is empty.

## Browser checks
Playwright (Python) with `channel="chrome"` in a scratchpad venv; log in with the credentials from `dev-tools/gaia-mcp/gaia-mcp.env` (never print them). Compare a UI change next to an existing screen of the same type.
