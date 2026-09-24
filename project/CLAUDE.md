# Project Constitution

This file is the standing contract for how this codebase is built. It loads
into every Claude Code session automatically. Specs and plans must not
contradict it; if a spec needs to, the constitution gets amended first,
explicitly, in its own commit.

## What this project is

<!-- One paragraph. What it is, who it's for, what the core loop is.
Write this once the first spec conversation has actually happened —
don't guess at it before then. -->

## Scale: what this is actually being built for

<!-- Fill this in honestly and concretely — the rules below are only
as useful as this paragraph is specific. How many people use it, on
what machines, against what data, and what actually breaks if it goes
wrong. A real example: "One user, me, on one Mac. State in a local
JSON file I'd be annoyed to lose but could recreate. Nothing else
reads it. No deployment, no uptime requirement, no other developer."
If any of that stops being true, amend this section before the spec
that changes it, not after. -->

**Build for the scale above, and not for one this project might reach
someday.** The characteristic failure here is not sloppy code, it is
ceremony: an interface with a single implementation, a configuration
value that never varies, a strategy pattern for two branches, a
repository layer over the one database this will ever use. Each is
defensible in the abstract, each is free to add, and each is a
permanent tax on every future reader — who is one person, working
alone, months removed from the reasoning.

Six tests, applied when the plan is drafted and again at review.
They're written as tests rather than as advice because "don't
over-engineer" is not something a reader can check an actual diff
against:

1. **An abstraction earns its place at the second real caller, not
   the first imagined one.** One caller means write it inline. Two
   callers that genuinely share behavior means extract it.
2. **An extension point needs a requirement that exists now.** No
   plugin surface, no "in case we swap this out" seam. A project this
   size swaps something by editing the one place it's used.
3. **A layer needs a boundary that actually varies.** A layer that
   forwards a call without transforming it is a layer that only adds
   a file to open.
4. **Prefer deleting to configuring.** An option nobody has asked for
   is two code paths, one of which is never exercised and both of
   which need to keep working.
5. **Backward compatibility with yourself is not a constraint** —
   except for data already on disk, which is a real constraint and
   gets the full treatment.
6. **Code that arrived with a starter template isn't yours until you
   use it.** A theme, a scaffold or a framework's example app brings
   working machinery for features this project may never have. It
   passes every test above — it has callers, it has structure, it was
   written by someone competent — and it is still dead weight,
   because nothing here asked for it. Delete on first contact rather
   than deferring: the decision only gets harder once it has been read
   a few times and started to look load-bearing.

**What does not scale down.** Tests, error handling on failures that
can actually happen, data integrity and migrations for anything
persisted, care with credentials and anyone else's data, and names a
cold reader can follow. None of these are enterprise overhead; a solo
project needs them *more* than a team project does, because there is
no second reader to catch what the first missed, and the person who
comes back in six months has forgotten everything and has nobody to
ask. Simplicity means fewer moving parts, not fewer safeguards.
Dropping the parts that make code survivable isn't simple, it's debt
with a shorter fuse.

**When the two pull against each other, say which won and why** — in
`plan.md` if it's a design call, or by returning the question if it's
genuinely a fork. Don't split the difference silently; a half-built
abstraction is worse than either choice made deliberately.

**How this shows up in review.** Over-engineering contradicts this
section, which makes it blocking — but only when it is concrete and
nameable: an interface with one implementation, an unused extension
point, a configuration value never varied, a pass-through layer.
Disagreements of taste about structure are second-look notes and
never block. A reviewer that blocks on style generates noise and
gets ignored, which costs more than the abstraction would have.

## Platform

<!-- Target platform/version, language, and any "no X unless Y" rules,
each with a one-line reason. Example shape:

- **Target**: [platform + minimum version]. [Back-compat policy.]
- **[UI framework, if any]**: [framework] only. No [alternative] except
  where [specific, narrow exception] — flagged when it happens, not a
  default.
- **Language**: [language + version]. [Concurrency/typing mode and why.] -->

## Architecture

<!-- Pattern (MVVM, MVC, whatever), and the actual rules that make it
real rather than aspirational — what layer owns what, what's forbidden
from importing what, how dependencies get injected for testability. -->

## Testing

