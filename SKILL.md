---
name: spec-driven-development
description: Solowright — a spec-driven development (SDD) system for a solo builder — run on a software project using Claude, Claude Code, and (if the project has a UI) Claude Design together — writing a constitution and specs before code exists, translating design references into implementation, and running a disciplined build-and-review loop. Use this whenever starting a new app, website, or software project from scratch with Claude Code as the implementer; when the user mentions "Solowright," "spec-driven development," "SDD," writing a CLAUDE.md/constitution, or wants a structured spec → plan → tasks → implement workflow; or when picking up an existing SDD project and needing to know how the pieces fit together. Also use when the user is deciding how to divide work between a chat-based planning conversation, Claude Design, and Claude Code, or asking how to review AI-written code without becoming a bottleneck.
---

# Solowright: Spec-Driven Development with Claude

A spec-driven development system for a solo builder, with Claude as the
planning partner and Claude Code as the implementer. This file is the
process;
the reasoning behind its less obvious choices, and the history of how
they were reached, is in `references/design-record.md`.

## The core idea

The session where a spec gets written and the session where it gets
built don't share memory. **The repo is the only
real interface between them.** Every decision that needs to survive past
one conversation has to end up in a file, or it's gone the moment either
session ends. This one fact drives almost everything else in this skill:
which documents exist, why `plan.md` matters more over time than `spec.md`
does, why a session can end and a fresh one pick up without losing
anything real, and why a project that documents its reasoning well is *easier* to
resume cold than one that doesn't.

## The three-tool division of labor

- **Claude Code** hosts the whole flow, a project's first day included.
  The first session scaffolds the repo from the skill's templates and
  then hosts the idea, constitution, and first-spec conversations (see
  "Starting a project"); every later spec is a conversation in a spec
  session of its own (see "Spec conversations" under "Model tiering").
  Plans and task lists are drafted by the `sdd-planner` subagent, one
  dispatch per spec (see "Who authors plan.md and tasks.md").
- **Claude Design** (if the project has a UI) produces visual references
  — screens, a token system, a written brief — not literal source code
  for a native app. Its output is HTML/CSS underneath. For a web app that
  might be directly usable; for anything else (iOS, desktop, etc.) treat
  it as what a designer's mockups would be for a native team: the target
  to translate toward, not something to import.
- **Claude Code, at implementation time,** builds against the documents
  as ground truth — reading `CLAUDE.md` at the start of every session
  and treating `spec.md`/`plan.md`/`tasks.md` as the source of truth
  for what's being built, not any conversation. It orchestrates rather
  than types: each routine task goes to the `sdd-implementer` subagent at
  the implementation tier, and the session triages, verifies, and
  commits (see "Model tiering").
- **Claude (chat) is optional.** Some people prefer to think an idea
  through in chat before there is a repo. That's fine: have the idea
  conversation there (the skill can be installed in claude.ai too) and
  bring its conclusions to the project's first session, which writes
  them into the scaffold. The documents are the interface either way;
  nothing in the workflow depends on chat.

## Involvement level: decide it once, at the start

The person's role in a project is a choice made in the constitution
conversation and written into `CLAUDE.md`, not something re-derived at
every pause. Two levels:

- **Product owner** (the default). The person owns `spec.md` — that's
  where most of their involvement lives — attests to behavior by using
  the app at phase pauses, and decides the two escalation triggers.
  They never approve technical work: `plan.md` and `tasks.md` are
  drafted by the sdd-planner and signed off by the skeptical-reviewer,
  and each phase (and any task the planner marked for its own review)
  is reviewed by the skeptical-reviewer rather than by the person. What
  reaches them is a spec-conformance summary, not an architecture
  review.
- **Technical lead**. The person also reads and approves `plan.md` and
  `tasks.md`, and implementation pauses for their review after every
  task the planner marked `review: per-task`.

**Pause cadence is a separate question, asked at the same time.**
Involvement level says who approves technical work; cadence says how
often the build stops. Three values, in the constitution next to the
involvement level: **when there's something to try** (the default),
**every phase**, and **only when blocked**. Under the default the
planner marks each phase header `walkthrough: <what to try>` or
`walkthrough: none — <why>`, and the orchestrator pauses after the
first kind and runs straight through the second.

The marking belongs to the planner because it can see the whole spec
at once, and because the orchestrator mid-phase cannot tell a phase
with nothing to show from one it merely hasn't described well. The
test is observability: does this phase change something the person
could see by using the app? A type, a shared helper, a test harness,
an internal refactor — no. A screen, a flow, different behavior — yes.

Two things keep the hands-off settings honest. A `none` phase still
gets its skeptical-reviewer phase review; what's skipped is the
person's attestation, not the check. And every unpaused phase appends
what it would have asked the person to try to a walkthrough list in
`tasks.md`, which is what they walk at the close-out — so the check is
deferred, never dropped, and the tier log says which phases ran
unpaused, so a defect found late can be traced to where it came from.
The real cost of a hands-off cadence is exactly that: a problem
introduced in phase one and found at the merge is more expensive to
unwind than one caught in phase one. That is the trade the setting
exists to let someone make deliberately.

Blockers pause under every cadence, including "only when blocked":
either escalation trigger, a product question `spec.md` doesn't
settle, the escape hatch firing twice on one task, and any finding
that can't be resolved without changing what the spec promised.

Everything below that mentions a review, an approval, or a pause is
written for the product-owner level at the default cadence unless it
says otherwise; the
technical-lead variant is the same flow with the person added back at
those gates. Whatever reaches the person — a pause report, a
spec-conformance summary, a question — is written in plain language:
short sentences, everyday words, no task IDs, agent names, or tier
names, for a reader who won't open `plan.md`. The report exists so
they can act, not so the work is documented. Product owner is the default because a per-task pause a
non-technical person clicks through looks like review without being
one — worse than no gate.

## The document set

- **`CLAUDE.md`** — the constitution. Lives at the repo root, gets read
  automatically at the start of every Claude Code session. Platform
  choices, architecture rules, testing requirements, git conventions —
  and *why*, not just the rule. This file accumulates the project's
  hard-won engineering lessons as they're discovered (see "Principles
  worth generalizing" below) and becomes more valuable over time, not
  less.
- **`specs/<NNN>-<slug>/spec.md`** — what and why. User-facing behavior,
  acceptance criteria, explicit non-goals. No implementation detail —
  that discipline matters, because a spec full of implementation detail
  stops being a place to argue about *behavior* and starts constraining
  a plan that hasn't been written yet.
- **`specs/<NNN>-<slug>/plan.md`** — technical design. This is the
  document that ends up mattering most, months in: not just what got
  built, but *why*, including the decisions that got reversed and the
  bugs that changed the design. Treat it as a living record, not a
  one-time handoff artifact — update it whenever a real decision gets
  made or corrected, not just at the start of a spec.
- **`specs/<NNN>-<slug>/tasks.md`** — ordered, small, independently
  verifiable execution steps. See "The multi-writer file problem" below
  — this file needs different handling than the others the moment
  implementation starts.
- **`DECISIONS.md`** (repo root) — business, product, and process context
  that doesn't fit the structured docs above: naming rationale, legal or
  business decisions, tooling choices. The dumping ground that keeps
  `plan.md` from accumulating things that aren't actually technical
  design.
