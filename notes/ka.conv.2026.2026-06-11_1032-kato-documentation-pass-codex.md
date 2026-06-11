---
id: kato-documentation-pass-97056db8
title: 'kato documentation pass'
desc: ''
created: 1781199134419
updated: 1781199134419
participants: [djradon, codex.gpt-5.5]
kato-sessionIds: [019eb770-4450-74c3-bf2e-10ff2026b246]
kato-workspaceIds: [da7fa4de-adc2-41bb-99be-8dcd8071babb]
kato-recordingIds: [97056db8-91f2-46c8-b50f-d91e799a77a3]
---

# User_2026-06-11_0907_31

kato documentation pass

The documentation/notes have several issues:
- [features.md](../../../../../documentation/notes/features.md) and [roadmap.md](../../../../../documentation/notes/roadmap.md)are skeletal
-  [dev.feature-ideas.md](../../../../../documentation/notes/dev.feature-ideas.md) and [product-ideas.md](../../../../../documentation/notes/product-ideas.md) should be combined
- [dev.feature-ideas.distribution-phase-2.md](../../../../../documentation/notes/dev.feature-ideas.distribution-phase-2.md) is mostly outdated; any remaining relevant ideas should be folded into a consolidated [product-ideas.md](../../../../../documentation/notes/product-ideas.md) 
- [dev.todo.md](../../../../../documentation/notes/dev.todo.md) and [roadmap.md](../../../../../documentation/notes/roadmap.md) might be combined
- [dev.product-vision.md](../../../../../documentation/notes/dev.product-vision.md) has several issues:
  - "Operate as CLI or daemon" but the CLI can't really operate independently of the daemon.
  - tense is off for things that are accomplished; most of those can go in [features.md](../../../../../documentation/notes/features.md) ; the vision should be a simple two- or three-sentence "north star"
- [user-guide.md](../../../../../documentation/notes/user-guide.md) is skeletal, probably should be dramatically fleshed out, perhaps into multiple sub-notes

# gpt-5.5_2026-06-11_0907_37

I’ll do this as a documentation refactor, starting by reading the repo guidance note and the affected Dendron notes so I can preserve the vault conventions instead of inventing a new taxonomy.

The current split is exactly as you described: product/feature ideas are fragmented, features and roadmap are placeholders, and the vision note is carrying too much state. I’m going wider now through the notes to pull actual shipped behavior and avoid writing aspirational docs as if they exist.

I found the guidance note’s main constraint: user-facing docs can live outside `dev*`, while developer-targeted material belongs under `dev*`. I’m going to keep the public-facing roadmap/features/user guide clean, and consolidate engineering backlog material without pretending it’s user documentation.

Next I’m checking the README, release notes, and note links so the rewritten feature list is grounded in what Kato actually ships today, not just old planning language.

There are unrelated local changes already in the worktree, including a deleted task note and two modified notes I won’t touch. I’ll keep this pass scoped to the documentation notes you named and any new user-guide subnotes.

The release notes give enough history to promote shipped web/session/workspace/security behavior into `features.md`. I’m reading the CLI surface too, because user-guide content should match commands the binary actually exposes.

One quick correction in my scan: there isn’t a `help_text.ts`; the command text lives in `usage.ts`. I’m reading that now and then I’ll start writing the consolidated notes.

I’ve got the command surface and current config shape now. The resulting structure I’m going to implement is: `features` for shipped capabilities, `product-ideas` for all future ideas, `roadmap` for prioritized backlog, `dev.product-vision` as a short north star, and `user-guide` as an index with focused subnotes.

Before editing, I’m sampling the active task notes that overlap the roadmap so I don’t bury current work under old “maybe someday” items.

I’m going to edit the named notes plus create focused `user-guide.*` subnotes. The existing deprecated notes will stay as short pointers so older wikilinks don’t break, but their real content will move into the consolidated destinations.

I’m generating Dendron-style IDs for the new user guide notes, then I’ll patch the note files in one controlled pass.

I confirmed JSONL is currently an export format rather than the normal recording append path, so the user guide will phrase that distinction plainly. That’s one of those little docs traps worth catching.

I’m applying the rewrite now. The notes will deliberately separate three kinds of truth: what exists now, what is planned next, and what is merely an idea worth keeping.

The first patch is in. I’m doing a link/path sanity pass now; I already spotted that generic `[[task]]` links would be weak in this vault, so I’m checking for those and other rough edges.

The first sanity pass found only the generic `[[task]]` placeholders as actual link issues. I’m replacing those with plain `task.*` wording so the vault links stay meaningful.

Now I’m checking the rewritten notes against the actual default workspace scaffold and wikilink targets, because a docs pass should not introduce tiny lies with a confident hat on.

One small correction: my illustrative workspace config snippet used a stale output directory/filename example. The actual scaffold uses `defaultOutputDir: "."`; I’m pulling the exact default template values now and will patch that snippet.

Patching the workspace guide to use the real default scaffold values.

`deno fmt` in this repo reports “No target files found” for Markdown targets, even with `--ext md`, so I’ll validate with cheaper checks instead of forcing a repo-wide formatter that could touch unrelated files.

`git diff --check` is clean. The repo intentionally excludes Markdown notes from Deno formatting, which explains the formatter result. I’m doing one final pass for wikilinks and maybe a couple wording nits before I call it done.

The only apparent missing wikilink is an example `[[note]]` inside prose, not a real note. I’m going to avoid that false link entirely so future link checks stay quiet.

All wikilink targets in the touched/new notes resolve to existing note files now. I’m reading the final rendered text at section level and then I’ll summarize the exact changes.

The final read looks coherent and the consolidated roadmap now carries the old `dev.todo` material without leaving it stranded. I’m rerunning whitespace/status checks after the last tiny patch.

Checks are clean for the touched notes: no whitespace errors, and all wikilinks in the rewritten/new notes resolve. The worktree still has pre-existing unrelated changes in the conversation/task notes; I left those alone.

Done. I turned the docs into a cleaner set:

- Expanded [features.md](../../../../../documentation/notes/features.md) into shipped behavior.
- Consolidated feature/product/distribution ideas into [product-ideas.md](../../../../../documentation/notes/product-ideas.md).
- Collapsed [dev.todo.md](../../../../../documentation/notes/dev.todo.md) into a real backlog/roadmap in [roadmap.md](../../../../../documentation/notes/roadmap.md).
- Rewrote [dev.product-vision.md](../../../../../documentation/notes/dev.product-vision.md) into a short north-star note, including the daemon/CLI correction.
- Reworked [user-guide.md](../../../../../documentation/notes/user-guide.md) into an index and added focused subnotes for installation, quickstart, workspaces, recording, web, configuration, and troubleshooting.
- Left [dev.feature-ideas.md](../../../../../documentation/notes/dev.feature-ideas.md) and [dev.feature-ideas.distribution-phase-2.md](../../../../../documentation/notes/dev.feature-ideas.distribution-phase-2.md) as historical backlink pointers.

Validation: `git diff --check` passes, and all wikilinks in the touched/new notes resolve. `deno fmt` is not applicable here because `deno.json` excludes Markdown/docs notes. I also left the pre-existing unrelated worktree changes alone.
