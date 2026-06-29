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