- **`ROADMAP.md`** (repo root) — the backlog of future specs. Deliberately
  *not* ordered or scheduled — priorities should be set once there's a
  working app to actually use, not guessed at from a backlog.
- **`design/brief.md`** (if the project has a UI) — visual and
  interaction direction for whatever design process is being used,
  referencing `spec.md` for functional detail rather than restating it.
  See "Design exploration" below for what actually needs to be in it.

## Design exploration, when the project has a UI

A written design brief precedes any actual screen design —
`project/design/brief.md` is the starting shape, deliberately empty of
any specific project's actual answers (see the note at the top of that
file for why copying a previous project's palette or signature element
forward defeats the point). Four principles are worth stating here
directly — they are the difference between a distinctive visual
identity and a generic one:

1. **Draw from the audience's own existing visual vocabulary, not a
   generic aesthetic.** Before reaching for a trendy look, ask whether
   this specific audience already sees, uses, or handles something in
   their own world that could inform the visual language authentically.
   A gear-hobbyist audience, for instance, already has a real vocabulary
   — aperture rings, VU meters, brass hardware — that's authentic to
   them rather than invented from a mood board. Ask the equivalent
   question for whoever the actual audience is; not every audience has
   an obvious answer, but it's worth genuinely checking first.

2. **Name and rule out the current AI-generated design defaults,
   explicitly, by name.** The single highest-leverage line in a design
   brief — and one with a short shelf life, since what's over-used in
   AI-generated design shifts over time. Look at what's actually common
   right now when writing a new brief, rather than reusing a previous
   project's list; the durable part is the habit, not any specific set
   of clichés.

3. **Look for one signature element — recurring, functional,
   distinctive — and don't force one where none exists.** The highest-
   value single decision in a visual identity, when a real candidate
   exists: something that shows up in multiple places, does real work
   rather than existing purely as decoration, and would be recognizable
   as belonging to this product from a screenshot of the element alone.
   Forcing one where the product has no natural candidate produces
   arbitrary decoration instead.

4. **If aiming for "flat but characterful" rather than purely generic-
   flat, state the skeuomorphism boundary explicitly.** Physical or
   tactile objects can supply shape and metaphor language without
   crossing into photorealistic rendering — genuinely hard to hit
   consistently unless the brief draws that line on purpose.

## The first-spec exception

A brand-new project's first spec is usually bigger and more coupled
than every spec after it — the data model, the core screens, and the
core flows are all interdependent, so splitting them into separate
specs before any of them work yet adds coordination overhead with zero
payoff. This is a deliberate, one-time exception, not a failure to
scope well. The moment that first spec ships, the condition that
justified it stops holding — there's now an established codebase, and
every spec after it returns to the normal rule: one feature, one spec,
sized to be reviewable on its own.

## Git conventions

- One branch per **spec**, never per task or phase — and only for a
  spec's implementation. Work that isn't that commits straight to
  `main`; read the category rule below before taking this line as a
  general ban.
- Open the PR as a **draft** immediately after pushing the branch — it
  gives a running diff to review commit-by-commit, separate from
  whatever Claude Code's own summaries say.
- Only mark it ready and merge once *every* task in that spec's
  `tasks.md` is done and verified — not when it "looks done." Merging
  partway through, even with good intentions, defeats the point of
  scoping a spec as one coherent unit.
- Marking ready also has a close-out step: update `ROADMAP.md` (drop
  or annotate what the spec shipped, add follow-ups it surfaced) and
  the repo README (if user-facing behavior or setup changed), then run
  the pre-merge review sweep so it verifies those updates rather than
  pre-dates them. These two files go stale precisely because nothing
  else forces them — no task references them, no test fails when they
  lag — so the check lives here at the merge gate, and as a standing
  final task in `tasks.md` (see the tasks template). README changes
  describing the spec's behavior ride the spec branch. `ROADMAP.md`
  and `DECISIONS.md` are the awkward case: the category rule below
  sends them straight to `main`, but the close-out task that writes
  them is a task in this spec's `tasks.md`. Resolve it the way the
  measured projects do — the close-out drafts the roadmap and
  decisions text into a file on the spec branch, and that text is
  applied to `main` after the merge. The draft is spec work and rides
  the branch; the edit to the living document is not, and doesn't.
- Spec-specific files commit to that spec's branch and ride into
  `main` when the spec merges. **Everything that isn't a spec's
  implementation commits straight to `main`, without asking** — a
  roadmap edit, a decisions entry, a constitution amendment, a docs
  fix, a design brief, whatever a conversation that didn't become a
  spec left behind. Write the rule as that category and not as a list
  of blessed filenames: the older wording named three repo-wide files
  as "the exception," and sessions grooming the roadmap read the
  headline ban, failed to match their change to the list, and stopped
  to ask permission — a gate on the cheapest, most reversible work in
  the repo. The test is whether the change implements part of some
  spec's `tasks.md`. A change that wants a reviewable diff anyway — a
  dependency bump, an unplanned refactor — gets a `fix/` or `chore/`
  branch and a PR, decided on size and risk rather than on whether it
  counts as a spec.
- Keep AI co-authorship attribution on commits. It's accurate, and for a
  project meant to demonstrate this workflow, the transparency is worth
  more than a clean-looking log.
- Never force-push.

## Review cadence: tiered by risk, not uniform

Reviewing every single task at the same depth is both exhausting and
miscalibrated — a wrong CRUD field is a five-minute fix; a wrong data
model decision discovered three weeks later is not. Tier the review
cadence by how expensive a mistake would be to unwind:

- **Review after every phase**, with a phase bundle — the default
  everywhere, foundational phases included. A foundational phase is
  short by construction, so its review comes a day later rather than
  the same afternoon, and the pre-merge sweep is still behind it.
- **Per-task review only where the planner marks it** — a task whose
  mistake would be genuinely expensive to unwind (a data-model contract
  a dozen later files will depend on), flagged in `tasks.md` with
  `review: per-task`. The exception, not a phase-wide rule.
- Re-tighten around anything that turns out to be a genuine judgment
  call, even mid-phase, rather than treating the cadence as fixed once
  set.

At the product-owner level every one of these reviews is a
skeptical-reviewer pass, and none of them pauses for the person. The
person's own pauses follow a different rule: after each phase (unless
they've said to run further), and whenever something unexpected
surfaces that bears on spec adherence. At the technical-lead level the
per-task reviews the planner marks are the person's as well. Either
way, reviews are scoped to a bundle — the diff, the plan sections it
implements, the acceptance criteria it serves — never a fresh
whole-codebase read; see "Keeping reviews cheap" in
`references/collaboration-workflow.md`.

## Model tiering: three tiers, and a row per dispatch

Three tiers, named once in the constitution's model policy, and a
nine-row table there that says which tier each dispatch runs at.
Under the standard profile the top and session tiers are the same
model at different effort — decided by experiment 1, see
`references/design-record.md`; under economy they are two different
models, which matters wherever this section says the two coincide:

- **The top tier decides, and writes the spec's prose.** Plan and
  task drafting (the `sdd-planner`, one dispatch per spec), the
  skeptical-reviewer on sign-off and on decision reviews, and the
  close-out dispatch — plus, under the standard profile, the spec
  conversation, which runs on the session tier's model and is the same
  one. Nothing else. It reaches the agents only through explicit per-call
  overrides on those dispatches.
