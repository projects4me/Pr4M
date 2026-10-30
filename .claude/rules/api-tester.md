---
paths:
  - "gaia/tools/api-tester/**"
  - "gaia/tests/**"
---

# Gaia tests and api-tester conventions

- **Unit tests** (`tests/libraries/security|utils`, the folders CI runs): stub external systems by subclassing or overriding a protected seam (e.g. a fake `Mailer`); never add test-only endpoints, flags or constants to production code. Each test file must pass when run alone.
- **api-tester catalogs** (`tools/api-tester/apis/`):
  - `backend.json` is generated (`php tools/api-tester/bin/generate-backend-apis.php`); don't hand-edit it. If regenerating shows unrelated drift, revert it.
  - `client.json` / `acl.json` are hand-maintained. Append cases with `dev-tools/ai/ai api-cases client|acl <cases.json>` or `gaia-dev scaffold api-cases` (keeps each file's style; re-serialising reformats the whole file).
  - Assertions: `gaia-dev assertions` lists them; `store` captures runtime ids (`{runtime.x}`); fixture values like `{issuestatusId}`.
  - Auth profiles (`fixtures/default.json`, seeded in `harness/prepare-test-db.sh`): `gaia-dev profiles`. New profile = user + role + user_role + permissions in the harness and the fixture (`gaia-dev scaffold api-cases --new-profile`).
  - Every new resource or permission rule gets `acl` cases; a bug fix starts with a failing case. Check with `gaia-dev check api-cases`.
- Never point a test at real external services (SMTP etc.); `gaia-test` has real Gmail credentials.
