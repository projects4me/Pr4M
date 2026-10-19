# Pr4M: the Projects4Me workspace

Everything needed to develop Projects4Me in one checkout: the four services and their Docker images as
submodules, the dev tooling, and the shared Claude Code skills and agents.

| Path | What it is |
|---|---|
| [`gaia/`](https://github.com/projects4me/gaia) | REST API (PHP 7 / Phalcon 4, PostgreSQL) |
| [`prometheus/`](https://github.com/projects4me/prometheus) | Web app (Ember) |
| [`hermes/`](https://github.com/projects4me/hermes) | Real-time relay (Node / Socket.IO) |
| [`ares/`](https://github.com/projects4me/ares) | AI service (FastAPI) |
| [`docker-images/`](https://github.com/projects4me/docker-images) | Base images |
| [`dev-tools/`](https://github.com/projects4me/dev-tools) | `installer/p4m` (run the stack), `p4m configure`, the gaia MCP server, the shared Agent Odysseus identity, the `ai` workflow CLI, Talos |
| `.claude/` | Claude Code skills (`/start-issue`, `/ship-issue`, `/address-review`, stack and test references) and agents (`test-runner`, `design-reviewer`, `deep-reasoner`) for everyone working in this folder |

Each repo has its own `CLAUDE.md` with its rules; they assume this workspace layout.

## Get started

Works on Windows 10/11, macOS and Ubuntu/Debian.

```bash
git clone --recurse-submodules --remote-submodules https://github.com/projects4me/Pr4M.git
cd Pr4M
git submodule foreach 'git switch $(git config -f $toplevel/.gitmodules submodule.$name.branch)'   # on branches, not detached
dev-tools/installer/p4m setup        # only if you run the stack locally (Docker)
dev-tools/installer/p4m configure    # your settings + the shared Agent Odysseus credentials
```

Then open Claude Code in this folder. Full details and prerequisites (Python 3.10+, git, Node 22.15+ for Talos,
GitHub CLI; Docker for the local stack): [dev-tools/README.md](https://github.com/projects4me/dev-tools#readme).

## Always on the latest

The repos follow their branches, not the commits this repo records: gaia, prometheus, hermes and ares follow `dev`,
docker-images follows `master`, and dev-tools follows `main`. Nobody works at a pinned commit, so a change in a
sub-repo **never** needs a commit here.

- `--remote-submodules` clones each repo at the tip of its branch, but **detached**. The `git submodule foreach`
  line puts each one **on** its branch (`dev`, `master` or `main`), so you can pull and commit straight away.
- **`dev-tools/installer/p4m repos sync`** brings everything to the latest. It fast-forwards this workspace,
  switches each repo to its branch and pulls. Repos with local changes, or on another branch, are left alone with a
  warning. Run it whenever you want to catch up.
- The Claude Code workflow (`/start-issue <n>`) syncs `dev` itself before branching `claude-issue-<n>`.
- `.gitmodules` sets `ignore = all`, so repos moving ahead never show as changes here, and nobody records a pin
  by accident.

## Licence

AGPLv3: see [LICENSE](LICENSE).
