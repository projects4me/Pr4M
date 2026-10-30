---
paths:
  - "gaia/core/**"
  - "gaia/app/**"
---

# Gaia correctness (from CodeRabbit findings on gaia #189–234)

For each input, write, endpoint and permission you add, ask what happens on odd input, failure, repeat and a user without permission.

- **Input edges:** treat `false`, `0`, `null` and `''` as different values; don't test them with `empty()` or loose `==` (g195).
- Trim or reject trailing newlines/whitespace in ids, names and file contents before comparing or storing (g205).
- Reject conflicting identifiers (path id vs body id, two ids naming different records) instead of picking one (g205).
- Validate that a decoded body has the expected shape: an array where an object is expected, or a scalar where a list is expected, gives a 4xx, not a fatal (p554/555).
- **Permissions:** a new write path checks the same permission as the standard action it resembles (e.g. `setting.update`) (g205); an endpoint that edits permissions or roles must not let a user grant more than they hold (g216/217).
- A user-supplied URL or host that the server fetches needs a scheme and allow-list check against SSRF (g226).
- Project-scoped reads and writes stay inside the caller's projects: check the ACL scope of every query, join and id lookup (g230).
- **Partial failure:** a failed side write (notification, log, history, mail) must not fail or roll back the main change; a failed main change must not leave side effects behind (g219/226/233).
- **Atomicity:** writes that must succeed together go in one transaction; no read-then-write without a lock or unique constraint where two requests could race (g192/193/196/198/234).
- **Existing data:** a new rule, required field or format must still work for rows that already exist (null columns, old values); say how old data is handled (g196/212).
- Keep layering: SQL and domain rules in models/libraries, HTTP handling in controllers (g192/217/219).
