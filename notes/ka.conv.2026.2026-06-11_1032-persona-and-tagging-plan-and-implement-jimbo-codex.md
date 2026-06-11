---
id: persona-and-tagging-plan-9bcf7d71
title: 'persona and tagging plan and implement @jimbo'
desc: ''
created: 1781199137946
updated: 1781199137946
participants: [djradon, codex.gpt-5.5]
kato-sessionIds: [019eb70c-941d-7663-9e6f-720efdf3093c]
kato-workspaceIds: [da7fa4de-adc2-41bb-99be-8dcd8071babb]
kato-recordingIds: [9bcf7d71-6724-4034-8fc1-875b8bf1f22e]
---

# User_2026-06-11_0719_26

persona and tagging plan and implement @jimbo

Please review [task.2026.2026-05-28-persona-support.md](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md) and let me know your thoughts. Goal is to build out this task note.

# gpt-5.5_2026-06-11_0719_33

I’ll read the task note alongside the repo’s developer guidance first, then I’ll trace the relevant tagging/persona code enough to give you grounded feedback instead of vibes in a trench coat.

The task note is still very skeletal, but the shape is good: it’s mixing three related contracts that need to be separated cleanly: participant identity, provider/model provenance, and capture metadata/tagging. I’m going to inspect the existing writer/session code so the plan fits Kato’s current seams and test expectations.

One important thing jumped out already: the current code already has `participants` and `tags` frontmatter plumbing, but `participants` is doing double duty by mixing the human participant username with model-derived assistant identifiers like `codex.gpt-…`. That makes your instinct to split `model`/assistant provenance out of participant identity look correct.

I’m narrowing this toward a task-note update rather than jumping into implementation: the implementation likely touches parser/extraction, contract types/config, writer frontmatter, filename template resolution, and web start actions. The note should make those phases explicit so this doesn’t become a half-invisible metadata migration.

I’d push back slightly on overloading the existing command path syntax for `@name`: Kato currently rejects mention-style command arguments as paths, which is good. The cleaner contract is “mention metadata is extracted from the user text around the capture boundary,” while command arguments remain filesystem paths.

The task note also needs to call out web-created recordings separately from in-chat commands. The web path currently creates a frontmatter-only file and arms future writes, so any pre-edit UI has to feed metadata into that start action before the writer gets called.

There’s one naming trap here: “persona” could mean the human username, an LLM/provider/model identity, or a reusable prompt/personality. I’m going to encode the task note so “participant username” is the human-facing `@name` value, and “assistant model provenance” is separate metadata. That should keep the future UI sane.

I’m going to rewrite the task note now: keep the original intent, but make the contract sharper and call out the risky ambiguity around whether `@jimbo` names the human participant, an assistant persona, or just capture metadata.

The patch missed because the note has a slightly different blank-line shape than my context expected. I’m refreshing the file and then I’ll apply the rewrite against the exact current contents.

I have the exact file now. I’ll keep the Dendron frontmatter intact and replace only the task body below it.

The note is updated. I’m doing a quick read-through now for formatting, accidental raw control-command lines, and whether the recommendations still match the code paths I inspected.

The sanity check looks good: no raw control-command lines, and I left the Dendron frontmatter untouched. One subtle note: because this task file is currently untracked in git, `git diff` doesn’t show it; I’m checking status so I can report the exact scope cleanly.

Updated [task.2026.2026-05-28-persona-support.md](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md) with a fuller plan.

My main pushback/thought: `@jimbo` needs one more semantic decision before implementation. If it means “the assistant/persona I’m addressing,” it should not be blindly routed through the same `participantUsername` path that labels user turns, or headings can become wrong. I captured that as an open issue and recommended separating human participant label, assistant/persona display label, and assistant model provenance.

I also wrote in these core decisions:
- keep command arguments as filesystem paths, not mentions
- move model-derived entries out of `participants` into plural `assistantModels`
- make metadata output-scoped, not a mutation of user config
- persist resolved metadata on workspace output state for future appends/re-arms
- split tagging into workspace defaults vs UI suggestions

No tests run since this was documentation-only. The repo already had other modified/untracked files; I only touched the persona task note.

# User_2026-06-11_0735_28

