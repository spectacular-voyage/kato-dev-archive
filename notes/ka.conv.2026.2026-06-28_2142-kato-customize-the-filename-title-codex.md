---
id: kato-customize-the-filen-103a1419
title: 'kato: customize the filename/title'
desc: ''
created: 1782708179466
updated: 1782708179466
participants: [djradon, codex.gpt-5.5]
kato-sessionIds: [019f107e-c42b-77c1-b721-9faa380c54c2]
kato-workspaceIds: [da7fa4de-adc2-41bb-99be-8dcd8071babb]
kato-recordingIds: [103a1419-a809-4aef-a97e-f6c0f41cc947]
---

# User_2026-06-28_1612_13

kato: customize the filename/title 

I thought we had a task to add "custom snippetSlugs" so that if I forget to give a good "first line" in a conversation, I can customize. We'd talked about adding a popover for the "new capture"/"new recording" buttons that let you set tags, but I can't find references to customizing the whole filename (or snippet slug).

# gpt-5.5_2026-06-28_1612_20

I’ll go spelunking through the repo notes first, starting with the required dev guidance, then I’ll search for the filename/title/snippet slug threads. If it exists, we’ll pin it down; if it doesn’t, I’ll call that out plainly.

I found the likely neighboring archive path in your IDE tabs is outside the repo root, so my first search only hit current Kato notes. The strongest hit so far is the session/output metadata task, which explicitly says titles should not implicitly rename files.

There’s a useful nuance showing up: “title” exists today as `displayTitle` metadata, while path/filename changes were deliberately carved out as a separate explicit action. I’m checking whether there’s already a planned task for that separate action or only adjacent notes.

No dedicated “custom snippet slug” note turned up. The closest committed plan is in persona support, where “optional destination filename/path” is mentioned as part of the New Capture/New Recording pre-edit flow, but it’s underspecified enough that I’d call your memory directionally right and the docs incomplete.

I’m going to make this concrete in the docs rather than leave it as a hazy “we probably meant that.” The shape I’m seeing is: add output-scoped `filenameSlug`/filename override for creation time, keep `displayTitle` separate, and treat post-start file moves as an explicit rename/retarget action.

I’m adding this as a new task note and linking the neighboring persona/tagging work to it. I’m going to be a little opinionated in the task: custom slug and custom full path are not identical features, because one keeps Kato inside the normal template while the other bypasses more of the path policy.

I’ve got enough context now. I’m making a small docs-only change: one new task note plus a few backlinks from the adjacent planning notes so this doesn’t disappear again under “persona” or “tagging.”

The docs patch is in place. I’m doing a quick sanity pass now for the exact references and diff shape, mostly to catch typo-level problems in the Dendron links before I call it done.

The backlinks look clean. One caveat: because the main task note is a new untracked file, `git diff` doesn’t include it by default, so I’m checking status and the new file directly.

I couldn’t find a dedicated “custom snippetSlug” task. Your memory was close, though: persona support had a vague “optional destination filename/path” mention, and session metadata explicitly says title edits should not rename files.

I added a proper task note for it: [task.2026.2026-06-28-output-filename-title-overrides.md](../../../../../documentation/notes/task.2026.2026-06-28-output-filename-title-overrides.md:1).

I also linked it from:
- [persona support](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md:248)
- [output tagging](../../../../../documentation/notes/task.2026.2026-06-11-output-tagging.md:164)
- [roadmap](../../../../../documentation/notes/roadmap.md:24)
- [product ideas](../../../../../documentation/notes/product-ideas.md:35)

I made the task deliberately distinguish `displayTitle`, `filenameSlug`, and full filename/path override. Small pushback: I don’t think “custom title” should silently rename an existing file. Creation-time slug override is clean; post-creation rename/retarget should be explicit. No code/tests run, since this was docs-only.

# User_2026-06-28_2134_49

agreed, custom title should only be choosable when you're about to start a recording. I'm guessing you found a reference to enlarging the popover where you choose which workspace to record to? That'd be a good place to override the snippet. Do we want both a title override and a snippet override? Maybe changing the title changes the snippet, unless the snippet has been customized already.

# gpt-5.5_2026-06-28_2135_04

