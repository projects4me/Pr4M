---
name: run-tests
description: How to run and write tests in the Projects4Me repos - suite commands, known failures/baselines, Gaia PHPUnit and api-tester conventions (catalogs, profiles, cases), Prometheus Gherkin acceptance tests on Mirage, hermes and ares. Use when running a suite, adding or changing tests, or judging whether a failure is new.
---

# Tests in Projects4Me

**Run the full suite:** `dev-tools/ai/ai test <gaia|prometheus|hermes|dev-tools>` (or the `test-runner` agent). It prints summaries plus failures marked `(known)` / `<-- unexpected`; full logs in `dev-tools/ai/state/test-*.log`.

**Full gaia run ≈ 1 min** (phpunit a few seconds, api ~1 min incl. the seed, in the copied-source `p4m-test-gaia-<key>` container; a baseline run adds about as much): run it in the foreground with a 600000 ms timeout, not `run_in_background` (Talos runs have no background tasks). Don't edit the repo while it runs: the result is fingerprinted at start, and a tree that changed mid-run is recorded stale and `ai ship` refuses it. A killed run's `pr4m_test` lock is taken over by the next run (dead holder PID); the wait message names the holder PID and command.

**While fixing:** `dev-tools/ai/ai test <repo> --changed` runs only the tests mapped from the branch's changed files (fixed rules in `dev-tools/ai/lib/testmap.py`; it lists what it couldn't map), or `--filter "<text|/regex/>"`. A targeted run never satisfies `ai ship`, which needs a full passing run on the final tree.

**Gaia test loop:** `ai doctor gaia` checks the test environment and prints the fix for each failure (`ai start` runs it); `ai test gaia --explain <case-id>` shows expected vs actual for one failed api case from the last log; `ai test gaia --case <id>` runs one api case on the already-seeded DB without re-seeding (substring match; cases that depend on earlier cases' data need the full run).

**Known vs new failures:** a failure is known if it's in `dev-tools/ai/config.json` `knownFailures` or it also fails on the base commit. The first time a branch shows a failure not seen before, `ai test` runs that suite once on a detached worktree of the merge-base (`dev-tools/ai/state/baseline/<repo>`) and records the failing names (`ai baseline <repo>` does it on demand). Don't stash and re-run by hand.

**Which code is tested:** `p4m test` runs against this workspace's checkout. Gaia always runs in its own `p4m-test-gaia-<key>` container (gaia-test's image, env and network, source copied in, OPcache on, Xdebug off). If the shared `hermes` container mounts another folder, it starts a throwaway `p4m-test-*` container from the same image, env and network instead; Prometheus always uses its own test container with the source copied in and an incremental test build (`p4m test clean` removes them). Never `docker exec`/`docker run` the suites yourself.

## Commands
| Repo | Command | Notes |
|---|---|---|
| gaia PHPUnit | `dev-tools/installer/p4m test phpunit tests/libraries/security`, `.../utils` and `tests/tools`, or one file | CI runs all three; `tests/tools` holds the gaia-dev consistency checks (graph up to date, behaviours tested, scaffolder anchors) |
| gaia API | `dev-tools/installer/p4m db prepare-test` then `dev-tools/installer/p4m test api --mode backend,acl,client [--filter a\|b]` | `ai test` does both under a lock (every run re-seeds `pr4m_test`). CI runs backend,acl. Known failure: client `post-membership-api-v1-membership` (agaia-3970) |
| prometheus | `dev-tools/installer/p4m test ember [--filter "<text or /regex/i>"]` | first run builds (~1 min), reruns 10–15 s; full suite ~1½ min, ~509 tests |
| hermes | `dev-tools/installer/p4m test hermes` | node:test, in-process app with a fake Gaia loader |
| dev-tools | `dev-tools/ai/ai test dev-tools` | unittest for talos, ai and installer |
| ares | none yet | add pytest + FastAPI TestClient under `tests/` with a fake adapter; never call the real API |

## Gaia
- Conventions for unit tests and the api-tester load from `.claude/rules/api-tester.md` when you touch `gaia/tools/api-tester/**` or `gaia/tests/**`.
- Checks before shipping: `php tools/gaia-dev/bin/gaia-dev check api-cases` (plus `graph export --check`, `docs generate --check`); `ai ship` runs them for Gaia PRs.

## Prometheus
Acceptance tests are Gherkin (`tests/acceptance/**/*.feature`, ember-cli-yadda) on Mirage: start from `There is no pre-existing data` / `default scenario is loaded`, add feature data with `<Model> has following details` tables, put new steps in `tests/acceptance/steps/<area>/` and register them in `steps.js`. Update the Mirage factory when a model gains attributes.
