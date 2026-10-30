---
paths:
  - "prometheus/app/**"
---

# Prometheus async, rollback and accessibility (from CodeRabbit findings on prometheus #526–577)

- **Latest request wins:** an older response must never overwrite newer state (p544/545/546/553). Default: an ember-concurrency `restartable` task. It is imported in 7 files but not declared in `package.json` (transitive only), so ask the reviewer to declare it before spreading its use; until then a request counter that ignores stale results is acceptable.
- **Drop the second submit:** a save/submit handler ignores calls while one is pending (`dropTask`, or a pending flag reset in `finally`) and disables its button (p528/561).
- **Unhandled rejections:** every `await`/promise in a handler or task has a catch that shows the error and restores state (p543/544/574).
- After every `await` in a component or controller, check `isDestroying`/`isDestroyed` before touching state (p563).
- **Rollback:** on a failed save, restore the previous values (`rollbackAttributes()` or the saved copy) and the UI state; don't leave an optimistic update in place (p553).
- Never put user text into `htmlSafe`, `{{{ }}}` or `innerHTML`; use escaped bindings (p527/543).
- **Keyboard/a11y:** modals move focus in and return it on close; every mouse action (toggle, row click) is reachable by keyboard (`button`, Enter/Space); ignore Enter during IME composition (`event.isComposing`); each label is tied to its control (`for`/`id`); state is exposed to screen readers (`aria-expanded`, `aria-pressed`, `aria-busy`) (p533/549/550/551/559/560/562).
- Styles go in the shared style files, not inline in templates or components (p533/558/560).
- **Tests:** every new test must fail if the line it covers is reverted; every failure path gets a rejection test (p544/557); fixtures must not already contain the value the test checks (p550/552).