Yep, that workspace chooser popover is the right target. I’m going to tighten the task note so the first slice is explicitly “creation popover only,” with title seeding the filename snippet until the user hand-edits the snippet.

I found the exact existing UI reference too: the Sessions `New capture` and `New recording` popovers already remember the last submitted workspace. I’m updating the task to name that as the home for these controls, rather than vaguely saying “some creation UI.”

That edit is in. I’m doing one quick diff check, then I’ll summarize the recommendation cleanly.

Agreed. I updated the task note to make the first slice explicitly live in the existing Sessions `New Capture` / `New Recording` workspace chooser popovers: [task note](../../../../../documentation/notes/task.2026.2026-06-28-output-filename-title-overrides.md:11).

My take: yes, we want both, but not as two equal chores.

- `displayTitle`: human-facing title and markdown frontmatter `title`.
- `filenameSlug`: the `{snippetSlug}` input for the filename template.
- UI behavior: changing title updates the filename snippet while the snippet is still auto-derived; once the user edits the snippet manually, title changes stop overwriting it.
- Add a reset affordance to return the snippet to title-derived behavior.

I also tightened the task to say no post-start title-edit UI in this task. Tiny pushback: I kept “capture or recording,” because the same bad-first-line filename problem applies to one-shot captures too, and the buttons already share the popover pattern. Docs-only change, so I didn’t run code tests.

# User_2026-06-28_2141_24

ok, what's the sequencing of the relevant tasks?

# gpt-5.5_2026-06-28_2141_37

I’ll map this from the task notes rather than from memory, especially because a couple of these overlap in the same popover and metadata layer. I’m checking the writer-controls note too, since the roadmap currently treats it as the first editable-recording proof point.

I’d sequence it this way:

1. `session-output-metadata`
   Already basically landed. This is the foundation: `displayTitle`, `tags`, persona metadata containers, mutation helpers, frontmatter sync.

2. `per-output-writer-controls`
   Also appears landed. It proves the “editable output row” path works without inventing new mutation plumbing.

3. `workspace-config-editing`
   Do this before tag/persona libraries, because shared tags/personas want `.kato-workspace-config.yaml` editing. It does not strictly block title/snippet overrides.

4. `output-filename-title-overrides`
   I’d move this earlier than tagging/persona. It is smaller, directly improves the existing `New capture` / `New recording` popover, and creates the creation-time metadata pattern that tags/personas can reuse.

5. `output-tagging`
   Build on the enlarged popover and metadata path. Add selected tags at creation, then post-start tag edits.

6. `persona-support`
   I’d put this after tagging unless persona is urgent. It touches more surfaces: workspace/user config, prefix extraction, `{username}` filename behavior, headings, participants/models frontmatter.

7. Explicit existing-output rename/retarget
   Later, separate task. Creation-time snippet/title solves the common case without opening the file-move can yet.

So my recommended correction to the roadmap is: put `output-filename-title-overrides` before `output-tagging` and `persona-support`, not after them. It’s the little hinge that makes the later popover expansion feel coherent.

# gpt-5.5_2026-06-28_2141_51

I’d sequence it this way:

1. `session-output-metadata`
   Already basically landed. This is the foundation: `displayTitle`, `tags`, persona metadata containers, mutation helpers, frontmatter sync.

2. `per-output-writer-controls`
   Also appears landed. It proves the “editable output row” path works without inventing new mutation plumbing.

3. `workspace-config-editing`
   Do this before tag/persona libraries, because shared tags/personas want `.kato-workspace-config.yaml` editing. It does not strictly block title/snippet overrides.

4. `output-filename-title-overrides`
   I’d move this earlier than tagging/persona. It is smaller, directly improves the existing `New capture` / `New recording` popover, and creates the creation-time metadata pattern that tags/personas can reuse.

5. `output-tagging`
   Build on the enlarged popover and metadata path. Add selected tags at creation, then post-start tag edits.

6. `persona-support`
   I’d put this after tagging unless persona is urgent. It touches more surfaces: workspace/user config, prefix extraction, `{username}` filename behavior, headings, participants/models frontmatter.

7. Explicit existing-output rename/retarget
   Later, separate task. Creation-time snippet/title solves the common case without opening the file-move can yet.