- **The implementation tier builds and checks.** The implementer, one
  task per dispatch — the edit, build, test loop that accounts for
  most of a spec's tokens — and the skeptical-reviewer's per-phase and
  marked per-task checks. Both definitions pin their model by name, so
  neither follows the session's. A second implementer definition,
  `agents/sdd-implementer-fable.md`, is the same body at the top
  tier's model and medium effort; a project names one or the other in
  its constitution, and the default is the implementation tier (see
  "Two rows worth explaining" below for the measurement behind that
  default). The close-out dispatch goes to the top tier's model under
  the standard profile either way, and to `sdd-implementer` under
  economy, where nothing runs on a separate top-tier model.
- **The session tier orchestrates**, at medium effort. The
  orchestrating session takes many bookkeeping turns and re-sends its
  whole context on each, which makes it the dominant cost of the
  workflow — and it makes no design decisions, so nothing about the
  role needs the top tier. What decides which model sits there is the
  price of a cache read, since re-sends are almost all of the seat's
  tokens, and the prose of the pause report, the one output a person
  reads. The seat runs Fable 5.1 at medium: measured against Opus 4.8
  in the same seat, it cost about a third per task and drew on Fable's
  allowance at a rate three concurrent projects could sustain (the
  design record has the numbers).

The planner and reviewer definitions carry `effort: high`, so
reasoning stays full-strength inside them regardless of the session's
setting. `sdd-implementer-fable` carries `effort: medium`, which is
what the close-out row and the opt-in task row resolve to.

The split is by *role*, decided per task at execution time — not a
per-task model table written in advance. Three things catch a lighter
model quietly doing a worse job on a task that looked mechanical: the
orchestrator reads every task before dispatch and every report after,
the implementer is under a standing rule to stop and return the moment
it hits a judgment call, and the reviewer sits at every phase boundary.

How the loop runs, per task, in the orchestrating session:

1. **Triage** (Step 1 of the collaboration workflow). Routine →
   dispatch. Not routine → the session frames the question in Plan
   Mode (so nothing is touched meanwhile) and dispatches the reviewer
   at the top tier on a decision bundle — task line, plan section,
   acceptance criteria, the options it can see — then transcribes the
   recommendation into `plan.md` and dispatches what remains. The
   session tier never resolves a design question itself.
2. **Dispatch with a packet, not a pointer.** The subagent starts cold
   — it sees `CLAUDE.md`, its own definition, and the prompt. Name the
   task line, the `plan.md` section it implements, the `spec.md`
   acceptance criteria it serves, the files involved, and the existing
   file whose pattern to copy. Findings from earlier tasks that aren't
   yet written down go in the packet too — or better, get written down
   first.
3. **Verify by running the verification command, not by reading.**
   The constitution names one filtered build-and-test command; the
   implementer runs it and reports its output verbatim. For a task the
   planner marked `review: per-task`, the orchestrator re-runs it, then
   assembles a review bundle with shell (diff, task line, plan section,
   acceptance criteria) and invokes the reviewer on that alone; for
   every other task the implementer's output is the verification and
   the phase review is the check. Open the diff yourself only when
   something failed — an orchestrator that reads every diff in full
   has paid for the work twice.
4. **One review, at most one re-review, per invocation.** The
   re-review sees the findings and the fix diff, nothing more, and
   whatever is still open after it goes to the tier log and the
   pre-merge sweep. Blocking is defined narrowly — would fail an
   acceptance criterion or a test, or contradicts the plan or
   constitution — and nothing else blocks.
5. **Commit, check the box, record findings — and nothing else by
   hand.** The orchestrator is the only writer of `tasks.md` and the
   only one who commits; a commit means orchestrator-verified. Findings
   from the report go into `plan.md` or `tasks.md` now, not later,
   since the next implementer won't have seen them otherwise. The
   orchestrator does not implement the reviewer's second-look notes
   itself, and does not do device, browser, or visual verification by
   hand: second-look items go to the log or the next task's bundle, and
   visual checks are the implementer's Verify criterion or the person's
   attestation at the phase pause. Every piece of work the
   orchestrator "folds in" itself is work the tiering exists to move
   off it.
6. **Sequential, one task at a time, in one session for the whole
   spec.** Commit-per-task and shared files make parallel implementers
   messy; parallel dispatch is a deliberate opt-in for a later day, not
   the default. A phase pause is a pause, not a session boundary: the
   report goes to the person, they attest by using the app, and the
   same session continues when they say so. Compact if the context
   grows large; the session ends at the merge (see "Session and
   context hygiene").

**The two session boundaries end with a continuation prompt. Nothing
else does.** A spec has exactly two: `plan.md` and `tasks.md` final,
handed to implementation; and the spec merged, with the next one
waiting on `ROADMAP.md`. At each, the report's last item is the exact
prompt to paste into the next session, in its own fenced block so it
copies in one click. It is self-contained: the spec directory, the
files to read, where to resume (the first unchecked task, or the
phase), the involvement level, the pause cadence, and any effort
switch the next session needs (a spec session opens at medium and
must be raised to high). Anything decided at the pause that
the next session needs is written to a file first; the prompt points
at files, it doesn't carry state. If nothing follows — the person has
to decide something before work can continue — say so instead of
inventing a next step.

**A phase pause gets no prompt.** The phase report ends with what the
person should check in the app and how to say continue. Write the rule
that way, as a flat default, and not as "unless they're stopping":
the session is writing the report before the person has said anything,
so a rule conditioned on their intent resolves to "include one, just
in case" every time. That is the failure this rule is written against
— a spec is one implementation session, and a continuation prompt
sitting at the end of every phase report is a standing invitation to
`/clear`, which costs a full re-read and buys nothing a pause didn't
already give. If the person says they're stopping, or just asks for a
prompt, write it then, in the next message, resuming from the first
unchecked task. On request it costs one turn. Volunteered eight times
a spec, it costs the session it was supposed to protect.

**The escape hatch.** If the implementer fails verification twice on
the same task, or returns "stopped on a judgment call" for something
the orchestrator considers well-specified, the orchestrator does that
task itself and notes the miss in `tasks.md`. A tier assignment is a
guess to verify, and the misses are the data.

**When the person's walkthrough finds something wrong.** At a phase
pause the person attests by using the app, and what they report back
is a finding against an acceptance criterion, in user terms. It is
not a task line, so it gets its own path, and the session's part of
it is procedure:

1. **Restate it, don't diagnose it.** Which acceptance criterion or
   task it touches, what the person saw, what the spec says should
   happen — and confirm the restatement with the person if it isn't
   obvious. Diagnosis is investigation, and investigation isn't done
   in the session.
2. **Dispatch a diagnosis bundle** to the implementer: the report in
   the person's words, the restatement, the task line, the plan
   section, the acceptance criterion, and the files that task touched.
   The implementer finds the cause; if the fix is routine and inside
   the footprint, it makes it and verifies; otherwise it returns the
   diagnosis and the options it can see, without picking one.
3. **Route the return.** A fix → verify, commit, and log it as a
   sub-lettered task (`T014a`), so the tier log shows what the
   walkthrough caught. Options → a decision review at the top tier,
   then transcribe and dispatch. A finding that turns out to be the
   spec being ambiguous, or the person wanting different behavior →
   a product question, back to the person; it becomes a spec
   amendment before any code changes.

The pause report that follows says, in plain language, what was
reported, what was found, and what changed — or what still needs the
person's decision.

**A lighter implementer is available but off by default.** A project
can add a fourth tier name and point the task-implementation row at
it — a lighter model such as Sonnet — for tasks that meet all three
of: an existing automated check as the Verify criterion (a
manual-check task never drops tiers, because the orchestrator can't
cheaply verify it), a named file in the codebase whose pattern it
copies, and a small footprint. Because the role table is per-role and
not per-task, turning this on means the row applies to every ordinary
task, so it belongs to a project whose tasks are uniformly that shape.
Leave it off until a project's first spec under this policy shows
implementation-tier dispatch working. Per-call improvisation is not
the mechanism: a model change is a committed row edit, so that the
next session knows what this one decided.

**Measure it.** The subagent's return reports its token usage; log it
per invocation in `tasks.md`'s tier log with the resolved model name,
alongside any escape-hatch misses, and compare the spec's total (from
`ccusage session --breakdown` afterward — the orchestrator can't see
its own usage) against a previous spec of similar size. The policy
earns its keep only while the coordination overhead stays smaller than
what it replaces. If a spec measured under it loses to the
single-session regime on the top tier's budget, the lever is the role
table: step the expensive rows down one at a time and re-measure,
rather than abandoning the dispatch loop wholesale.

