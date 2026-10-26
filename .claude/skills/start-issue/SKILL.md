---
name: start-issue
description: Start work on a Projects4Me issue - sync dev, branch claude-issue-N, mark in_progress, start the timer, then read the issue and check its dependencies before coding. Use when the user says to start/tackle/pick up an issue number.
argument-hint: "<issue-number> [gaia|prometheus|hermes|ares|dev-tools|pr4m]"
---

# Start an issue

1. Pick the repo: A Gaia issues → `gaia`, A Prometheus → `prometheus` (dev-tools work → `dev-tools`, workspace
   files such as `.claude/` → `pr4m`; ask only if the issue doesn't say).
2. Run (one call does sync, branch, in_progress, timer, compact issue view):
   `dev-tools/ai/ai start <repo> <issue>`
   If the base branch can't fast-forward, stop and tell the user.
3. Run `dev-tools/ai/ai context <issue>`: the files, classes and endpoints the issue names, as outlines with
   line numbers, plus their tests, api-tester cases, recent commits and the referenced issues' status. Check the
   real dependencies from it ("Depends on" merged? referenced code exists?). If one isn't met, say so and stop.
4. Check scope against the house rules before planning:
   - Gaia: standard CRUD only, no custom REST actions or one-off routes; tests never change the system under test (see memory `gaia-rest-and-testing-rules`).
   - Design: layering + SOLID per the repo's `### Design` section.
   - If the issue's own scope conflicts with a rule (even scope I wrote earlier), raise it with the reviewer before building it.
5. Read only the line ranges you need (Read with offset/limit), not whole files. Ask the user only for decisions
   that change the contract; give a recommended option.
6. Bug fix: write the failing test first.

Time is tracked by checkpoints: `dev-tools/ai/ai time <issue> "<what was done>"` logs actual elapsed time since the last checkpoint. Never estimate.

## Command card (use these exactly; don't read dev-tools source to work out how)

| Need | Command |
|---|---|
| Context pack for the issue | `dev-tools/ai/ai context <issue>` |
| Fast check while fixing | `dev-tools/ai/ai test <repo> --changed` (tests mapped from changed files, seconds) or `--filter "<text or /regex/>"` |
| Gaia environment check (also run by `ai start`) | `dev-tools/ai/ai doctor gaia` (each failure prints its fix) |
| Explain one failed api case | `dev-tools/ai/ai test gaia --explain <case-id>` |
| Run one api case, no re-seed | `dev-tools/ai/ai test gaia --case <id>` |
| Full suite (before shipping) | `dev-tools/ai/ai test <repo>`: runs against this workspace's checkout; failures already failing on dev are marked `(known)` by name |
| One PHPUnit file | `dev-tools/installer/p4m test phpunit <file>` |
| Ship (tests gate, commit, PR, notes, time, in_review) | `dev-tools/ai/ai ship <repo> <issue> ship.json` (see `/ship-issue`) |
| Review threads | `dev-tools/ai/ai review <repo> <pr> --open` (bot boilerplate folded; `--raw` for everything) |
| API probe | `dev-tools/ai/ai call GET <path> [--target dev] [--profile <api-tester profile>]` |
| Time / status / comment | `ai time <issue> "<what>"`, `ai status <issue> <status> [--assignee reviewer]`, `ai comment <issue> <file.html>` |

Never call `docker exec`/`docker run` for tests yourself: the shared containers may serve another checkout, and
`p4m test` / `ai test` already handle that. On Windows use `python`, never `python3` (the Store stub hangs).