So my recommended correction to the roadmap is: put `output-filename-title-overrides` before `output-tagging` and `persona-support`, not after them. It’s the little hinge that makes the later popover expansion feel coherent.

# User_2026-06-28_2147_13

OK, you wanna give me me a prompt for a fresh conversation? Or just do it here? Any open issues to resolve?

# gpt-5.5_2026-06-28_2147_24

We can do it here. I’ve got the relevant task graph loaded, and this is a small enough feature slice that a fresh conversation would mostly cost us momentum.

# gpt-5.5_2026-06-28_2147_34

Do it here. No fresh conversation needed.

Open issues I’d resolve with these defaults:

1. First slice: `New capture` and `New recording`
   Default: include both, because they share the workspace chooser popover and filename problem.

2. Fields: title plus snippet
   Default: show both. Title is human/frontmatter; snippet is filename seed.

3. Auto behavior
   Default: title changes update snippet until snippet is manually edited. Manual snippet edit “breaks the link.” Add a reset control to derive from title again.

4. Persistence
   Default: persist both `displayTitle` and the concrete `filenameSlug` used at creation, even if the slug was auto-derived from title. That makes the created output reproducible and avoids later title edits implying rename.

5. Full path override
   Default: defer. Custom `{snippetSlug}` solves the pain without opening arbitrary path/retarget complexity.

6. Post-start title editing
   Default: out of scope for this task.

If you did want a fresh prompt, use:

```text
Implement Kato creation-time output title and filename snippet overrides from documentation/notes/task.2026.2026-06-28-output-filename-title-overrides.md. Use the existing Sessions New Capture/New Recording workspace chooser popovers. Add displayTitle and filenameSlug creation metadata, with title-derived snippet behavior until the snippet is manually customized. Include capture and recording, defer full path override and post-start title editing. Follow documentation/notes/dev.general-guidance.md and update task checkboxes as implementation proceeds.
```

My vote: do it here with those defaults.

# User_2026-06-28_2147_44

OK, proceed

# gpt-5.5_2026-06-28_2147_50

Great, I’ll take the defaults we agreed on and implement the creation-time title/snippet path end to end: contracts, filename rendering, web actions, popover UI behavior, tests, and task checkboxes as they become true.

# gpt-5.5_2026-06-28_2147_58

I’m starting with the existing flow from the popover submit through destination resolution. The important bit is finding the single place to pass metadata into both capture and recording rather than making two nearly identical paths.

# gpt-5.5_2026-06-28_2148_05

The backend already has a good seam: web capture/recording uses the same `runSessionRecordingAction`, and generated filenames flow through `resolveWorkspaceCommandDestination`. I’m reading the route/view code next so the UI and action payload line up cleanly.

# gpt-5.5_2026-06-28_2148_17

The create action currently computes the conversation title from the first snippet and passes that to the writer. I’ll change that so a provided title wins, and pass a filename slug into the same generated filename resolver for both capture and recording.

# gpt-5.5_2026-06-28_2148_24

The UI lives outside `session_routes.ts`; that file was just URL builders. I’m narrowing in on the page renderer and loader now, then I’ll patch.

# gpt-5.5_2026-06-28_2148_32

Interesting small wrinkle: the live route serves JSON, but the actual HTML renderer is likely bundled/generated enough that simple string search missed the human labels. I’m checking the server/static composition and compiled entrypoints before touching it.

# gpt-5.5_2026-06-28_2148_38

Found it: this is a Fresh app, so the page components are under routes/islands rather than `apps/web/src`. I’m switching there for the popover changes.

# gpt-5.5_2026-06-28_2148_46

The popover is in `apps/web/islands/SessionsLive.tsx`, which is nice: its client-side state can handle “title drives snippet until dirty” without touching the server-rendered route too much.

# gpt-5.5_2026-06-28_2149_38

For the filename preview, the Sessions loader currently has workspace ids/labels but not templates. I’ll extend those workspace options with the effective `filenameTemplate`, so the popover can show whether `{snippetSlug}` is actually in play.

# gpt-5.5_2026-06-28_2150_15

I’m ready to edit now. The code changes are scoped to the shared metadata contract, workspace filename resolver, web create action, Sessions popover/view model, and focused tests.

