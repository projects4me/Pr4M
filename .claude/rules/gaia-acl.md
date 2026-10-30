---
paths:
  - "gaia/core/libs/security/**"
  - "gaia/app/metadata/**"
---

# Gaia ACL conventions

- Look up before reading source: `gaia-dev acl <resource> [profile]` (allow / members / deny per profile) and `gaia-dev model <Name>`; the rules themselves are in `docs/generated/acl-matrix.md` (`ai kb search --kind doc acl`).
- When adding a resource, check its permissions on existing roles and add `acl` api-tester cases (`gaia-dev check api-cases` fails a restricted profile with no `acl` case).
- Project-scoped data declares `'acl' => ['groups' => ['Project']]` in its metadata; field-level rules use `<model>.<field>.<action>` permissions.
- No permission seeds: the reviewer grants new module permissions manually; test-only permission rows go in the harness.
- Changing metadata or ACL changes the graph: run `gaia-dev graph export` and `docs generate` and commit `docs/generated/`.
