---
id: t0dotivcc1dvyw70nti9c2h
title: 2026 06 11 Secret Redaction Coderabbit
desc: ''
updated: 1781194103700
created: 1781194090859
---

Verify each finding against current code. Fix only still-valid issues, skip the
rest with a brief reason, keep changes minimal, and validate.

Status legend:

- `[ ]` valid/actionable, not yet implemented
- `[x]` implemented locally
- `[c]` cancelled; do not implement as requested by CodeRabbit

## Inline comments

In `@apps/daemon/src/orchestrator/daemon_runtime.ts`:

- [x] Around line 658-677: remove `droppedEventIds` from the
      `secrets.redacted` / `secrets.detected` audit payload emitted during
      provider-source replay. Keep the audit payload to rule/count summaries:
      `provider`, `sessionId`, `mode`, `eventsAffected`, and `countsByRule`.
      If dropped-event identifiers are needed for troubleshooting, emit only a
      count in operational logging or use the existing separate dropped-events
      path.

In `@apps/daemon/src/orchestrator/provider_source_replay.ts`:

- [x] Around line 149-156: re-apply the replay secrets policy to
      twin-backed history before returning it from
      `loadPersistedSessionHistoryEvents`. The current code trusts twin history
      because live ingestion now redacts it, but old twins can predate the
      redaction feature. Implement with the existing
      `applyReplaySecretsPolicy(twinConversation, options?.secretsPolicy)`
      helper, keep `source: "twin"`, and preserve the redaction summary when
      the helper reports one. Also verify whether the `session_snippets.ts`
      twin fast-path needs the same treatment, since it maps twin events
      directly today.

In `@apps/runtime/src/config/shared_behavior_config.ts`:

- [x] Around line 301-303: reject explicit `secretsPolicy.mode: null` instead
      of treating it as absent. Replace the nullish default with logic that
      defaults only `undefined` to `"redact"`, rejects `null`, and then
      validates the resulting value with `SECRETS_POLICY_MODES`.

In `@apps/runtime/src/policy/secrets_redaction.ts`:

- [x] Around line 162-169: guard the combined keyword regex when all rules are
      disabled. `new RegExp("", "gi")` matches zero-length strings, so use a
      guaranteed non-matching pattern or make the pattern nullable, and return
      an empty keyword set immediately for the no-keywords case. Add a
      regression test using `disabledRules: SECRETS_RULES.map((rule) =>
      rule.id)` to ensure processing terminates and reports no matches.

In `@apps/web/src/session_recording_actions.ts`:

- [x] Around line 470-547: thread `sharedConfig.secretsPolicy` into the
      stop/conflict-resolution history loads so stop paths honor `detect` and
      `off` instead of falling back to fail-closed `redact`. Current valid
      targets are the `loadPersistedSessionHistoryEvents` call inside
      `stopConflictingActiveOutputsAtPath` and the one inside
      `runSessionRecordingStopAction`; the restart/new-recording call sites
      around the original line 670 and 854 already pass the policy.

In `@documentation/notes/task.2026.2026-06-11-session-output-metadata.md`:

- [c] Line 5: do not remove the Dendron `updated` frontmatter field. This repo's
      notes use standardized Dendron frontmatter with `updated` present, and
      local guidance says Dendron manages that field automatically. Treat this
      as bot churn unless a human asks for a broader frontmatter cleanup.

## Outside diff comments

In `@apps/daemon/src/main.ts`:

- [x] Around line 435-480: make the custom `loadSessionSnapshot` passed to
      `runtimeLoop` fall back to persisted history replay when the in-memory
      `sessionSnapshotStore.get(sessionId)` has no usable snapshot. The fallback
      should resolve matching session metadata, call
      `loadPersistedSessionHistoryEvents(metadata, sessionStateStore, {
      secretsPolicy: sharedConfig.secretsPolicy })`, and return those events so
      export requests use the same replay/redaction path as the orchestrator's
      default loader.

In `@documentation/notes/dev.decision-log.md`:

- [c] Around line 1-7: do not remove `updated` keys from Dendron frontmatter in
      `dev.decision-log.md`, `task.2026.2026-05-26-secrets-suppression.md`, or
      `task.2026.2026-05-28-persona-support.md`. This conflicts with the
      repository's Dendron note convention, and the
      `task.2026.2026-05-26-secrets-suppression.md` file is currently deleted
      in the worktree anyway.

## Nitpick comments

In `@documentation/notes/dev.decision-log.md`:

- [x] Around line 53-54: replace the plain-text reference
      "the 2026-05-26 secrets-suppression task note" with the Dendron wikilink
      `[[ka.completed.2026.2026-05-26-secrets-suppression]]`.

In `@tests/runtime-config_test.ts`:

- [x] Around line 883-891: add `{ mode: null }` to the invalid
      `secretsPolicy` cases so config tests assert explicit null is rejected.