**Three names, one place, two profiles.** The constitution's model
policy names the top, implementation, and session tiers once;
everything else refers to the roles. Two named profiles fill the
names. **Standard** (the measured default): top `fable`,
implementation `opus`, session `claude-fable-5-1` at medium.
**Economy**, for a small or personal project, or one that should leave
the top tier's separate allowance to other projects: one model family
throughout — top and implementation `opus`, session `claude-opus-4-8`
at medium — so the planner and sign-off run at their definitions'
default with no override and nothing draws on Fable. The choice is
made in the constitution conversation and can change later: edit the
names, swap the settings file, one commit; the tier log shows from
which spec. The session's model and effort live in the project's
`.claude/settings.json`, written at setup from the profile's template
(`project/.claude/settings.json` or
`project/.claude/settings.economy.json`) and recreated by the
orchestrator if missing — project settings outrank the app's picker
for new sessions, so they hold without anyone remembering. Two
fallbacks, both under the standard profile: when the top
tier's budget is exhausted, drop the override on the planner and
sign-off dispatches for the rest of the window (both definitions
default to the implementation tier), switch the session itself to the
implementation tier's previous generation (`/model claude-opus-4-8`,
mid-session — one cache re-write, then business as usual) and log
both in the tier log, since the session shares that budget; and if the session drops the protocol — a skipped
review, a stale `tasks.md` edit, a task done by hand — raise its
effort to high, one line in the settings file, before changing its
model.

**Under the two profiles, a role table.** The profile fills the three
tier names; the table says which tier each role runs at, one row per
dispatch — spec conversation, planner, sign-off, decision review, task
implementation, close-out, per-task and phase review, pre-merge sweep,
and the session's own bookkeeping. Its cells hold tier names rather
than model IDs, so switching profile re-points every row at once and
the three names stay the only place a model is written down. It lives
in the constitution's model policy, so every session reads it before
it dispatches anything.

The table exists because the profiles are all-or-nothing and most real
requests aren't. "Keep the spec conversation on the top tier and move
everything else down" is a reasonable thing to want when an allowance
is running low, and before the table there was nowhere to record it:
a session would do it, and the next session would never know it had
been asked. Now it is a row edit. Stepping a role down means replacing
"top tier (override)" with "implementation tier (no override)", which
is literally what the dispatch does differently.

**Two rows worth explaining.** *Task implementation* defaults to the
implementation tier because it was measured there: two kaazap specs
ran the implementer at the top tier's model at medium against the Opus
baseline, and cost per completed task matched within noise, first-try
rate matched at 6/6 and 9/9, and the top tier's separate allowance
drew about a fifth more per task. The stronger model earns its place
where judgment is the work, and bounded transcription against a
verification command is not that place. The honest limit: it was
measured on the simplest of three projects, the case least likely to
reward a stronger model, so a project whose tasks are genuinely hard
is the open question this row exists to let someone answer. Ask it
once in the constitution conversation, with the default stated; never
re-open it spec by spec.

*Close-out* goes to the top tier under the standard profile whatever
the implementation row says, because it writes the `ROADMAP.md` and
`DECISIONS.md` entries, the acceptance evidence, and the spec's
summary — synthesis and prose, the one implementer dispatch shaped
like planning. The measured close-outs cost $5.66 and $3.74 at the
implementation tier against $0.81 and $1.31 at the top tier's model at
medium, but those specs also handed close-out a pre-assembled bundle,
so the gap is confounded and worth watching in the tier log rather
than trusting.

**A model change the person asks for is written into the table before
it is acted on**, in the same turn, with a tier-log row naming the
date and the spec it changed at — then the session does it. This is
the same principle as everything else here: a preference that lives
only in a session's context dies at the session boundary, and the next
session doesn't know enough to ask. If the person frames it as
temporary, the condition and the date go in the row's comment, and
nobody reverts it by inference — the person says when it ends. The
allowance fallback is the one exception, because it fires on a
condition rather than a request, applies for the rest of the window,
and leaves the rows alone.

**Spec conversations happen in Claude Code, in a spec session of its
own** — the project's first session included, which scaffolds the repo
and then hosts the idea, constitution, and first-spec conversations
(see "Starting a project"). Never inside the implementation session,
whose context is the cost the tiering exists to contain. A session
opens at the settings default — the top tier's model at medium effort
— so a spec session opens by stating its model
and effort (`/effort status` is the authoritative check) and asks the
person to raise effort to high for this session only (`/effort high`).
That is the single manual choice in the whole workflow; it's a choice
about how hard the person's own thinking seat reasons, which is why
it's the one left to them. The spec session also runs planning: once `spec.md` is approved it assembles
the planning bundle, dispatches the `sdd-planner`, then the sign-off,
then writes the spec-conformance summary — everything the top tier
does for a spec, with the person who just wrote the spec still there
for any product question the planner or reviewer returns. When
`plan.md` and `tasks.md` are final, the session ends with the
continuation prompt that starts implementation in a new session.

**Why the session runs on the top tier's model, at medium effort.**
The routing above does the real work: the policy sends every judgment
call away from the session — design to the planner, the sign-off, and
the decision review, correctness to the verification command and the
reviewer, product questions to the person — and what's left is
procedure. The session used to sit one tier down because the top tier
was assumed to charge a premium on every re-send of the longest-lived
context. Fable 5.1 broke that assumption: its cache reads bill at
$0.25 per million tokens, half the Opus rate. Measured across five
specs on three projects, the seat on Fable 5.1 at medium cost about a
third per task of the same seat on Opus 4.8 — mostly because it took
a quarter of the turns — and the allowance held with three projects
drawing at once. Medium effort stays for the behavioral reason: high
effort makes a session investigate before acting, and everything a
hands-off orchestrator reads inflates every later re-send.
`references/design-record.md` has the experiment, its confound (the
baseline turned out to run at high effort), and the follow-ups.