what about, instead of @jimbo, we just have the rule of "if the first line starts with persona-name:" (e.g. "jimbo: test this") and the persona name is defined in the workspace config, then the persona is listed as a participant instead of the model name. 

We add a seaparte "models:" frontmatter field. And for the headings, it would looke like "jimbo_gpt-5.5_2026-05-11_1623_16" only if a persona name were detected. 

I'm curious what our current "mid-conversation model switch" detection and recording looks like. If we can pick up model changes, do we append to frontmatter and change headings.

If we're going to have tagging, it'd be nice to add tags after recording has started too.

# gpt-5.5_2026-06-11_0735_42

That rule feels cleaner than `@jimbo` to me: `persona-name:` is less likely to collide with path syntax, and making it workspace-config-gated prevents random “Note:” or “TODO:” lines from becoming people. I’ll verify the current model-switch/render behavior in the writer so I can answer that part concretely.

The writer confirms the shape: assistant message headings are model-based today, and tool-call headings follow the latest assistant model the writer has seen. I’m checking provider parsing too, because the answer depends on whether model changes actually make it into `ConversationEvent.model`.

Current model detection is provider-data-driven: Claude, Codex, and Gemini parsers all populate `event.model` when the source transcript gives them one. There isn’t a separate “switch detector”; the renderer just reacts to each assistant event’s model field.

I’m going to fold this into the task note because it materially improves the plan: config-gated `persona:` prefix, plural `models:` frontmatter, persona+model headings only when a persona is detected, and post-start tag mutation.

I found one wrinkle worth capturing: today, if the provider gives us a new model on a later assistant event, frontmatter will pick that up on subsequent appends because the writer merges participant arrays. Headings react per event too, but only when a new heading is emitted, so consecutive assistant events with different models may not visibly split unless the role changes or a tool heading appears.

I’ve updated the note. I’m reading the edited sections once more because the scenario table is new, and I want to make sure the “Decisions” section didn’t get lumpy from the rewrite.

There was one leftover duplicate decision from the earlier sketch. I’m trimming that now so the note says one thing, cleanly.

The note is clean now. I’m doing one final lightweight check for stale `@jimbo`/`assistantModels` wording and raw control-command lines.

The stale search only finds the “superseded decision” note, which is intentional. No tests needed here since it’s still task-note work.

One extra important detail surfaced: Codex currently keeps the first `turn_context.model` it sees and ignores later model values, so mid-conversation switch detection is not uniform across providers. I’m adding that nuance to the task note because it affects the implementation.

Done. The note now distinguishes the desired future model-switch behavior from today’s provider-specific reality.

I like `persona-name:` better than `@jimbo`. Config-gated prefixes are much safer: `jimbo: test this` only means persona metadata if `jimbo` is defined in the workspace config, while random `note:`/`todo:` prose stays normal content.

I updated [task.2026.2026-05-28-persona-support.md](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md) to reflect that direction:
- configured `persona-name:` prefix instead of `@name`
- plural `models:` frontmatter instead of `assistantModels`
- `participants:` gets persona/human labels, not model names
- headings become `jimbo_gpt-5.5_2026-05-11_1623_16` only when a persona is detected
- post-start tag editing is in scope
- added a scenario table for the command/state-machine implications

Current model behavior is mixed:
- `ConversationEvent.model` exists on assistant messages in the shared contract: [events.ts](../../../../../shared/src/contracts/events.ts:56)
- headings use `event.model` when present: [markdown_writer.ts](../../../../../apps/daemon/src/writer/markdown_writer.ts:771)
- tool-call headings use the latest assistant model the writer has seen: [markdown_writer.ts](../../../../../apps/daemon/src/writer/markdown_writer.ts:856)
- frontmatter currently puts model-ish values into `participants`: [recording_pipeline.ts](../../../../../apps/daemon/src/writer/recording_pipeline.ts:785)

Important caveat: Codex currently captures only the first `turn_context.model` because of `if (!model && payload?.["model"])`: [parser.ts](../../../../../apps/daemon/src/providers/codex/parser.ts:309). Claude/Gemini are more per-message. So we should explicitly fix model accumulation as part of the `models:` work.

# User_2026-06-11_0742_18

