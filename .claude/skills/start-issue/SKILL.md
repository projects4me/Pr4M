---
name: start-issue
description: Start work on a Projects4Me issue - sync dev, branch claude-issue-N, mark in_progress, start the timer, then read the issue and check its dependencies before coding. Use when the user says to start/tackle/pick up an issue number.
argument-hint: "<issue-number> [gaia|prometheus|hermes|ares]"
---

# Start an issue

1. Pick the repo: A Gaia issues → `gaia`, A Prometheus → `prometheus` (ask only if the issue doesn't say).
2. Run (one call does sync, branch, in_progress, timer, compact issue view):
   `dev-tools/ai/ai start <repo> <issue>`
   If local `dev` can't fast-forward, stop and tell the user.
3. Read the printed issue. Check the real code dependencies it names ("Depends on" issues merged? referenced code exists?). If a dependency isn't met, say so and stop.
4. Check scope against the house rules before planning:
   - Gaia: standard CRUD only, no custom REST actions or one-off routes; tests never change the system under test (see memory `gaia-rest-and-testing-rules`).
   - Design: layering + SOLID per the repo's `### Design` section.
   - If the issue's own scope conflicts with a rule (even scope I wrote earlier), raise it with the reviewer before building it.
5. Investigate with targeted reads/greps (excerpts, not whole files). Ask the user only for decisions that change the contract; give a recommended option.
6. Bug fix: write the failing test first.

Time is tracked by checkpoints: `dev-tools/ai/ai time <issue> "<what was done>"` logs actual elapsed time since the last checkpoint. Never estimate.