# gpt-5.5_2026-06-28_2151_09

The first code patch is in. Next I’m adding tests around the contract, filename resolver, web action persistence/frontmatter, and the title/snippet helper behavior.

# gpt-5.5_2026-06-28_2151_37

Tests are in place now. I’m going to run the focused suite first; I expect type/format nits because several TypeScript surfaces changed at once.

# gpt-5.5_2026-06-28_2151_51

The focused tests passed first try. I’m formatting the touched files next, then I’ll run the repo type check because Fresh/Preact props are where small type mismatches like to hide.

# gpt-5.5_2026-06-28_2151_59

Formatting only changed the web action file. I’m running the broader type check now; if that passes, I’ll update the task checkboxes and do one final focused test run.

# gpt-5.5_2026-06-28_2152_10

Type check passed too. I’m going to mark the completed implementation-plan checkboxes in the task note now, then run the focused tests one more time after that docs edit.

# gpt-5.5_2026-06-28_2152_21

Small surprise: the docs changes from the planning pass are no longer showing as dirty, so this task note is already tracked/clean in the current worktree state. I’m still updating its checkboxes now so the implementation state matches the code.

# gpt-5.5_2026-06-28_2152_32

I’m going to add the small developer/user documentation updates too, not just tick the task. Since this changes a visible workflow and persisted metadata shape, the codebase overview, decision log, user recording guide, and current release notes should know about it.

# gpt-5.5_2026-06-28_2153_01

Documentation is updated and the task checklist is almost complete. I left the final CI checkbox open until I actually run the CI task.

# gpt-5.5_2026-06-28_2153_08

`deno fmt` doesn’t target these Markdown notes in this repo, so that docs-format command had nothing to format. I’m running the focused tests again, then I’ll run full CI if they stay green.

# gpt-5.5_2026-06-28_2153_17

Focused tests are green again. I’m starting `deno task ci` now; it may take a bit, but it’s the right final gate for this change because we touched shared contracts, web UI, and daemon path rendering.

# gpt-5.5_2026-06-28_2153_24

CI stopped at the audit step on an existing Vite high advisory in the web lockfile, before it reached fmt/lint/check/test. I’m going to run the remaining CI gates manually so we know whether this patch itself is clean aside from that dependency audit blocker.

# gpt-5.5_2026-06-28_2153_47

The rest of the CI gates are clean: fmt, lint, frozen check, and the full test suite passed. Full `deno task ci` itself is blocked at `deno audit` by the current Vite advisory, so I’m leaving the final CI checkbox open and doing a final diff/status pass.

# gpt-5.5_2026-06-28_2153_52

The checklist’s final validation item is too coarse now that audit is blocked upstream. I’m splitting it into “what passed” and “full CI once Vite audit is resolved,” so the task note tells the truth instead of looking half-forgotten.

# gpt-5.5_2026-06-28_2154_08

The final diff check is clean. Since this includes a visible web UI change, I’m checking the repo’s dev-server task before I decide whether to leave a local server running for you.

# gpt-5.5_2026-06-28_2154_22

The web app’s dev task is `deno task dev:web` via Vite. I’m starting it now so you can poke at the Sessions popover locally.

# gpt-5.5_2026-06-28_2154_32

Port 5173 is already occupied, so I’m starting the web dev server on the next port instead.

# gpt-5.5_2026-06-28_2154_47

The task wrapper is passing the separator through literally, so Vite is ignoring the alternate port. I’ll start Vite directly with the port flag from `apps/web`.

# gpt-5.5_2026-06-28_2154_53

Vite is running at `http://127.0.0.1:5174/`. I’m doing a quick HTTP smoke check against `/sessions`; auth may redirect, but that still confirms the dev server is serving.

# gpt-5.5_2026-06-28_2155_19

The smoke check got the expected `/sessions` -> `/login` redirect, so the dev server is alive. I’ll leave that Vite session running for you at port 5174.

# gpt-5.5_2026-06-28_2155_35

Implemented here.