Perhaps we split tags into a separate task. Agree with everything else, including fixing model accumulation. See also [task.2026.2026-05-11-per-output-writer-controls.md](../../../../../documentation/notes/task.2026.2026-05-11-per-output-writer-controls.md) -- that could relevant as well.

# gpt-5.5_2026-06-11_0742_31

I agree: tags are related but they have their own lifecycle and UI surface, especially once “edit after recording starts” enters the picture. I’ll read the per-output writer controls note, then I’ll split tagging out of the persona task and create a focused tagging task note that cross-links the relevant work.

The per-output writer-controls note gives us a useful pattern: persisted output metadata is the source of truth, frontmatter is descriptive, and stopped outputs can still be edited for future re-arm. I’m going to apply that same shape to the new tagging task, while narrowing persona support back to personas plus model accumulation.

I’m also removing one small cross-contamination I noticed: the persona config example still had `tags?: string[]`. If tags are their own task, persona definitions should stay persona-shaped and any persona-associated default tags can be picked up later by the tagging task.

Now I’ll add the separate output-tagging task note. It will explicitly reuse the per-output writer-controls source-of-truth pattern instead of inventing a parallel persistence model.

The split note is created. I’m checking both notes now for stale tag scope in the persona task and for Dendron/control-command formatting issues.

The split looks good overall. I’m tightening a couple of remaining wording crumbs in the persona note so it doesn’t quietly keep tag scope in its non-goals/open issues.

I’ll run one last quick verification on the two notes, then I’ll summarize the split.

Agreed, and I split it.

Updated [persona support](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md) so it now stays focused on:
- configured `persona-name:` detection
- persona frontmatter/filename/heading behavior
- `models:` frontmatter
- fixing model accumulation
- persisted output metadata, using the same source-of-truth instinct as per-output controls

Created [output tagging](../../../../../documentation/notes/task.2026.2026-06-11-output-tagging.md) for:
- workspace default tags
- tag suggestions
- per-output persisted tags
- post-start tag edits
- tag-only frontmatter mutation
- optional in-chat tag mutation later

I used [per-output writer controls](../../../../../documentation/notes/task.2026.2026-05-11-per-output-writer-controls.md) as the pattern: persisted output metadata is authoritative; frontmatter is descriptive; stopped outputs can still be edited for future re-arm.

No tests run since this was documentation-only.

# User_2026-06-11_0752_58

For output tagging, I'd like to be able to register possible tags in a per-workspace tag-library via web (and I suppose the CLI too?). Since we also want per-workspace possible personas, I think we upgrade /workspaces endpoint with a simple CRUD page (or popover) that allows editing those things. Perhaps we pave the way with a separate task for adding an "Edit" button that for now just lets us change the "wikilink scope" and maybe some of those other workspace config values. (But I'm open to just starting this edit page with one of the other two tasks.)

Thinking critically, the workspace-config is meant to be shared, and personas might be personal, so I suggest we support personas in the user-level kato config file too. Perhaps tagging as well? So those would be cross-workspace tags and personas.

# gpt-5.5_2026-06-11_0753_09

That’s a good bit of pressure on the design. I’m going to inspect the current workspace config/user config/web workspaces surfaces, then I’ll update the task notes so we don’t accidentally put personal persona data into a shared workspace file by default.

The current web Workspaces page already does a few mutations, but not general workspace-config editing. Since shared workspace config and personal user config have different audiences, I’m going to make that split explicit in the task docs and add a small prerequisite task for a workspace edit surface.

One critical design point I’d push: user-level tag/persona libraries should be suggestions and detection vocabulary by default, but not automatic write defaults unless explicitly enabled. Otherwise a personal taxonomy can quietly leak into shared workspace notes. I’ll encode that boundary in the docs.

Persona support now has both shared workspace personas and personal user-level personas. Next I’m updating the tagging task with the same shared-vs-personal distinction and a cautious defaulting rule.

I’m adding a separate workspace-config editing task now. It will cover the Workspaces “Edit” button/page foundation, starting with existing shared config values like filename/default output/timezone/wikilink-related writer flags before personas or tags pile on.

The new workspace-editing task is in place. I’m doing a consistency pass across the three notes for Dendron link format, control-command lines, and whether user-level vs workspace-level ownership is stated cleanly.