The policy is written into each project's `CLAUDE.md` (see the
constitution template's "Model policy" section), next to the
involvement level — orthogonal settings, both decided once at the
start. The skeptical-reviewer's definition defaults to the
implementation tier (`model: opus`); the orchestrator overrides it up
for sign-off and decision reviews only. See "Keeping reviews cheap" in the
collaboration workflow for the bundles that keep every review — and
every implementer dispatch — from reading the codebase at all.

## The flow at a glance: where each step runs, and on what

Everything above, laid out as the sequence a spec actually follows.
**This is the standard profile's resolution of the role table in the
project's `CLAUDE.md`, with that table's defaults unchanged.** Read
the table itself for what a given project actually does: it is the
authority, this is the illustration. "Fable" and "Opus" stand for the
tier names — under the standard profile the session tier is Fable at
medium and the top tier is Fable at high, so the two coincide
everywhere below. Under the economy profile they don't: the top tier
is Opus 5 and the session tier is Opus 4.8, and every row that reads
"Fable" resolves to one or the other by which tier the role table's
row names. The roles are what's fixed; the names change as models do.

**A brand-new project, once.** The person creates an empty repository
and opens Claude Code in it; the first session is a spec session:

| Step | Where | Model | Who's talking |
|---|---|---|---|
| "Start a new Solowright project" → scaffold from `project/`, committed | Claude Code, **the first spec session** | opens at medium; the person raises effort to high as for any spec session | the person and the session |
| Idea conversation | same session | Fable, high | the person and Claude |
| Constitution → `CLAUDE.md` filled in | same session, committed | Fable, high | the person and Claude |
| First spec → `spec.md` | same session | Fable, high | the person and Claude |
| Plan and tasks onward | exactly as below, from "Plan and tasks drafted" | | |

**Every spec after that.** The project's `.claude/settings.json` opens
every Claude Code session on Fable 5.1 at medium effort; the agent
definitions and the orchestrator's overrides do the rest:

| Step | Where | Model | Who's talking |
|---|---|---|---|
| Spec conversation → `spec.md` | Claude Code, **the spec session** | Fable, high — the session opens at medium, says so, and the person raises effort for this session (`/effort high`) | the person and Claude |
| Plan and tasks drafted | same session | the spec session dispatches `sdd-planner` at **Fable, high** | orchestrator → planner |
| Sign-off | same session | `skeptical-reviewer` at **Fable, high**; one review, at most one re-review | orchestrator → reviewer |
| Spec-conformance summary | same session | Fable | orchestrator → the person |
| Plan and tasks final | **new session** — the spec session ends with the prompt to paste there; the new session opens at medium from settings | — | — |
| Non-routine task | the implementation session | `skeptical-reviewer` at **Fable, high**, on a decision bundle; the session transcribes the recommendation | orchestrator → reviewer |
| Implementation, per task | same session | the implementer the constitution names — `sdd-implementer` at **Opus, high** by default, on a task bundle | orchestrator → implementer |
| Marked per-task review | same session | `skeptical-reviewer` at **Opus, high** | orchestrator → reviewer |
| Phase review | same session | `skeptical-reviewer` at **Opus, high**, on a phase bundle | orchestrator → reviewer |
| Phase pause report | same session | Fable, medium | orchestrator → the person, who attests by using the app and says continue; the session stays open. Only for phases the planner marked with a walkthrough, under the default cadence; an unmarked phase adds its line to the walkthrough list and the session continues |
| Walkthrough finding | same session | the same implementer, on a diagnosis bundle; a decision review at **Fable** if it returns options | the person → orchestrator → implementer |
| Pre-merge sweep | same session | `skeptical-reviewer` at **Opus, high**, documents + spec diff | orchestrator → reviewer |
| Close-out task | same session | `sdd-implementer-fable` at **Fable, medium**, on a close-out bundle | orchestrator → implementer |
| Merge | same session | Fable, medium | orchestrator; ends with the prompt for the next spec session, if `ROADMAP.md` has one |
| Spec merged | **new session** for the next spec, which opens at medium and asks for high effort | — | — |

**The one manual step** is the effort switch at the top of each spec
session. The session prompts for it; it can't be automated under the
standard profile, because the settings file pins effort per model and
the spec session and the implementation session are the same model
there. Under economy they are different models, so the pins do the
work and the prompt is a confirmation rather than a request. Everything else resolves from `.claude/settings.json`, the
agent frontmatter, and the orchestrator's overrides. Both session
boundaries are new sessions, not `/clear`: `/clear` resets context
but keeps the session's settings, which would leave implementation at
high effort after the spec session, and would skip the next spec
session's opening prompt. `/clear` has no place in the workflow;
`/compact` is the tool for an implementation session that grows long.

**Fable's footprint per spec** is the spec session (the conversation
and the handful of turns that dispatch planning), one planner run,
one sign-off (plus at most one re-review), any decision reviews, the
close-out dispatch, and the whole implementation session at medium.
The implementer, the reviewer's phase and per-task checks, and the
sweep run on Opus. Measured across two specs with the implementer
*also* on Fable, that extra row cost about a fifth more of the weekly
allowance per task and bought nothing measurable, which is why the
implementer default sits where it does.

## Build for the scale the project actually has

Solowright is for one person's projects, and the code should say so.
The characteristic failure of a capable implementer working from a
plan is not sloppiness, it's ceremony: an interface with a single
implementation, a configuration value that never varies, a seam for a
swap nobody is going to make. Every one of those is defensible in the
abstract, free to add, and a permanent cost to the one person who has
to read it later.

So the constitution template carries a **Scale** section, filled in
per project with the real numbers — how many users, which machines,
what data, what breaks — followed by five tests that a reader can
actually apply to a diff: an abstraction earns its place at the second
real caller rather than the first imagined one; an extension point
needs a requirement that exists now; a layer needs a boundary that
actually varies; prefer deleting to configuring; backward
compatibility with yourself isn't a constraint, except for data
already on disk.

Written as tests, not as advice, and deliberately. "Don't
over-engineer" is an adjective, and an agent can't check a diff
against an adjective — it resolves the ambiguity the safe way, which
here means building the abstraction. The same failure mode this record
documents for the phase-pause rule and the commit-to-main rule: a
rule the actor can't evaluate becomes a guess, and the guess has a
direction.

The counterweight matters as much as the rule, and lives in the same
section. Tests, error handling for failures that can happen, data
integrity and migrations, care with credentials, and names a cold
reader can follow do **not** scale down. A solo project needs them
more than a team project does, because there's no second reader to
catch what the first missed and the person returning in six months has
forgotten everything and has nobody to ask. Simplicity is fewer moving
parts, not fewer safeguards.

Three places enforce it. The planner reads Scale before drafting,
because that's where over-engineering enters — the implementer builds
what the plan names, and a reviewer checking the diff against the plan
finds it correct. The implementer builds exactly what its task asks
for and returns anything that seems to want a larger shape. And the
reviewer treats structure built past the stated scale as blocking, but
only the nameable kind — an interface with one implementation, an
unused extension point, a pass-through layer — with taste
disagreements left as second-look notes, since a reviewer that blocks
on style gets skimmed.

