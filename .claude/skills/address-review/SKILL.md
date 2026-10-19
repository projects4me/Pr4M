---
name: address-review
description: Address PR review feedback on a Projects4Me PR - fetch open review threads, make the changes, re-test, push a follow-up commit, reply to each thread, update the PR description and tracker. Use when the user says they reviewed/commented on a PR or asks to handle review comments.
argument-hint: "<repo> <pr-number> [issue-number]"
---

# Address a PR review

1. `dev-tools/ai/ai timer <issue>` (start the time checkpoint), then
   `dev-tools/ai/ai review <repo> <pr> --open` - only threads without my reply, plus follow-ups.
2. For each thread decide: change, or explain (with evidence) why not. Treat reviewer direction as a standing rule: if it generalises (e.g. "no custom REST actions"), save it to memory `gaia-rest-and-testing-rules` (or a new feedback memory).
3. Make the changes on the existing branch (`git checkout claude-issue-<issue>`). Prefer reverting to `origin/dev` over hand-undoing. Re-run tests via the `test-runner` agent.
4. Commit as a **new** commit (no amend/force-push):
   `dev-tools/ai/ai commit <repo> <issue> -F <msg-file> --files ...` (message: `refactor|fix(scope): ... Review feedback on #<pr>`).
5. Reply to every thread in one call: write `{"<comment id>": "<reply>"}` to a JSON file, then
   `dev-tools/ai/ai reply <repo> <pr> <file.json>`. Say what changed and the commit hash; answer questions directly.
6. If scope changed, update the description: `dev-tools/ai/ai pr-body <repo> <pr> <file.md>`.
7. Tracker: `ai comment <issue> <file.html>` summarising the round, then `ai time <issue> "review round: ..."`. Then hand it back: `dev-tools/ai/ai status <issue> in_review --assignee reviewer` (Talos moves picked-up issues to `in_progress`, so set it explicitly).
8. Tell the user what changed per comment and any open question, in a few lines.