I spotted a couple of wording places where the docs still implied “workspace-only” personas. I’m tightening those and adding the CLI/user-settings surfaces explicitly, because otherwise the implementation plan would under-spec how personal libraries get managed.

Now I’m making the same management-surface distinction in the tagging task: shared tag libraries on Workspaces/edit and CLI; personal tag libraries in Settings/user CLI.

I’m cleaning the workspace-edit task wording so it says “tag and persona” plainly instead of hiding the distinction behind slashy shorthand.

I’ll do a final pass across the touched notes for link/control-command hygiene and then I’ll give you the short design readout.

There’s one awkward line wrap in the workspace-edit note from my last patch. I’m smoothing that before wrapping up.

I’m checking final status and making sure the only weird search hit is the deliberate “assistantModels is superseded” note.

I agree with the split, with one important boundary: shared workspace config should hold shared vocabulary; user config should hold personal vocabulary.

I updated the docs accordingly:

- [persona support](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md) now supports both shared workspace personas and personal user-level personas, with personal editing living in user settings/CLI, not shared workspace config.
- [output tagging](../../../../../documentation/notes/task.2026.2026-06-11-output-tagging.md) now has shared workspace tag libraries plus personal user-level tag libraries. User-level tags are suggestions by default, not automatic write defaults, to avoid leaking personal taxonomy into shared notes.
- Added [workspace config editing](../../../../../documentation/notes/task.2026.2026-06-11-workspace-config-editing.md) as a paving task for a `/workspaces` Edit surface. First slice edits existing shared config values like output dir, filename template, timezone, frontmatter toggles, and wikilink-related writer flags.

My design read: start with the workspace edit page first. It gives us a clean home for shared tag/persona libraries later, while user-level tag/persona libraries can land in Settings and CLI without muddying shared config.

No tests run; documentation-only changes.

# User_2026-06-11_0806_40

OK, we can start with the workspace edit page.

Please add to the other two tasks updating the functionality on /settings for user-level tag library and persona names.

# gpt-5.5_2026-06-11_0806_49

Got it. I’ll update the persona and tagging task notes so user-level libraries explicitly land in `/settings`, while the workspace edit page remains the shared-config surface.

I’ve got the broad split in, but you asked specifically for `/settings`, not just “user settings.” I’m making that explicit in both task notes now so it can’t get lost in implementation.

Persona task now says `/settings` directly. I’m applying the same direct wording to the tagging task.

I’ll verify the exact `/settings` mentions and status, then I’ll wrap up.

The two requested task notes now explicitly call out `/settings`. I’m doing one quick status sanity check because the workspace-editing note was a new file from the previous step and I want the final summary to be accurate.

Interesting, git only shows the two task notes modified now, so the workspace-editing note is already being treated outside this status slice. The requested changes are scoped to persona/tagging notes.

Done.

Updated:

- [persona support](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md): user-level persona names/libraries are now explicitly managed through `/settings`, with CLI support too. Shared personas remain under workspace config/editing.
- [output tagging](../../../../../documentation/notes/task.2026.2026-06-11-output-tagging.md): user-level tag libraries are now explicitly managed through `/settings`, with CLI support too. Shared tag libraries remain under workspace config/editing.

I also added `/settings` web action/test/implementation-plan bullets to both notes. No tests run; docs only.

# User_2026-06-11_0808_08