## When the constitution gains a principle, what happens to existing code

A new principle **binds work from the commit that adds it**. Code
written before it is not retroactively a defect list, and the honest
reason is that the code was built correctly under the rules in force
at the time — which is exactly what this system asks for.

So the default after an amendment is: nothing. Don't open a cleanup
spec, don't annotate the old code, don't let the next reviewer treat
the whole repo as out of compliance. A principle that costs a
retroactive sweep every time it's added is a principle nobody will add.

A cleanup becomes worth doing when the old shape is **actively
costing** something, and that has two concrete tells, neither of which
is "it doesn't match the new rule":

- You keep reading past it. The same indirection gets re-traced every
  time someone touches that area, which is a tax being paid
  repeatedly rather than a shape that merely offends.
- It blocks a change you actually want to make. The next spec has to
  work around it, or would have to extend it to do something simple.

If one of those is true, the cleanup is **its own spec**, with the
normal spec-plan-tasks treatment, scoped to the specific structures
named in `spec.md` — not "apply the new principle to the codebase,"
which has no acceptance criterion and no end. If neither is true,
leave it, and let the principle do its work on everything written
after it. Most of the value of a rule like this is in the code that
doesn't get written.

The exception is a principle about **correctness or data** —
persistence without migration, an unhandled failure that can really
happen, a credential in the wrong place. Those are bugs, they were
bugs before the principle named them, and they take the
bug-after-a-spec-ships path immediately, at whatever size fits.

## Principles worth generalizing

These aren't language-specific or platform-specific — they're patterns
worth carrying into every project from day one instead of rediscovering
each time.

1. **Test the architectural claim, don't just assert it in a document.**
   If a plan document says "this schema is compatible with X" or "these
   colors are distinguishable," that's a testable claim — write the test
   that would catch it being false, don't just write the sentence and
   trust it. This generalizes further than it sounds: "any claim about
   how the system behaves is a test, not a comment."

2. **A passing test is not evidence it can fail.** Mutation-test
   anything that matters: deliberately break the rule the test claims to
   guard, and confirm the test actually goes red. Tests that can't fail
   are surprisingly common and surprisingly hard to spot by reading them
   — they read exactly like real coverage. When one turns out to be
   false-passing, audit for the same *shape* elsewhere rather than
   fixing only the instance found.

3. **A test that asserts on source text pins the spelling, not the
   behavior.** The specific shape principle 2 catches most often: some
   layer is genuinely awkward to test — SwiftUI view bodies, a
   framework's wiring, generated code — so the test reads the source
   file and asserts it contains a string. It passes principle 1's
   letter, since a claim did become a test, and fails principle 2's
   entirely: rename a parameter and it goes red with nothing broken;
   write the same behavior a different way and it goes red; delete the
   behavior and keep the string and it stays green. It cannot fail for
   a behavioral reason and cannot pass for one.

   These don't arrive as a decision, they accumulate — one per spec,
   each locally reasonable, until a third of the suite is source
   scanning. Measured in one project here: 502 of 1,547 tests, 13,000
   lines. The rule worth setting per project, in the constitution's
   Testing section, is narrow rather than absolute: **a source scan
   may pin an injection point that nothing else can reach, and never a
   behavior a unit test could reach instead.** Absolute bans lose to
   the real problem the pattern is solving. And when one of these is
   the only coverage of something load-bearing, the honest reading is
   that the thing is untested.

   The same root cause produces the opposite symptom, and it's worth
   checking for both at once: **testing effort follows testability,
   not risk.** The awkward-to-test layer either gets a fake test like
   the above, or gets none, while some easy pure function next to it
   accumulates exhaustive coverage. Both were found in the projects
   this skill was measured on — one had a third of its suite scanning
   source text, the other had 190 test lines on bounds-checking a
   drawing primitive and zero on the render path whose resize
   short-circuit is the only thing preventing an out-of-bounds write.
   Coverage inverted against risk reads as discipline and isn't. The
   check is cheap: list what would actually hurt if it broke, then see
   where the tests are.

4. **Verify the mechanism, not a proxy for it.** When checking whether
   something actually happened, instrument the thing itself — a log
   statement, a direct check — rather than inspecting a visual or
   indirect artifact that might not reliably show it. A fast or
   synchronous action can complete before any screenshot catches it;
   "I didn't see evidence of X" and "X didn't happen" look identical and
   mean opposite things. This mistake is expensive specifically because
   it produces confident, wrong conclusions rather than uncertainty. A
   corollary worth remembering: a long-standing, well-documented platform
   capability is the least likely thing in the room to be broken —
   suspect the newest, most custom code first, and check a surprising
   "this doesn't work" conclusion against real documentation before
   accepting it.

5. **One source of truth, not two things that could silently drift.**
   Any time the same fact, threshold, or calculation is needed in two
   places, make one canonical and have the other reference it. Two
   independently-written versions that happen to agree today are a bug
   waiting for the day someone edits only one of them.

6. **Correctness over reference-fidelity, but never silently.** When an
   implementation and a design reference (or a spec and an earlier
   assumption) genuinely conflict, resolve toward whichever is actually
   correct — but always surface the divergence explicitly, with
   reasoning, rather than quietly picking a side. The person steering
   the project should see every place execution disagreed with the plan,
   not just the places it matched.

7. **Reserve real "use it yourself" time — don't review only diffs and
   summaries.** Some of the most important catches come from someone
   actually using the running thing, not from reading what changed. A
   summary can describe a feature working correctly while the actual
   feel of it is off in a way no diff would show.

8. **Own mistakes plainly, in both directions.** This applies to the AI
   and the human equally. When a wrong technical conclusion gets
   reached, say so plainly and explain what the right verification
   would have been — don't quietly correct course without naming the
   error. When a person's own edit or assumption caused a problem
   (a stale file handed over as current, a misplaced instruction), that
   deserves the same treatment, not defensiveness.

## The multi-writer file problem

The moment implementation starts, `tasks.md` gets written by more than
one party — the planning conversation adds scope and reshuffles tasks,
and the orchestrator checks boxes and adds findings as it works. This
is different in kind from every other document, which has exactly one
writer, and it needs different handling:

- **Never edit from memory or an old copy.** Before making any change,
  get the actual current file and verify it, even if a version was seen
  five minutes ago in the same conversation.
- **Prefer small, targeted edits over regenerating the whole file.** A
  full-file replacement silently discards whatever the other party
  added since the copy being edited from was taken — checked-off boxes,
  new findings, added scope. This is the single most common way this
  kind of project gets corrupted, and it's avoidable by discipline alone.
- **After any edit, sanity-check structure**, not just content — grep
  for section headers and ID sequences to confirm nothing got duplicated
  or dropped. This catches an editing mistake immediately rather than
  three tasks later.

## Session and context hygiene

Cache re-sends of carried context are the cost — on measured sessions,
97% of all tokens — so context size and turn count are the levers.
Under the dispatch policy the orchestrator's own context is small: the
exploration, diffs, and logs live in the subagents' contexts, and what
the session carries is bundles it wrote to files, short returns, and
bookkeeping. What makes a session large is holding work that belongs
to a different stage, so that is where the boundaries go.

