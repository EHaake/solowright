# Project Constitution

This file is the standing contract for how this codebase is built. It loads
into every Claude Code session automatically. Specs and plans must not
contradict it; if a spec needs to, the constitution gets amended first,
explicitly, in its own commit.

## What this project is

<!-- One paragraph. What it is, who it's for, what the core loop is.
Write this once the first spec conversation has actually happened —
don't guess at it before then. -->

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
Implementation pauses after each phase unless the person says to run
further, and whenever something unexpected bears on spec adherence.

**Technical lead.** As above, but the person also reads and approves
`plan.md` and `tasks.md`, and implementation pauses for their review
after every task the planner marked `review: per-task`.

## Model policy

<!-- Decided once, alongside the involvement level: pick one profile
and delete the other. Adjust the tier names as models change; the
roles don't. A project can move between profiles later — change the
names here and swap the settings file, one commit — and the tier log
shows from which spec. -->

**Standard profile** (the default; measured on the projects the skill
came from). Top tier `fable`; implementation tier `opus`; session
tier `fable` at medium effort (the top and session tiers are the same model
at different effort; the fallback session model is `claude-opus-4-8`,
the full ID, since a previous-generation model has no short alias).
Settings from the skill's `project/.claude/settings.json`.

**Economy profile** (a small or personal project, or one that should
leave the top tier's separate allowance to other projects). One model
family throughout: top tier `opus`; implementation tier `opus`;
session tier `claude-opus-4-8` at medium effort. Nothing runs on
Fable: the planner and sign-off dispatches carry no override, and the
top-tier fallback below never applies. Settings from the skill's
`project/.claude/settings.economy.json`. Move to the standard profile
when a spec's plan is the kind a stronger planner would change — a
sign-off that keeps finding blocking problems is the signal.

These names are the only place a model is spelled out; everything
below refers to the roles.

**Task implementer**: `sdd-implementer` (the implementation tier,
`opus` at high). <!-- Asked once, at project setup, and answered here.
The alternative is `sdd-implementer-fable` (the top tier's model at
medium effort) — same body, different frontmatter. Measured over two
kaazap specs: cost per completed task was the same within noise, first-
try rate was the same at 6/6 and 9/9, and the Fable implementer drew
about a fifth more of the top tier's separate allowance. So the default
is the implementation tier, and the top tier's model is the choice for
a project whose tasks are genuinely hard rather than bounded
transcription against a fast automated check. That case is untested —
the measurement ran on the simplest of three projects, which is the
case least likely to reward a stronger model. Switching later is one
word in this line and a tier-log row saying from which spec; no
re-scaffolding, both agent definitions stay installed. Under the
economy profile this line always reads `sdd-implementer`. -->

**Close-out dispatch**: `sdd-implementer-fable` under the standard
profile, `sdd-implementer` under the economy profile — whatever the
line above says for ordinary tasks. The close-out task writes prose:
the `ROADMAP.md` and `DECISIONS.md` entries, the acceptance evidence,
the spec's own summary. That is synthesis, not transcription, and it
is the one implementer dispatch the top tier's model is worth paying
for. The measured close-out dispatches cost $5.66 and $3.74 on the
implementation tier against $0.81 and $1.31 on the top tier's model at
medium, though that gap is confounded by bundle shape and is a reason
to watch the tier log, not a settled number.

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
  role never needs the top tier. Under the standard profile it sits
  on the top tier's model because, measured, Fable 5.1 at medium in
  this seat cost about a third per task of Opus 4.8 and its allowance
  held (the skill's design record has the numbers); under the economy
  profile it sits on Opus 4.8, the model whose reports read most
  clearly to the person, and the same discipline about turns applies.
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
- **The top tier runs only inside the decisions**: the `sdd-planner`
  (one dispatch per spec) and the `skeptical-reviewer` on plan/tasks
  sign-off and on decision reviews — each dispatched with an explicit
  per-call override to the top tier's name. The three agent
  definitions carry `effort: high`, which overrides the session's
  medium, so reasoning stays at full strength where it matters.
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
- **Implementation runs in the subagent the "Task implementer" line
  above names**, and the close-out task in the one the "Close-out
  dispatch" line names, one task per dispatch, sequentially. Neither
  is re-decided per spec: the lines are the answer until the person
  changes them. The orchestrating
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
- **Fallback** (standard profile): if the top tier's usage budget runs out, dispatch the
  planner and sign-off at the implementation tier for the rest of the
  window (drop the override; both definitions default to `opus`), and
  switch the session itself to `claude-opus-4-8` mid-session
  (`/model claude-opus-4-8` — one cache re-write, then continue), and
  dispatch `sdd-implementer` for any task whose line above names
  `sdd-implementer-fable`, including the close-out. Nothing else
  changes; the tier log records what ran and when the switch happened.
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
- **Never commit implementation work directly to `main`.** All code
  changes happen on a spec branch. Repo-wide docs (`CLAUDE.md`,
  `ROADMAP.md`, `DECISIONS.md`) are the exception: they commit straight
  to `main`, while spec-specific files ride the spec branch.
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