<!-- Framework, and the real requirements:
- What needs tests, and what the fake/mock boundary is.
- "A task is not done until tests exist and pass — run the command
  yourself, don't assert completion from reading the code."
- Leave room here to add principles as they're discovered — this
  section should grow over the project's life, not stay static. See
  the parent skill's "Principles worth generalizing" section for the
  kind of thing that belongs here once found. -->

## Dependencies

<!-- Default policy (e.g. "no third-party packages without discussion")
and why. -->

## Project file safety

<!-- If there's a project-format file that agents reliably mishandle
(Xcode's .pbxproj is the classic example — a semi-binary format that's
easy to corrupt with a direct edit), name it explicitly and state the
safe path around it. -->

## Involvement level

<!-- Pick one, decided in the constitution conversation, and delete the
other. The skill's default is product owner. -->

**Product owner.** The person owns `spec.md`, attests to behavior by
using the app at phase pauses, and decides escalations. They do not
approve technical work: `plan.md` and `tasks.md` are drafted by the
`sdd-planner` and signed off by the `skeptical-reviewer`, each phase
(and any task the planner marked for its own review) is reviewed by
the `skeptical-reviewer` rather than the person, and what reaches the
person is a spec-conformance summary, not an architecture review.
Implementation pauses on the cadence set below, and whenever something
unexpected bears on spec adherence.

**Technical lead.** As above, but the person also reads and approves
`plan.md` and `tasks.md`, and implementation pauses for their review
after every task the planner marked `review: per-task`.

## Pause cadence

<!-- A separate question from the involvement level, asked at the same
time and answered here. Involvement level says who approves technical
work; this says how often the build stops for the person. Changing it
later is one word on this line plus a tier-log row, same as a role
table row — and it can change mid-spec. -->

**When there's something to try** (the default).

The three values:

- **When there's something to try.** Pause after a phase the planner
  marked with a walkthrough, and run straight through the phases it
  marked `walkthrough: none`. A foundational phase that adds a type,
  a shared helper, or a test harness changes nothing the person can
  observe, so a pause there asks them to attest to something they
  cannot see — which is a gate in appearance only.
- **Every phase.** Pause after each one regardless. The most
  conservative setting, and the right one for a project whose phases
  are hard to tell apart from outside, or early in a project where
  nobody trusts the marking yet.
- **Only when blocked.** No phase pauses at all. The spec runs to the
  pre-merge sweep and the person does one walkthrough at the end,
  against the accumulated list. The most hands-off setting, and the
  one that costs the most to unwind if something went wrong early.

**Blockers pause under every cadence, including "only when blocked".**
These are not phase pauses and are never skipped:

- Either escalation trigger: something in the design turns out
  infeasible or needs real rework, or an unknown surfaces that would
  materially change the project's direction.
- A product question `spec.md` doesn't settle.
- The escape hatch firing twice on one task, which means the task
  list itself is wrong.
- A sweep or review finding that can't be resolved without changing
  what the spec promised.

**Nothing skipped is dropped.** Every phase that runs without a pause
adds its walkthrough items — or, if it had none, its one-line reason —
to a running list in `tasks.md`, and that list is what the person
walks through at the close-out. Skipping a pause defers the person's
check; it does not remove it. The tier log records which phases ran
unpaused, so a defect found late can be traced to the phase that
introduced it.

## Model policy

<!-- Decided once, at the start: the person is asked directly, "Fable
or Opus?", and this section keeps the profile they chose and deletes
the other. Adjust the tier names as models change; the roles don't. A
project can move between profiles later — change the names here and
swap the settings file, one commit — and the tier log shows from which
spec. -->

**Fable profile.** Top tier `fable`; implementation tier `opus`;
session tier `claude-fable-5-1` at medium effort (the top and session
tiers are the same model at different effort). The fallback session
model is `claude-opus-5-5`. Settings from the skill's
`project/.claude/settings.json`.

**Opus profile.** Opus for everything: top tier `opus`;
implementation tier `opus`; session tier `claude-opus-5-5` at medium
effort. Every dispatch runs at high effort — the agent definitions'
own default — and only orchestration runs at medium. Nothing runs on
Fable: the planner and sign-off dispatches carry no override, the
close-out goes to `sdd-implementer`, and the top-tier fallback below
never applies. Settings from the skill's
`project/.claude/settings.opus.json`.