- **Two session boundaries per spec, both new sessions.** The spec
  session ends when `plan.md` and `tasks.md` are final — the spec
  conversation is the largest single context in the workflow and has
  no further value once the files hold it. The implementation session
  ends at the merge — one spec's implementation never carries into
  the next spec's conversation. A project with real documentation
  discipline loses nothing at either boundary: `CLAUDE.md` re-reads
  automatically, and `tasks.md` is exactly the file designed to answer
  "where was I" cold.
- **Phase pauses stay in the session, and get no continuation
  prompt.** The person attests and says continue. `/compact` if the
  implementation session has grown large; never clear or compact
  mid-task. If the person decides to stop at a phase pause, they say
  so or ask for a prompt, and it gets written then — the session
  doesn't offer one in advance. The full rule, with the reason it is
  written as a flat default rather than a condition, is under "Model
  tiering" above, in the paragraph beginning "A phase pause gets no
  prompt."
- Each of the two session boundaries ends with a continuation prompt
  for the next session, so a new session costs the person a paste, not
  a reconstruction.
- Batch bookkeeping into single shell commands. Each turn saved is a
  re-send of the whole context saved.
- In a chat interface without a clear command, the equivalent move is
  starting a fresh conversation with the project's key documents
  uploaded to its knowledge base — same principle, same payoff, since
  the real state was never only in the chat to begin with.
- If a long planning conversation accumulates genuinely valuable
  reasoning that isn't fully captured in the terse final form of the
  docs — the *why* behind a decision, not just the decision — write a
  short session summary before moving to a fresh conversation, so that
  reasoning isn't lost even though it's not literally in the repo.

## The collaboration workflow

See `references/collaboration-workflow.md` for the full, step-by-step
version of this. In short: **the default is to stay inside Claude Code**,
using Plan Mode (research and propose before touching any files) for
real decisions, and a custom reviewer subagent (see
`agents/skeptical-reviewer.md`) for a genuinely independent second look
without leaving the tool. A separate conversation with the person is
reserved for two specific triggers, not general "foundational"
judgment: something in the design turning out infeasible or needing
substantial rework, or a previously-unknown consideration surfacing
that would materially change the project's direction. Everything else
— including plenty of things that feel weighty in the moment —
resolves inside Claude Code. This tiering follows the same risk-based
logic as review cadence above, applied to *which surface a decision
happens on*.

## Starting a project: a realistic first sequence

Not every step below deserves equal engagement. For a solo or personal
project specifically, the natural weighting is uneven: the idea itself,
the spec's user flows, and the design direction are where iteration
genuinely pays off. The constitution's technical choices — framework,
hosting, package manager, and similar — are comparatively fungible;
"good enough, quickly" costs little there. This is a preference, not a
universal rule — say so explicitly if a given project actually wants
more rigor upfront on the technical side (real infra stakes, a team
involved), and follow that instead.

0. **Scaffold, from the skill's own templates.** The person creates an
   empty repository, opens Claude Code in it, and says "Start a new
   Solowright project." This first session is a spec session — it
   opens at medium and asks for high effort like any other — and
   before any conversation it writes the project's skeleton by
   copying the skill's `project/` folder into the repo as it is —
   `CLAUDE.md`, `.claude/settings.json`, `.github/`, `.gitignore`,
   `specs/001-spec-name/`, `design/` — nothing assembled by hand. Three
   adjustments follow: rename `specs/001-spec-name/` to the slug the
   idea conversation settles on; delete `design/` if the project has
   no UI; and keep one settings file — `.claude/settings.json` is the
   standard profile, and if the constitution conversation picks
   economy, replace its contents with `settings.economy.json` and
   delete that file either way, so the project carries exactly one.
   One commit: "Scaffold the project." There is no project template to
   clone — the skeleton lives in the skill so that every new project
   gets the current one, and there is exactly one copy to maintain.
   The orchestrator recreates `.claude/settings.json` from the skill's
   copy if it's ever missing; nobody creates it by hand.
1. **Idea conversation, before any technical decision.** Audience,
   purpose, what makes this distinctive, the core loop or the point of
   the thing. Reaching for a framework choice before the idea itself is
   settled is working backwards — technical decisions usually clarify
   naturally once the idea is clear, not the other way around. Someone
   who prefers to think this through in chat first can, and brings the
   conclusions here.
2. **Constitution conversation.** Platform/language/architecture choices,
   testing philosophy, dependency policy, the person's involvement
   level (ask once, directly, and default to product owner), and the
   model profile (standard by default; economy for a small or personal
   project, or one that should leave the top tier's allowance to other
   projects — see "Three names, one place, two profiles"; if economy,
   swap `.claude/settings.json` for the economy template), and, on the
   standard profile, which implementer the role table's task row names
   (ask this one directly, with the default stated; the rest of the
   table is left at its defaults and moved later if the person wants),
   and the pause cadence (ask it right after the involvement level —
   they sound alike and aren't: one is who approves, the other is how
   often the build stops) — fill in `CLAUDE.md` before any code
   exists, so the first thing an
   implementation session reads is the constitution, not its own
   defaults, and commit it. Move through this efficiently once the idea
   is settled: when someone doesn't have a strong preference on a
   technical choice, recommend a sensible default and explain briefly
   why. "No preference" is a signal to move quickly, not an invitation
   to generate a longer list of options. If asked to help explore an
   option (hosting, for instance), give a genuine, opinionated
   recommendation grounded in what's already been decided — not a
   neutral menu that hands the decision back.
3. **First spec.** Accept that it'll be larger than specs after it (see
   "The first-spec exception"). Push on ambiguity now — it's nearly free
   to resolve in conversation and expensive to resolve after code exists.
   The planner will see only the documents, not this conversation, so a
   decision that lives only in the conversation isn't made yet.
4. **Design exploration**, if the project has a UI — see "Design
   exploration, when the project has a UI" above for what actually needs
   to be in the brief and why. The deliverables are screens exported as
   images plus a tokens document, both becoming implementation
   references for whoever builds from them.
5. **Plan and tasks**, the same way as for every later spec: the spec
   session dispatches the `sdd-planner` on a planning bundle — here
   the spec, the constitution, and the skill's templates as the
   pattern, since there is no previous plan — then the sign-off, then
   the spec-conformance summary. The plan should write down "someone
   should double-check this claim" moments as things to verify, not
   assume them correct; tasks are ordered, small, independently
   verifiable, and tiered by risk for review cadence.
6. **Implement**, in a new session, with the review discipline actually
   followed, not just agreed to in principle. The first phase or two is
   where the pattern either sticks or doesn't — it's worth being strict
   early even if it feels like overkill, because that's also when a
   mistake is cheapest to catch.

**For every spec after the first, this same sequence applies minus
steps 0 and 2** — the scaffold exists, and the constitution stays in
force unless this particular feature genuinely requires amending it,
per `CLAUDE.md`'s own rule (amend explicitly, in its own commit, before
the spec proceeds). Steps 1 and 3 still happen as a conversation with
the person, in a spec session of its own, raised to high effort — a
second or tenth spec doesn't skip the idea-and-design phase just
because the project already has a working codebase.

## Who authors plan.md and tasks.md

