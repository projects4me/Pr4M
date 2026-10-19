# Projects4Me workspace (Pr4M)

The Pr4M workspace holds four service repos as submodules, each with its own `CLAUDE.md` of house rules:
`gaia` (PHP API), `prometheus` (Ember UI), `hermes` (Socket.IO relay) and `ares` (FastAPI AI service). It also
has `docker-images` and `dev-tools/` (the tooling). The engineering guide is
`Projects4Me — Engineering Guide for Claude Code.md` in this folder.

## Working rules (summary; the repo CLAUDE.md files are authoritative)
- Every piece of work has a Projects4Me issue, in milestone **Claude-v1**:
  - Gaia → **A Gaia** (`agaia`)
  - Prometheus → **A Prometheus** (`aprom`)
  - Hermes → **A Hermes** (`ah`)
  - Ares → **A Ares** (`aa`)

  Log time and comment details on the issue as you work.
- Branch `claude-issue-NNNN` off an up-to-date `dev`.
- Commit, push and open PRs only as **Agent Odysseus**, the team's shared agent identity:
  - use `dev-tools/odysseus/odysseus git|gh …` (or `dev-tools/ai/ai`);
  - push only to the `odysseus` remote.

  Your reviewer (`reviewer` in `dev-tools/ai/config.local.json`) reviews and merges.
- UI work in Prometheus follows `prometheus/docs/ui/design-system.md` (AdminLTE boxes, square edges).
- Check real code dependencies before splitting or ordering issues.
- Design: follow the layering and separation of concerns from Fowler's *Patterns of Enterprise Application
  Architecture*, and SOLID, above all Single Responsibility. Each repo's `### Design` section maps the layers.

## Workflow automation (use these instead of step-by-step tool calls)
- `dev-tools/ai/ai` (see `dev-tools/ai/README.md`):
  - commands: `start`, `time`, `test`, `commit`, `pr`, `review`, `reply`, `issue`, `status`, `call`, `api-cases`;
  - skills: `/start-issue`, `/ship-issue`, `/address-review`;
  - agents: `test-runner`, `design-reviewer`, `deep-reasoner`.

  Log time with `ai time`: the actual time elapsed since the last checkpoint, never an estimate.
- Talos (`dev-tools/talos`) works on issues you assign to Agent Odysseus with headless Claude Code, and hands them
  back to you.
- First-time setup: `dev-tools/installer/p4m setup`, then `dev-tools/installer/p4m configure`.
- After a PR is opened and the issue handed off, suggest `/clear` before the next issue.

## Reference (load the skill when needed)
- Local stack, URLs, Postgres, migrations and seeds, run commands, Windows/Git Bash gotchas, browser checks: skill
  `p4m-stack`.
- Running or writing tests, known failures, api-tester and Gherkin conventions: skill `run-tests`.
- Never write PHP (namespaces with `\`) through a heredoc or inline Python, because the backslashes get lost. Use
  the Write/Edit tools or a script file.