These names are the only place a model is spelled out; everything
below refers to the roles.

### The role table

Every dispatch in this project resolves here. Cells hold tier names,
never model IDs, so switching profile re-points every row at once and
the three names above stay the only place a model is spelled out.
"Override" means the orchestrator passes a per-call model override on
that dispatch; without one, the agent's own frontmatter applies, and
every agent definition defaults to the implementation tier except
`sdd-implementer-fable`, which pins the top tier's model at medium.

| Role | Dispatched as | Model | Effort |
|---|---|---|---|
| Spec conversation | the spec session itself | session tier | high (raised per session) |
| Plan and tasks draft | `sdd-planner` | **top tier** (override) | high |
| Plan and tasks sign-off | `skeptical-reviewer` | **top tier** (override) | high |
| Decision review | `skeptical-reviewer` | **top tier** (override) | high |
| Task implementation | `sdd-implementer` | implementation tier | high |
| Close-out task | `sdd-implementer-fable` | top tier | medium |
| Per-task and phase review | `skeptical-reviewer` | implementation tier | high |
| Pre-merge sweep | `skeptical-reviewer` | implementation tier | high |
| Orchestration and bookkeeping | the session itself | session tier | medium |

**Moving a role.** Edit its row, nothing else. To step a role down,
replace **top tier** (override) with "implementation tier (no
override)"; the dispatch then carries no override and the agent runs
at its own default. To change the implementer, change the agent name
in that row — both definitions stay installed, so it is a word, not a
reinstall. Under the Opus profile the top tier *is* the
implementation tier, so the overrides become no-ops and the close-out
row reads `sdd-implementer` at high; nothing else in the table
changes.

**A change the person asks for gets written here before it is acted
on.** If they say to move a role — for one window, for this project,
for good — edit the row, note it in the current spec's tier log with
the date and which spec it changed at, and commit, in the same turn,
*before* the next dispatch. Then do it. A model preference that lives
only in a session's context is gone at the next session boundary and
the next session will not know it ever existed, which is the one
failure this whole repo-as-interface arrangement exists to prevent.
If the person frames it as temporary ("while my allowance is low"),
write the row with the condition and the date in the comment, so
whoever reads it next knows when it stops applying and can ask. Never
infer the end of a temporary change and revert it unasked.

<!-- Why the two rows most likely to be questioned read as they do.

Task implementation sits at the implementation tier because it was
measured there: over two kaazap specs with the implementer at the top
tier's model at medium, cost per completed task matched within noise,
first-try rate matched at 6/6 and 9/9, and the top tier's separate
allowance drew about a fifth more per task. The honest limit is that
it was measured on bounded transcription against a fast automated
check, the case least likely to reward a stronger model, so a project
whose tasks are genuinely hard is the open question this row exists to
let someone answer. Ask it once, at project setup; don't re-open it
per spec.

Close-out sits at the top tier because it writes the ROADMAP.md and
DECISIONS.md entries, the acceptance evidence and the spec summary —
synthesis and prose, the same work the top tier earns its place on
everywhere else. Its cost depends mostly on its bundle, not its model.
On the same model, one close-out that read the documents and hunted its
own evidence cost $12.38 over 111 turns. Another, handed the evidence
with full reads forbidden, cost $0.99 over 24 turns. So the close-out
bundle carries per-criterion evidence, the walkthrough record, the tier
log, the spec summary and the ROADMAP entries, and the dispatch says not
to read spec.md, plan.md or tasks.md in full. -->