Authorship splits along the what/how boundary. `spec.md` is a
conversation with the person; `plan.md` and `tasks.md` are drafted by
the `sdd-planner`, once per spec, first spec included.

**Plans are drafted in a dispatch, against the repo.** A plan against a
real codebase needs the actual model definitions, the actual view
structure, the actual dependency-injection shape — ground truth only a
context with the files open can see. Once `spec.md` is approved, the
spec session assembles a planning bundle with shell — the spec, the
previous spec's `plan.md` and `tasks.md` as the pattern (or the skill's
templates, for a first spec), a file listing — and dispatches the
`sdd-planner` subagent (`agents/sdd-planner.md`) on it, once, at the
top tier. The planner reads the code the spec touches,
writes both files marked Draft, and returns a summary with its token
usage for the tier log. The orchestrator commits the drafts to the spec
branch with the PR still in draft, and the skeptical-reviewer signs
off. The exploration a plan needs is the expensive part of planning,
and this puts it in a discardable context, bounded by the bundle,
instead of in the spec session's own context.

**A first spec goes the same way, and that is a feature.** Its plan
invents an architecture rather than extending one, and the temptation
is to draft it inside the design conversation, which holds all the
context. But the planner sees only the documents, and so will every
implementation session after it — the repo is the only interface. A
first plan the planner can't draft from `spec.md` and `CLAUDE.md`
alone is a sign that a decision still lives only in the conversation,
and the fix is to write it down, not to hand the planner the
conversation.

**`spec.md` stays a conversation with the person**, in a dedicated
Claude Code spec session — the project's first session for the first
spec, a fresh one for each after. It captures product intent,
user-facing behavior, and decisions, and the model writing it should
be reasoning about the product, not reading the code.

**Who signs off depends on involvement level, and this is the one place
the levels differ materially.** At the technical-lead level, the person
reads and approves `plan.md` and `tasks.md` before any implementation
task starts. At the product-owner level,
the planner's draft plus the skeptical-reviewer's sign-off is the gate:
the reviewer checks the draft against `spec.md` and `CLAUDE.md`,
blocking findings go back to the orchestrator to be fixed and
re-reviewed once, and the person receives a **spec-conformance
summary** rather than the plan itself — which acceptance criteria the
plan serves and how, where it deviates from the spec and why, and any
product question it surfaced that needs their call. They approve the
*what* that summary describes; the *how* is already signed.
Supervision hasn't been removed, it's been relocated: to the spec (the
contract), the reviewer (the check), and the escalation triggers (the
exit). One standing rule for the planner: anything that turns out to
be a product decision goes back to the person, never settled silently
in `plan.md`.

## Building tasks.md: from plan to an ordered task list

Turning an approved `spec.md`/`plan.md` into `tasks.md` is a skill worth
being deliberate about, not just "break it into steps":

1. **Every task has a checkable "Verify:" criterion**, not just a
   description of what to build. "Implement the item list" isn't a
   task; "`ItemListView`: list with filter/sort controls, using
   `ItemListViewModel`. Verify: [specific test or manual check]" is. If
   a task's completion can't be checked concretely, it's still too
   vague to hand off.
2. **Phase boundaries follow dependency, not just feature grouping.**
   Foundational work (data models, shared utilities other screens will
   reuse) comes first, in its own phase, before anything that depends
   on it. Within one feature area, view models typically precede the
   views that use them — build and test the logic layer, then build UI
   against it.
3. **Shared components get one task, reused everywhere — not rebuilt
   per screen.** If two screens need the same picker or the same
   formatting helper, that's a task in a shared-utilities phase,
   referenced by every screen's own task rather than reimplemented each
   time it comes up.
4. **The whole spec's task list gets planned before implementation
   starts**, not written phase by phase as you go. This is the same
   "approved spec and plan before implementation" discipline `CLAUDE.md`
   already states, applied to `tasks.md` too — reviewed as a whole
   before handoff, even though execution then proceeds phase by phase.
5. **When real scope gets discovered mid-build that wasn't in the
   original plan, append a sub-lettered task (`T036a`, `T036b`) rather
   than renumbering everything after it.** Cheap, doesn't disturb
   already-completed task references, and reads honestly as "found
   here" rather than implying it was planned from the start.
6. **State the review cadence per phase, not per task** — see "Review
   cadence" above — and state the person's pause cadence separately
   from the reviewer's, per the involvement level in `CLAUDE.md`.
   the `tasks.md` skeleton's handoff note is where both get stated
   explicitly for whoever picks up implementation, along with the shape
   of the report a pause should produce.

## Bugs found after a spec ships

A bug discovered in already-merged code isn't a new spec, and it isn't a
reason to reopen the spec branch that shipped it — that branch's job
ended at merge. Handle it on a spectrum, matching its actual size:

- **Trivial** (a typo, an off-by-one, a clearly-wrong constant, no real
  design implication) — a small dedicated branch (`fix/<short-
  description>`, not the `<NNN>-<slug>` spec convention), a clear commit
  message explaining what was wrong and why, PR, merge. No `plan.md`
  update needed — the commit message carries the explanation.
- **Real, but contained** (the root cause needed real investigation, or
  the fix corrects a genuine misunderstanding about how something
  works, even if the code change itself is small) — same small
  branch/PR shape, but also update the original spec's `plan.md` in
  place to reflect the corrected understanding. `plan.md` is a living
  document, not a frozen snapshot — a fix that changes what's actually
  true about the system belongs there, even months after that spec
  shipped.
- **Bigger than a fix** (the correction needs substantial rework, or
  reveals a genuinely new design question) — this has become a new spec
  in its own right, not a patch. The same two escalation triggers
  apply: infeasibility/rework and a direction-changing unknown are
  exactly the signal that a "bug fix" needs the real spec → plan →
  tasks treatment before more code gets written — the spec in its own
  session with the person, the plan and tasks via the `sdd-planner`.

The per-task triage in `references/collaboration-workflow.md` already
governs how much scrutiny any given fix needs; what this section adds
is the git shape, since "one branch per spec" was never written with
work-that-isn't-a-spec in mind. The same gap showed up at the other
end of the scale — a roadmap paragraph is also work that isn't a spec,
and it needs no branch at all. Both are the same correction: the
branch shape follows the change's size and risk, not the question of
whether it belongs to a spec.

## Using the templates

`project/` is a new project's skeleton, laid out exactly as it lands
in the repo: `CLAUDE.md`, `.claude/settings.json` (the standard
profile) beside `settings.economy.json`, `.github/`, `.gitignore`,
`specs/001-spec-name/` with `spec.md`, `plan.md`, and `tasks.md`, and
`design/brief.md` for projects with a UI. `agents/` holds the four
Claude Code subagent definitions — `sdd-planner.md`,
`sdd-implementer.md`, `skeptical-reviewer.md`, and
`sdd-implementer-fable.md` — which install to `~/.claude/agents/`, not
into any project. Install all four: `sdd-implementer-fable` is the
close-out dispatch under the standard profile, so every project uses
it even when the constitution conversation leaves ordinary tasks on
`sdd-implementer` (the default). Which implementer the task row names
is one of the questions that conversation settles. The document skeletons have
placeholders and inline guidance comments, not fill-in-the-blank forms
— expect to restructure sections as the actual project's needs diverge
from the template, the same way real projects always do.