unrelated: I see on the /settings page I have "Exclude me from participant list" checked. But I believe I am not getting excluded. (I don't want to be, just interesting that might not be working as intended. I barely even recognize that functionality). Can you check?

# gpt-5.5_2026-06-11_0808_22

I’ll trace the setting from `/settings` through user config resolution and writer output, then I’ll inspect the active archived note enough to see whether the observed behavior is a real bug or just an unclear label/old output.

The first path I tried was off by one directory, so I’m locating the archive from the repo root now. The code trace already suggests this setting only suppresses Kato’s configured username in frontmatter; it does not remove model-derived `participants` entries or change headings/content.

I’m tracing this through the config resolver and the writer rather than guessing from the checkbox label. So far it looks like the setting is narrower than the UI implies: it suppresses adding your configured username to frontmatter participants, but it does not touch headings or model-derived participant entries.

I found the active archive note you had open, and its frontmatter does include `participants: [djradon, codex.gpt-5.5]`. Now I’m checking whether that is stale frontmatter, model accumulation, or an active resolver issue.

The workspace config for that archive has `addParticipantUsernameToFrontmatter: true`, so the setting would matter for newly computed frontmatter. The interesting bit is that existing notes are merged rather than fully rederived, so I’m looking for whether Kato ever removes an already-present username when the checkbox later changes.

Your actual user config file is at `~/.kato/kato-user-config.yaml`; I’m going to inspect just that config shape next. The code path is deterministic enough that we can separate “checkbox persisted correctly” from “writer behavior after that setting is true.”

The persisted checkbox is definitely `true` in your user config. The code also respects that when computing `participantUsername`, so this is not a simple “settings page failed to save” bug.

There are two separate mechanisms here. New-file rendering only includes the user if `frontmatterParticipants` has one, but appending/overwriting existing markdown uses an accretive frontmatter merge, which can add values and does not remove old participants.

There is also an explicit test locking in that accretive behavior: if existing frontmatter has `participants: [djradon]` and a later append only supplies the assistant participant, the result remains `[djradon, codex...]`. So the observed old note is consistent with current design, even with the checkbox now checked.

The focused resolver test passes: when `excludeMeFromParticipantList` is true, the resolver returns `undefined`. My filter for the writer test was too narrow and filtered everything, so I’m running the full writer markdown test file once to verify the merge behavior instead of pretending that counted.

So the short version is: the persisted setting and resolver are working, but the product surface makes it look like a broader scrubber than it is. I’m doing one final status check to make sure I didn’t touch code while investigating, then I’ll summarize the exact behavior and the likely fix choices.

I checked. The checkbox is saved and the narrow code path works: `excludeMeFromParticipantList: true` is in `~/.kato/kato-user-config.yaml`, and [participant_username.ts](../../../../../apps/runtime/src/config/participant_username.ts:71) returns no human username when it’s true.

The confusing part: it is not a scrubber. Existing markdown frontmatter is merged accretively, so old `participants: [djradon, ...]` entries are preserved rather than removed. Also, [recording_pipeline.ts](../../../../../apps/daemon/src/writer/recording_pipeline.ts:785) still adds assistant/model participants like `codex.gpt-5.5`, so `participants:` can still appear even when “exclude me” is on.

Your open archive note already has `participants: [djradon, codex.gpt-5.5]`, which is consistent with stale/accretive frontmatter, not necessarily a current resolver failure.

I ran:

```sh
deno test --allow-env --allow-read --allow-write tests/participant-username-resolver_test.ts
deno test --allow-env --allow-read --allow-write tests/writer-markdown_test.ts
```

Both pass. My take: behavior is technically working, but `/settings` copy is misleading. It should probably say “don’t add my configured username to new/updated frontmatter” rather than implying it removes you from participant lists during rendering. The persona/model split task will also clean up the model-in-`participants` confusion.

# User_2026-06-11_0816_49

We could also have a pre-person/tagging task for updating session settings/metadata. And perhaps per-recordings stuff too. Presumably a tag in a session translates to a tag in a recording/output.

# gpt-5.5_2026-06-11_0817_01

That feels like the right layer to pull forward. I’m going to look at the existing session/output metadata shape and the two task notes, then I’ll add a small precursor task note and wire the dependencies so persona/tagging don’t have to smuggle in metadata plumbing.

The output tagging note already says “store per-output tag choices in persisted session metadata,” but your suggestion exposes a broader missing piece: we should make “edit session/output metadata” its own substrate task, then let tags/personas/writer controls ride on it. I’m checking the current contract names now so the new task note uses the repo’s actual vocabulary.

I’m going to make this a docs-only change: add a precursor task note for session/output metadata editing, then adjust the tagging and persona notes to depend on that instead of each inventing their own persisted metadata shape. I’ll treat session-level tags as inherited defaults for outputs, not as frontmatter-only labels.

One bit of pushback I’m baking into the note: per-recording-cycle metadata sounds useful, but it should not be the first persistence primitive. Top-level markdown frontmatter describes the output file, so cycle-specific tags in one file get messy fast; better to support session defaults and output-level metadata first, then reserve cycle metadata for audit/history UI.

The new precursor note is in place. Next I’m updating output tagging so “session tag becomes effective output tag” is an explicit contract, with output-specific tags still living on the output.

Now I’m doing the same for persona support: keep persona-specific detection/rendering in that task, but make the persisted metadata container come from the new foundation note.

The task notes now point to the new metadata foundation. I’m checking the edited docs for formatting/wording and the git diff so I can catch any accidental churn.

The docs read cleanly. I’m going to run a targeted markdown format check on just these task notes; if the formatter wants mechanical wrapping, I’ll apply that rather than leaving little paper cuts.

Deno didn’t consider those Markdown files as formatter targets with the default invocation, so I’m leaving them as hand-wrapped task notes. I’m doing one final `git diff --stat`/status pass now.

I spotted one naming nit before closing this out: the new note title reads a little like “metadata for session output” rather than “session and output metadata.” I’m tightening that title only.

That title reads better. One last status check, then I’ll summarize the note structure and the couple of product decisions I encoded.

Done. I added a precursor task note: [task.2026.2026-06-11-session-output-metadata.md](../../../../../documentation/notes/task.2026.2026-06-11-session-output-metadata.md).

I also updated:

- [task.2026.2026-06-11-output-tagging.md](../../../../../documentation/notes/task.2026.2026-06-11-output-tagging.md)
- [task.2026.2026-05-28-persona-support.md](../../../../../documentation/notes/task.2026.2026-05-28-persona-support.md)

The main decision I encoded: session-level tags/settings are inherited defaults for outputs. So yes, a tag on a session resolves into effective tags for recordings/outputs, and that effective output metadata is what gets written to frontmatter.

I pushed back a bit on per-recording-cycle metadata: the note leaves room for it, but doesn’t make it the first primitive, because one markdown output can contain multiple recording cycles and top-level frontmatter can’t represent cycle-specific tags cleanly without getting fussy.

No code tests were needed; this was docs-only. `deno fmt --check` didn’t treat these Markdown files as formatter targets with the default invocation.

# User_2026-06-11_0835_38

I didn't mean for per-recording-cycle, just per-recording. But I guess we don't have a separate recording concept, right?

# gpt-5.5_2026-06-11_0835_47

I’m going to sanity-check the current naming in the contracts before I answer, because Kato uses “recording” in a few adjacent ways and it’s easy for the nouns to get blurry.

You’re right to separate those. I found the current split: the persisted contract has `workspaceOutputs[]` and `recordingCycles[]`; the runtime has an in-memory `ActiveRecording`, but there isn’t a durable top-level “Recording” entity independent of an output.

I think I overcorrected in the note by talking about cycle metadata. I’m going to tighten that task note now: “per-recording” should map to the durable workspace output for now, because Kato doesn’t currently have a persisted Recording object separate from the output/cycle machinery.

I’ve corrected the note so “per-recording” is not treated as “per-cycle.” It now says the durable write target is workspace output metadata, while the UI may still call that workflow a recording.

I noticed one leftover phrase about “reserved recording metadata” in the mutation helper section. I’m removing that so the note stays output-centric all the way through.

That cleanup is done. I’ll do a quick final grep/status and then answer the conceptual question plainly.

Yes, exactly. Kato has “recording” as a user/workflow concept, but not as a separate durable top-level entity right now.

Persisted state is basically:

- session: `SessionMetadataV1`
- output: `workspaceOutputs[]`
- lifecycle history on that output: `recordingCycles[]`

There is also an in-memory `ActiveRecording`, but it’s runtime bookkeeping, not the durable metadata home. So for “per-recording” tags/persona/settings, the practical first home is per-output metadata. The UI can still call it editing a recording, but the write should land on the workspace output represented by that row.

I updated [task.2026.2026-06-11-session-output-metadata.md](../../../../../documentation/notes/task.2026.2026-06-11-session-output-metadata.md) to remove the per-recording-cycle framing and say this explicitly. No tests needed; docs-only. I also noticed an unrelated modified `apps/daemon/src/orchestrator/provider_ingestion.ts` in the worktree and left it untouched.

# User_2026-06-11_0839_29

But your push-back still stands, right? The correct way to implement per-recording settings might be to create a new conceptual thing in code that corresponds to a recording that is composed of recording-cycles? I'm fuzzy on why we even did the cycles to begin with, but I'm guessing it was for a good reason that would make a simple "recording" concept tricky.

# gpt-5.5_2026-06-11_0839_41

I’m going to answer from the current contract rather than touching the notes again right away. The useful distinction is whether “recording” means “the saved thing” or “one active interval”; Kato currently has those split as output vs cycle.

Yes, the pushback still stands, but more precisely:

Kato probably needs a clearer **conceptual** “recording” in the product/model language. But I would not rush to add a separate persisted `Recording` entity unless we find settings that truly cannot belong to the output.

Right now `SessionWorkspaceOutputStateV1` is basically “the durable recording output”: destination, desired on/off state, write cursor, active cycle id, and `recordingCycles[]`. The cycles exist for a good reason: one saved output can be stopped and restarted multiple times, and Kato needs contiguous start/stop spans with cursors and ids. See [session_state.ts](../../../../../shared/src/contracts/session_state.ts:12) and [runtime_workspace_output_state.ts](../../../../../apps/daemon/src/orchestrator/runtime_workspace_output_state.ts:94).

So I’d model the language like:

- **Session**: provider conversation.
- **Recording output**: the saved artifact/destination plus settings/metadata.
- **Recording cycle**: one start/stop interval of that output.

For tags/persona/writer settings, output-level still feels right because they affect the saved artifact and its frontmatter. A future separate `Recording` entity might just be a rename/wrapper around today’s workspace output, not a whole new persistence layer.

I updated the metadata task note to preserve that open issue: don’t add a new persisted Recording entity yet, but acknowledge that `/recordings` UI may deserve a cleaner “recording output” concept.

# User_2026-06-11_0843_18

Would "change the name of the recording used in the UI" be a "setting that truly cannot belong to the output"

# gpt-5.5_2026-06-11_0843_39

Not by itself, no. I’d still put that on the output.

If the UI “recording name” means “the label for this saved recording/output row,” then it has exactly the same lifecycle as the output: destination, frontmatter, restart/re-arm, tags, persona, writer settings. So `outputMetadata.displayName` or `outputMetadata.label` fits.

I’d only avoid output-level if the name means one of these:

- **Session name**: one label for the provider conversation across all outputs.
- **Cycle name**: a label for one start/stop interval inside a longer output.
- **Artifact title**: the markdown note title/frontmatter title, which may deserve explicit sync rules with the file.

My preference: add an output-level `displayName?: string` later, and define whether it is UI-only or also updates markdown `title`. I’d avoid calling it `recordingName` in the contract because that bakes in the fuzzy noun.

# User_2026-06-11_0845_43

Yeah, it would probably update the markdown title too. 

The sense I'm getting is that we should've just called outputs "recordings"

# gpt-5.5_2026-06-11_0845_55

That’s a useful naming conclusion, and I think it belongs in the metadata note. I’m going to add a short “recording output” naming decision plus title/display-name semantics, so the task doesn’t keep using “output” as if it were obvious product language.

I agree with the drift, with one caveat I’m writing down: “recording” is the better user-facing noun, but the contract should still preserve that the thing is an output/artifact with a destination. That keeps capture/export/frontmatter behavior from getting weird later.

I added the title/display-name semantics and the naming decision to the metadata task. Now I’m doing a quick readback to make sure the note doesn’t overpromise a schema rename.

Yeah, I think you’re basically right: product-language-wise, these probably should have been called “recordings” earlier.

My refined take:

- In the UI: call them **recordings** when users are managing saved conversation recordings.
- In developer docs where destination/artifact semantics matter: call them **recording outputs**.
- In the persisted contract for now: keep `workspaceOutputs[]`, because it already owns destination, write cursor, active state, cycles, and future metadata.

For “rename recording,” I added `displayTitle?: string` to the proposed output metadata shape in [task.2026.2026-06-11-session-output-metadata.md](../../../../../documentation/notes/task.2026.2026-06-11-session-output-metadata.md). The note now says changing that should best-effort update markdown frontmatter `title`, but should not silently rename/move the file path. File rename/retarget should stay a separate explicit action.