What changed:
- Added `filenameSlug?: string` to output metadata and effective metadata resolution.
- Taught generated workspace filenames to use `filenameSlug` for `{snippetSlug}`, falling back safely to the extracted conversation snippet.
- Wired `displayTitle` and `filenameSlug` through web `New capture` / `New recording`.
- Expanded the Sessions popover with `Title`, `Filename snippet`, preview, and `Reset`.
- Title edits update the snippet until the snippet is manually customized.
- Persisted creation metadata onto the new workspace output.
- Updated task/user/dev/release notes.

Key files:
- [session_state.ts](../../../../../shared/src/contracts/session_state.ts)
- [runtime_workspace_paths.ts](../../../../../apps/daemon/src/orchestrator/runtime_workspace_paths.ts)
- [session_recording_actions.ts](../../../../../apps/web/src/session_recording_actions.ts)
- [SessionsLive.tsx](../../../../../apps/web/islands/SessionsLive.tsx)
- [task note](../../../../../documentation/notes/task.2026.2026-06-28-output-filename-title-overrides.md)

Validation:
- Focused tests passed: `45 passed`.
- `deno task check` passed.
- Non-audit CI gates passed: fmt, lint, frozen check, full test suite `860 passed`.
- Full `deno task ci` is blocked at `deno audit` by the existing Vite high advisory `GHSA-fx2h-pf6j-xcff`.

I also started the web dev server on `http://127.0.0.1:5174/` because `5173` was already in use. `/sessions` smoke-checks correctly with the expected redirect to `/login`.

# User_2026-06-28_2156_06

commit message please.

# gpt-5.5_2026-06-28_2156_14

```text
feat(web): add creation-time output title and filename slug controls

- add filenameSlug to output metadata and use it as the explicit {snippetSlug} source for generated workspace filenames
- wire displayTitle and filenameSlug through web New capture/New recording creation flows
- expand the Sessions workspace chooser popover with title, filename snippet, preview, and reset behavior
- persist creation metadata on new workspace outputs and render displayTitle into markdown frontmatter
- add focused contract, path, web action, and view-model tests
- update task, user guide, release notes, codebase overview, and decision log
```

# User_2026-06-28_2156_33

What's next?

# gpt-5.5_2026-06-28_2156_46

Next, I’d do this in order:

1. Commit this feature.
   Use the commit message above.

2. Fix or consciously park the Vite audit blocker.
   `deno task ci` is blocked before our code gates by the existing Vite advisory. Since non-audit CI passed, this feature is healthy, but the branch won’t get a clean full CI until Vite is updated or the advisory is otherwise handled.

3. Manual UI smoke test.
   Open `http://127.0.0.1:5174/sessions`, start a New capture/recording, and confirm:
   - title pre-fills from snippet,
   - filename snippet follows title,
   - manual snippet edit breaks that link,
   - reset restores title-derived snippet,
   - generated filename uses the snippet.

4. Then I’d go to `workspace-config-editing`.
   That unlocks shared workspace tag/persona libraries cleanly. After that: `output-tagging`, then `persona-support`.

So: commit, clear Vite audit if you want clean CI, quick UI poke, then workspace config editing.

# User_2026-06-28_2157_40

let's fix the vite audit blocker.

# gpt-5.5_2026-06-28_2157_50

Yep, let’s clear that gate. I’m going to stop the Vite dev server first, verify the advisory/fixed version from the advisory data, then update the web Vite dependency and lockfile and rerun audit/CI.

# gpt-5.5_2026-06-28_2157_54

The dev server took the interrupt; I’m checking that it actually exited before touching dependency files.

# gpt-5.5_2026-06-28_2158_05

It looks like Vite didn’t exit on the first interrupt, so I’m sending one more clean stop signal.

# gpt-5.5_2026-06-28_2158_21

The advisory’s patched 7.x version is `7.3.5`, so I’ll keep Kato on the same Vite major/minor line and move the web dependency from `^7.3.3` to `^7.3.5`, then regenerate the locks.

# gpt-5.5_2026-06-28_2158_41

I found the dependency is only in the web app lock/config, not the root lock. I’m patching `apps/web/deno.json` and letting Deno refresh `apps/web/deno.lock` through the web check.

# gpt-5.5_2026-06-28_2158_48

Now I’m refreshing the web lock/check. If Fresh pulls a compatible nested Vite range too, this should collapse all resolved Vite entries to the patched version.

# gpt-5.5_2026-06-28_2159_00

