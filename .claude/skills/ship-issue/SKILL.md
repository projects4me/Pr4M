---
name: ship-issue
description: Finish a Projects4Me issue - run the full test suite, review the diff against the design rules, write the tracker notes, then (after the user approves) commit, push and open the PR as Agent Odysseus and hand the issue to the reviewer. Use when implementation is done or the user says "create/open/submit the PR".
argument-hint: "<issue-number>"
---

# Ship an issue

Repo comes from the timer (`dev-tools/ai/state/timers.json`) or the branch `claude-issue-N`.

1. **Tests:** delegate to the `test-runner` agent (`dev-tools/ai/ai test <repo>`). Any unexpected failure → fix first; report known failures as known.
2. **Design check:** for non-trivial diffs, ask the `design-reviewer` agent to review `git diff origin/dev` against the repo's `### Design` section and Gaia review rules. Fix real findings.
3. **Files:** list exactly the files this issue changed (`git status --short`). Never include files I didn't create for this issue, `CLAUDE.md`, or `tests/.phpunit.result.cache`.
4. **Tracker notes:** `ai comment <issue> <file.html>` with what was done, decisions, files, test results, follow-ups. Log anything out of scope as a new issue instead of fixing it.
5. **Approval:** summarise for the user and ask to commit + open the PR. An approval covers this PR only. If the user already said "create the PR", go ahead.
6. **Commit + push:** write the message to a file (`type(scope): summary`, body, `Refs agaia-N`), then
   `dev-tools/ai/ai commit <repo> <issue> -F <msg-file> --files <f1> <f2> ...`
   (checks branch, refuses if origin/dev moved, appends the co-author trailer, pushes to `odysseus`).
7. **PR + hand-off:** write the body (What · Why · How tested · Contract impact · Migration impact · Out of scope), then
   `dev-tools/ai/ai pr <repo> <issue> --title "<title>" --body-file <file> --time-note "<what the last stretch covered>"`
   This opens the PR, comments the link on the issue, logs actual elapsed time, sets `in_review` and assigns your reviewer (`ai/config.local.json`).
8. Tell the user: PR link, test results, follow-up issues. Keep it short.