- **The session runs at the session tier, at medium effort**, set in
  this repo's `.claude/settings.json` — written at project setup from
  the profile's settings file in the skill's `project/.claude/` (`"model"`
  set to the session tier's full ID, `"effortLevel": "medium"`, and a
  level under `"modelSettings"` for each tier's full model ID). If that file is missing or lacks these
  keys, recreate it from the template and commit it before dispatching
  anything; nobody creates it by hand. Project settings outrank user
  settings, so a model picked in the app's picker only affects the
  session it was picked in — new sessions in this repo start here
  regardless. The app's effort indicator may show the model's default
  rather than the level in effect; `/effort status` inside the session
  is the authoritative check. The orchestrating session takes many
  bookkeeping turns and re-sends its whole context on each one — the
  dominant cost of the workflow — and it makes no design decisions: it
  assembles bundles, dispatches, verifies, commits, and reports. The
  role never needs the top tier. Under the Fable profile it sits
  on the top tier's model because, measured, Fable 5.1 at medium in
  this seat cost about a third per task of Opus 4.8 and its allowance
  held (the skill's design record has the numbers); under the Opus
  profile it sits on Opus 5.5, and the same discipline about turns
  applies.
  If it drops the protocol (a skipped review, a stale `tasks.md`
  edit, a task done by hand), the first fix is high effort, one line
  in the same file.
- **The session tier never resolves a design question.** When triage
  finds a task that isn't routine, the session frames the question in
  Plan Mode — so nothing is touched meanwhile — and dispatches the
  `skeptical-reviewer` at the top tier on a decision bundle: the task
  line, the plan section, the acceptance criteria, and the options as
  the session sees them. It transcribes the recommendation into
  `plan.md` and dispatches what remains. A product question `spec.md`
  doesn't settle goes to the person instead.
- **Everything the person reads is plain language.** Pause reports,
  spec-conformance summaries, and questions use short sentences and
  everyday words — no task IDs, agent names, tier names, or internal
  shorthand unless the person asks — and assume the reader won't open
  `plan.md`. Say what can now be tried, where execution deviated from
  the spec and why, and what needs a decision. Technical detail
  belongs in `plan.md` and the commit log, not in the report.
- **What the person's walkthrough finds is a finding, not a task
  line.** When the person reports at a phase pause that something is
  wrong, the session restates it — which acceptance criterion, what
  they saw, what the spec says — and dispatches a diagnosis bundle to
  the `sdd-implementer` (the report, the restatement, the task line,
  the plan section, the acceptance criterion, the files). The
  implementer finds the cause and fixes it if the fix is routine and
  inside the footprint; otherwise it returns the diagnosis and
  options, which go to a decision review at the top tier. A fix is
  logged as a sub-lettered task; a finding that is really the spec
  being ambiguous goes back to the person as a product question. The
  session never diagnoses in place.
- **The top tier runs only where the role table says it does**: by
  default the `sdd-planner` (one dispatch per spec), the
  `skeptical-reviewer` on plan/tasks sign-off and on decision reviews,
  and the close-out dispatch. The first three carry an explicit
  per-call override to the top tier's name; drop the override and the
  definition's own implementation tier applies, which is exactly what
  stepping one of those rows down means. The agent definitions carry
  `effort: high`, which overrides the session's medium, so reasoning
  stays at full strength where it matters.
- **Spec conversations happen in a Claude Code spec session of their
  own**, at the top tier, never inside an implementation session. The
  spec session also runs planning once `spec.md` is approved — the
  planner dispatch, the sign-off, the spec-conformance summary — and
  ends when `plan.md` and `tasks.md` are final, with a new session
  (not `/clear`, which keeps the model) whose opening prompt is the
  spec session's last message. A session in this repo opens at the
  session tier at medium, so a spec session
  states its model and effort first (`/effort status`) and asks the
  person to raise effort to high for this session (`/effort high`)
  before continuing. The next session opens at medium again from
  `.claude/settings.json`. (The project's first session is a spec
  session too: it scaffolds the repo from the skill's templates, then
  hosts the idea, the constitution, and the first spec.)
- **The `skeptical-reviewer` runs at the implementation tier by
  default** (its definition says `opus`) for per-phase reviews, the
  per-task reviews the planner marks, and the pre-merge sweep. Each
  review gets a single bundle file assembled with shell — diff, task
  lines, plan sections, acceptance criteria; for the sweep, the
  documents and the spec's full diff — and reads nothing else.
- **Review loop cap**: one review and at most one re-review per
  invocation — task, phase, sign-off, or sweep. The re-review sees the
  findings and the fix diff only. Blocking
  means it would fail an acceptance criterion or a test, or contradicts
  `plan.md` or `CLAUDE.md`; nothing else blocks. Anything open after
  the re-review goes to the tier log and the sweep.
- **Implementation runs in the subagent the role table names** — the
  task implementation row for ordinary tasks, the close-out row for
  close-out — one task per dispatch, sequentially. Neither is
  re-decided per spec: the table is the answer until the person
  changes a row. The orchestrating
  session triages each task, dispatches routine ones on a task bundle
  assembled with shell (task line, plan section, acceptance criteria,
  files, the pattern file to copy), and on return verifies with the
  verification command below — re-run by the orchestrator for tasks
  marked `review: per-task`, taken from the implementer's verbatim
  output otherwise — never by re-reading the diff. Only the
  orchestrator edits `tasks.md` or commits, and the orchestrator never
  implements second-look notes or does device or browser checks by
  hand.
- **One implementation session per spec.** It opens when `plan.md` and
  `tasks.md` are final and ends at the merge; a phase pause is a pause
  in it, not a boundary — the person attests and says continue.
  `/compact` if the context grows large; never clear or compact
  mid-task. `/clear` is not part of the workflow: both session
  boundaries are new sessions.
- **Two pauses end with a continuation prompt, and only two**: plan
  and tasks final, and the merge with the next spec waiting on
  `ROADMAP.md`. At those the report's last item is the exact prompt to
  paste into the next session, in its own fenced block. It names the
  spec directory, the files to read, where to resume, the involvement
  level, the pause cadence, and any effort switch the next session
  needs. Write anything the next session needs to a file first; the
  prompt points at files. If nothing can proceed until the person
  decides something, say so instead.
- **A phase pause never ends with a continuation prompt.** The phase
  report ends with what to check in the app and how to say continue —
  nothing else. The session cannot know whether the person is about to
  stop, so a rule conditioned on that produces a prompt at every phase,
  which is what this line exists to prevent: the spec runs in one
  implementation session, and a prompt offered unasked invites a
  `/clear` that costs a re-read and buys nothing. If the person says
  they are stopping, or asks for a prompt, write one then, as the next
  message — the resume form from the first unchecked task. Asked for,
  it costs one turn; volunteered, it costs the session.
- **Batch the bookkeeping**: commit, checkbox, and tier-log row in one
  shell command; bundle assembly and dispatch back to back. Every turn
  saved is one fewer re-send of the whole context.
- **Fallback** (Fable profile): if the top tier's usage budget runs out, dispatch the
  planner and sign-off at the implementation tier for the rest of the
  window (drop the override; both definitions default to `opus`), and
  switch the session itself to `claude-opus-5-5` mid-session
  (`/model claude-opus-5-5` — one cache re-write, then continue), and
  dispatch `sdd-implementer` for any row that names
  `sdd-implementer-fable`, including close-out. This is the whole role
  table stepped down at once, and it is the automatic form: it fires on
  the allowance, for the rest of the window, and the rows are not
  edited. A step-down the person *asks* for is the other form — it
  edits the rows and persists until they say otherwise. Don't confuse
  them, and don't silently revert one the person asked for. Either way
  the tier log records what ran and when the switch happened.
- **Escape hatch**: two failed verifications on one task, or a "stopped
  on a judgment call" the orchestrator considers well-specified, and
  the orchestrator does that task itself, noting the
  miss in `tasks.md`.
- **Lighter implementer**: off. <!-- Turn on per project once the
  first spec's tier log justifies it: "the session tier for tasks with
  an automated Verify check, a named pattern file, and a small
  footprint." -->
- **Log token usage per implementer run and per reviewer invocation**,
  plus tier misses, in `tasks.md`'s tier log for the first spec under
  this policy, and compare against a previous spec before treating the
  policy as settled.

## Spec-driven workflow

This project follows spec → plan → tasks → implement, gated by review
between each phase — the person's or the `skeptical-reviewer`'s, per
the involvement level above. Artifacts live in `specs/<NNN>-<slug>/`:

- `spec.md` — what and why, user-facing behavior, acceptance criteria,
  explicit non-goals. No implementation detail.
- `plan.md` — technical design: types, data flow, what changes where.
- `tasks.md` — ordered, small, independently verifiable tasks.

Authorship: `spec.md` is written with the person in a spec session.
`plan.md` and `tasks.md` are drafted by the `sdd-planner` subagent —
at the top tier, from a planning bundle, against the actual codebase
(or, for the first spec, against the constitution, the spec, and the
skill's templates) — and the orchestrator commits them to the spec
branch with the PR still in draft. Both are signed off before any
implementation task starts: at the product-owner level by the
`skeptical-reviewer` (blocking findings fixed and re-reviewed), with
the person receiving a spec-conformance summary to approve; at the
technical-lead level by the person directly.

Do not begin implementation on a feature without an approved spec and
plan in that feature's directory. When resuming a session, check
`specs/<feature>/tasks.md` for current state before doing anything else.

## Collaboration workflow

If the `spec-driven-development` skill is installed
(`~/.claude/skills/spec-driven-development/` or a project-level
`.claude/skills/`), its collaboration workflow applies automatically —
routine tasks proceed normally, real decisions resolve via Plan Mode and
the `skeptical-reviewer` subagent, and beyond the pauses their
involvement level defines, the person is looped in only when something
in the design turns out infeasible or needs real rework, or a
previously-unknown consideration surfaces that would materially change
the project's direction. Nothing needs to be repeated here.

<!-- Add project-specific refinements only if this project's escalation
criteria genuinely differ from the skill's default — a different risk
tolerance, a different definition of "foundational" for this particular
codebase. Most projects won't need anything here at all; the one case
this project came from needed a hand-written version only because the
skill didn't exist yet at the time. -->

## Verification

The verification command for this project is:

    [verification command]

<!-- One exact command, with its output filter, that builds, runs the
tests, and prints a short summary — `xcodebuild ... -quiet 2>&1 | tail
-n 40`, `npm test 2>&1 | tail -n 40`, or a script in the repo. Raw
build logs are the largest single thing an agent can put in its
context, and in the first measured spec they were inside every
implementer, reviewer re-run, and orchestrator check. Every one of
those uses this command and nothing more verbose. -->

After any implementation task, Claude Code must run that command and
report its actual output, not a paraphrase.

A task is not complete until that output is green. Do not weaken, skip,
or delete a test to make it pass — if a test seems wrong, flag it and
ask. When the task was dispatched to the `sdd-implementer`, its verbatim
output is the verification; for a task marked `review: per-task` the
orchestrator re-runs the command itself before committing.

## Git conventions

- **One branch per spec, not per task or phase.**
- **A spec's implementation never goes to `main` directly.** Its code,
  its files under `specs/<NNN>-<slug>/`, and any README change
  describing its behavior live on that spec's branch and reach `main`
  through its PR.
- **Work that isn't a spec's implementation commits straight to
  `main`, and doesn't need to ask.** Roadmap grooming, a
  `DECISIONS.md` entry, a constitution amendment, a docs or README
  correction unrelated to a spec in flight, a design brief, the
  written residue of a conversation that didn't become a spec,
  tooling or config no spec touches. None of this has a spec branch,
  and none of it deserves one — a branch and a PR for a roadmap
  paragraph cost more than they protect, and stopping to ask costs a
  turn and interrupts the conversation that produced the change. The
  test is whether the change implements part of some spec's
  `tasks.md`, not whether the file appears on a list. If it doesn't:
  commit it to `main`, push, and say so in the report. This line is
  the permission; don't ask for it again.
- **Branch anyway when the change wants a diff someone will look at** —
  a dependency bump, a refactor with no spec behind it, anything where
  being wrong is expensive or awkward to unwind. Use
  `fix/<short-description>` or `chore/<short-description>`, not the
  spec `<NNN>-<slug>` convention, and open a PR. Size and risk decide
  this, not whether the work counts as "a spec."
- Open the PR as a draft immediately after pushing the branch, for a
  running diff. Only mark it ready and merge once every task in the
  spec's `tasks.md` is complete and verified.
- Marking ready has a close-out step, before any pre-merge review
  sweep: update `ROADMAP.md` (drop or annotate what this spec shipped,
  add follow-ups it surfaced) and the README if user-facing behavior
  or setup changed. README changes describing this spec's behavior
  ride the spec branch.
- Keep AI co-authorship attribution on commits — accurate, and worth
  keeping for a project meant to demonstrate this workflow.
- Never force-push.

## Commits

- One commit per completed task where practical, referencing the task ID
  — made by the orchestrating session after its own verification, never
  by the implementer subagent.
- Commit messages describe what changed and why, not "implement task 3".
