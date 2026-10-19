---
name: run-tests
description: How to run and write tests in the Projects4Me repos - suite commands, known failures/baselines, Gaia PHPUnit and api-tester conventions (catalogs, profiles, cases), Prometheus Gherkin acceptance tests on Mirage, hermes and ares. Use when running a suite, adding or changing tests, or judging whether a failure is new.
---

# Tests in Projects4Me

**Run the full suite:** `dev-tools/ai/ai test <gaia|prometheus|hermes>` (or the `test-runner` agent). It prints summaries plus failures marked known/unexpected; full logs in `dev-tools/ai/state/test-*.log`. Known failures live in `dev-tools/ai/config.json`.

## Commands and baselines
| Repo | Command | Notes |
|---|---|---|
| gaia PHPUnit | `dev-tools/installer/p4m test phpunit tests/libraries/security` and `.../utils` | CI runs both folders; needs `logs/tests` in the container (`docker exec gaia-dev mkdir -p /var/www/html/logs/tests`). A single file: `MSYS_NO_PATHCONV=1 docker exec -e XDEBUG_MODE=off -w /var/www/html gaia-dev vendor/bin/phpunit -c tests/phpunit.xml <file>` |
| gaia API | `dev-tools/installer/p4m db prepare-test` then `dev-tools/installer/p4m test api --mode backend,acl,client` | CI runs backend,acl. Known failure: client `post-membership-api-v1-membership` (agaia-3970) |
| prometheus | `docker exec -e TRAVIS=true -e CI=true prometheus-dev ./node_modules/.bin/ember test` | ~2¼ min, 485 tests. Baseline on dev 2026-10-13: **8 known failures** (Dashboard "Issue Today" features x4, taskboard toggle milestone, update issue status scaffold, select-and-navigate issue, task-board integration render) |
| hermes | `dev-tools/installer/p4m test hermes` | node:test, in-process app with a fake Gaia loader |
| ares | none yet | add pytest + FastAPI TestClient under `tests/` with a fake adapter; never call the real API |

## Gaia
- **Unit tests** (`tests/libraries/security|utils`, the folders CI runs): stub external systems by subclassing or overriding a protected seam (e.g. a fake `Mailer`); never add test-only endpoints, flags or constants to production code. Each test file must pass when run alone.
- **api-tester catalogs** (`tools/api-tester/apis/`):
  - `backend.json` is generated (`php tools/api-tester/bin/generate-backend-apis.php`); don't hand-edit it. If regenerating shows unrelated drift, revert it.
  - `client.json` / `acl.json` are hand-maintained. Append cases with `dev-tools/ai/ai api-cases client|acl <cases.json>` (keeps each file's style; re-serialising reformats the whole file).
  - Assertions: `status`, `shape`, `errorEqual`, `attributesPresent/Absent/Equal`, `idsIncludes`, `idsExcludes`, `resourceNamesPresent/Absent`, `includedTypesPresent/Absent`; `store` captures runtime ids (`{runtime.x}`); fixture values like `{issuestatusId}`.
  - Auth profiles (`fixtures/default.json`, seeded in `harness/prepare-test-db.sh`): `aclNoProject`, `aclProjectOnly`, `aclMembers`, `aclNoConversation`, `aclFields`, `aclNoSetting`. New profile = user + role + user_role + permissions in the harness and the fixture.
  - Every new resource or permission rule gets `acl` cases; a bug fix starts with a failing case.
- Never point a test at real external services (SMTP etc.); `gaia-test` has real Gmail credentials.

## Prometheus
Acceptance tests are Gherkin (`tests/acceptance/**/*.feature`, ember-cli-yadda) on Mirage: start from `There is no pre-existing data` / `default scenario is loaded`, add feature data with `<Model> has following details` tables, put new steps in `tests/acceptance/steps/<area>/` and register them in `steps.js`. Update the Mirage factory when a model gains attributes.
