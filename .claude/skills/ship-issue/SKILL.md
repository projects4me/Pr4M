---
name: ship-issue
description: Finish a Projects4Me issue - run the full test suite, review the diff against the design rules, then (after the user approves) commit, push, open the PR, write the tracker notes and hand the issue to the reviewer in one ai ship call. Use when implementation is done or the user says "create/open/submit the PR".
argument-hint: "<issue-number>"
---

# Ship an issue

Repo comes from the timer (`dev-tools/ai/state/timers.json`) or the branch `claude-issue-N`.

1. **Tests:** delegate the full run to the `test-runner` agent (`dev-tools/ai/ai test <repo>`, in the foreground;
   a full gaia run takes about a minute). Don't edit the repo while it runs. Use `--changed` while fixing. Fix any
   unexpected failure first. Failures marked `(known)` already fail on the base commit (or are listed in
   `dev-tools/ai/config.json`); report them as known, don't investigate them. `ai ship` refuses unless the last
   full run passed on exactly the current tree, so re-run after any edit.
2. **Review (required for every code change):** run the `design-reviewer` and `correctness-reviewer` agents in parallel (two foreground calls in one message) on `git diff origin/dev`; docs-only changes may skip. `design-reviewer` checks the repo's `### Design` section and Gaia review rules; `correctness-reviewer` walks failure, repeat, race, odd-input and permission cases against `.claude/rules/`. Fix real findings (then step 1 again).
3. **Files:** list exactly the files this issue changed (`git status --short`). Never include files I didn't create for this issue, `CLAUDE.md`, or `tests/.phpunit.result.cache`.
4. **Approval:** summarise for the user and ask to commit + open the PR. An approval covers this PR only. If the user already said "create the PR" (or Talos assigned the issue), go ahead.
5. **Ship in one call:** write `ship.json` (in the scratchpad or `dev-tools/ai/state/`):
   ```json
   {"title": "feat(scope): summary", "body": "commit body",
    "files": ["path/a", "path/b"],
    "what": "…", "why": "…", "howTested": "optional extra", "contract": "None or …", "migration": "None or …",
    "outOfScope": "optional", "notes": "<p>decisions, deviations, follow-ups (HTML)</p>",
    "timeNote": "what the last stretch covered"}
   ```
   Check it with `dev-tools/ai/ai ship <repo> <issue> ship.json --dry-run`, then run it without `--dry-run`.
   For Gaia it first runs the gaia-dev checks (`graph export --check`, `docs generate --check`, `check api-cases`) and stops on a failure. It checks the branch and base, commits exactly `files` as Agent Odysseus (+co-author, `Refs <issue>`), pushes to
   `odysseus`, opens the PR (body: What · Why · How tested with the recorded result · Contract · Migration · Out of
   scope), posts the tracker notes with the PR link and test result, logs actual elapsed time, sets `in_review` and
   assigns your reviewer. With `--defer-handoff` it hands off normally when the repo isn't in `codeRabbitRepos`
   (output says "CodeRabbit not enabled"): then finish `done`, not `awaiting_bot_review`. Docs-only change with no suite: `--no-tests "<why>"`.
   Log anything out of scope as a new issue instead of fixing it.
6. Tell the user: PR link, test results, follow-up issues. Keep it short.
