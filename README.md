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

Works on Windows 10/11 (Git Bash + Docker Desktop), macOS and Ubuntu/Debian.

```bash
git clone --recurse-submodules https://github.com/projects4me/Pr4M.git
cd Pr4M
dev-tools/installer/p4m setup        # Windows: dev-tools\installer\p4m.cmd setup
dev-tools/installer/p4m configure    # your settings + the shared Agent Odysseus credentials
```

Then open Claude Code in this folder. Full details and prerequisites (Python 3.10+, Docker, git, Node 22.15+
for Talos, GitHub CLI): [dev-tools/README.md](https://github.com/projects4me/dev-tools#readme).

## Working in the submodules

The submodules start detached at the commit this repo pins. Before working in one, check out its branch:

```bash
git -C gaia switch dev && git -C gaia pull --ff-only     # gaia, prometheus, hermes, ares: dev
```

The Claude Code workflow (`/start-issue <n>`) does this for you and branches `claude-issue-<n>` from an up-to-date
`dev`. To move every submodule to the latest upstream branch: `git submodule update --remote`.

## Licence

AGPLv3: see [LICENSE](LICENSE).
