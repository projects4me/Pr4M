---
name: run-tests
description: How to run and write tests in the Projects4Me repos - suite commands, known failures/baselines, Gaia PHPUnit and api-tester conventions (catalogs, profiles, cases), Prometheus Gherkin acceptance tests on Mirage, hermes and ares. Use when running a suite, adding or changing tests, or judging whether a failure is new.
---

# Tests in Projects4Me

**Run the full suite:** `dev-tools/ai/ai test <gaia|prometheus|hermes|dev-tools>` (or the `test-runner` agent). It prints summaries plus failures marked `(known)` / `<-- unexpected`; full logs in `dev-tools/ai/state/test-*.log`.

**While fixing:** `dev-tools/ai/ai test <repo> --changed` runs only the tests mapped from the branch's changed files (fixed rules in `dev-tools/ai/lib/testmap.py`; it lists what it couldn't map), or `--filter "<text|/regex/>"`. A targeted run never satisfies `ai ship`, which needs a full passing run on the final tree.

**Known vs new failures:** a failure is known if it's in `dev-tools/ai/config.json` `knownFailures` or it also fails on the base commit. The first time a branch shows a failure not seen before, `ai test` runs that suite once on a detached worktree of the merge-base (`dev-tools/ai/state/baseline/<repo>`) and records the failing names (`ai baseline <repo>` does it on demand). Don't stash and re-run by hand.

**Which code is tested:** `p4m test` runs against this workspace's checkout. If the shared container (`gaia-test`, `hermes`) mounts another folder, it starts a throwaway `p4m-test-*` container from the same image, env and network instead; Prometheus always uses its own test container with the source copied in and an incremental test build (`p4m test clean` removes them). Never `docker exec`/`docker run` the suites yourself.

## Commands
| Repo | Command | Notes |
|---|---|---|
| gaia PHPUnit | `dev-tools/installer/p4m test phpunit tests/libraries/security` and `.../utils`, or one file | CI runs both folders |
| gaia API | `dev-tools/installer/p4m db prepare-test` then `dev-tools/installer/p4m test api --mode backend,acl,client [--filter a\|b]` | `ai test` does both under a lock (every run re-seeds `pr4m_test`). CI runs backend,acl. Known failure: client `post-membership-api-v1-membership` (agaia-3970) |
| prometheus | `dev-tools/installer/p4m test ember [--filter "<text or /regex/i>"]` | first run builds (~1 min), reruns 10–15 s; full suite ~1½ min, ~509 tests |
| hermes | `dev-tools/installer/p4m test hermes` | node:test, in-process app with a fake Gaia loader |
| dev-tools | `dev-tools/ai/ai test dev-tools` | unittest for talos, ai and installer |
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