Deno resolved Vite to `7.3.6`, which is beyond the vulnerable range. I’m checking the lockfile entries and then rerunning audit.

# gpt-5.5_2026-06-28_2159_06

Audit is clean at the high-severity gate now; only two low findings remain below the configured threshold. I’m running full `deno task ci` to make sure the former blocker is actually gone.

# gpt-5.5_2026-06-28_2159_25

Full CI is green now. I’m going to update the existing task note validation checkbox and add a short release-note line for the dependency security fix, then I’ll do one last status/diff check.

# gpt-5.5_2026-06-28_2159_30

The only dirty files right now are the web dependency files, so the previous feature docs must already be committed. I’m adding a tiny note and checking the validation box now, which will make this audit fix self-documenting too.

# gpt-5.5_2026-06-28_2159_41

Docs are updated. Since Markdown is excluded from formatting and no code changed after CI except docs, I’m doing a final diff/status sanity check.

# gpt-5.5_2026-06-28_2159_56

Fixed. I updated Kato Web’s Vite dependency from `^7.3.3` to `^7.3.5`; Deno resolved it to `vite@7.3.6` in [apps/web/deno.lock](../../../../../apps/web/deno.lock). The GitHub advisory lists Vite `7.3.5` as the patched 7.x version for `GHSA-fx2h-pf6j-xcff` / `CVE-2026-53571`: https://github.com/advisories/GHSA-fx2h-pf6j-xcff

Also updated:
- [apps/web/deno.json](../../../../../apps/web/deno.json)
- [release-notes.v0.2.13.md](../../../../../documentation/notes/release-notes.v0.2.13.md)
- [task.2026.2026-06-28-output-filename-title-overrides.md](../../../../../documentation/notes/task.2026.2026-06-28-output-filename-title-overrides.md)

Validation: `deno task ci` now passes. `deno audit` still reports 2 low vulnerabilities, but no high/moderate blocker remains under the configured high-severity gate.

# User_2026-06-28_2208_39

OK, what's next?

# gpt-5.5_2026-06-28_2208_50

Next:

1. Commit the Vite audit fix.

Suggested commit message:

```text
chore(web): update vite to patched 7.3.x

- bump Kato Web Vite dependency from ^7.3.3 to ^7.3.5
- refresh apps/web/deno.lock to resolve vite 7.3.6
- document the audit fix in v0.2.13 release notes
- mark the filename/title override task fully validated now that deno task ci passes
```

2. Do a quick manual UI smoke of the title/snippet popover if you haven’t already:
   `New capture` / `New recording` on `/sessions`, title drives snippet, manual snippet breaks the link, reset restores it.

3. Then I’d start `workspace-config-editing`.

That unlocks the next arc cleanly: shared workspace tag libraries and shared personas both want an actual `.kato-workspace-config.yaml` edit surface. After that, I’d do `output-tagging`, then `persona-support`.

# User_2026-06-28_2227_46

works great. thanks! Context is getting full... you wanna proceed here, or start a new conversation?

# gpt-5.5_2026-06-28_2227_54

Nice, glad it works.

I’d start a fresh conversation for the next implementation slice. This one now has planning, implementation, CI, and dependency-audit context all mixed together, and the next task deserves a clean runway.

Use this prompt:

```text
kato: implement workspace config editing

Please continue in /home/djradon/hub/spectacular-voyage/kato. Read AGENTS.md instructions and documentation/notes/dev.general-guidance.md first.

Implement documentation/notes/task.2026.2026-06-11-workspace-config-editing.md. Current intended sequencing: this comes after completed creation-time title/filename snippet overrides, and before output tagging/persona support.

Keep the first slice focused on editing existing .kato-workspace-config.yaml fields from Kato Web:
- defaultOutputDir
- filenameTemplate
- workspaceTimezone
- markdown frontmatter toggles
- writer feature flags including Dendron wikilinks and relative-link mode

Do not implement tag libraries or persona libraries yet. Update task checkboxes as implementation proceeds, add focused tests, update developer/user docs and release notes, and run focused validation plus deno task ci if feasible.
```

Tiny extra note for future-you: `deno task ci` was passing after the Vite update, with only low audit findings below the configured high-severity gate.