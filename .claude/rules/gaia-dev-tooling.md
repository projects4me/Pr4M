---
paths:
  - "gaia/tools/gaia-dev/**"
---

# gaia-dev tooling (from CodeRabbit findings on gaia #206/209/211/212)

- Check the result of every `file_put_contents`, `json_encode`, `rename`, `mkdir` and regeneration step; a false return is a failure, not success (g206/211/212).
- Exit non-zero on any failure and print the cause to stderr; never print "done" after a failed step (g206/211).
- Reject unknown options and missing option values with a usage error instead of ignoring them (g206/209).
- Write generated files to a temp file and rename, so a failed run doesn't leave a half-written file.
- A `--check` mode must not modify files and must exit non-zero when the output differs.
