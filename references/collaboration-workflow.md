# Collaboration Workflow: Claude Code, the Reviewer Subagent, and Chat

The default is to stay inside Claude Code. A separate chat is a
deliberate escalation for specific moments, not the normal loop. This
document is written to be followed without guessing — if a step feels
ambiguous in the moment, that ambiguity is the thing to fix in this
document afterward, not something to improvise around silently.

## One-time setup

1. Place the three agent definitions from this skill's `assets/`
   folder — `skeptical-reviewer.md`, `sdd-implementer.md`, and
   `sdd-planner.md` — in `~/.claude/agents/`, the user-level directory,
   so they're available in every project automatically, not just the
   one they were first set up in.
2. In any Claude Code session, confirm they're recognized: ask "what
   subagents do you have available?" or equivalent, and check all
   three appear.
3. Done. They never need to be recreated per project.

If a specific project wants its own customized version of either
instead of the shared one, a copy at that project's `.claude/agents/`
takes precedence over the user-level one for that project only.

## The per-task loop

**On cadence, stated plainly up front**: the subagent has two kinds of
invocation. The first is per-decision — driven by the nature of each
task (Step 1 below), which could mean zero invocations in an
all-mechanical phase or several in a decision-heavy one; routine work
should never see the subagent on this path, and that's the point of
Step 1 existing. The second is a small set of fixed checkpoints, which
depend on the involvement level in `CLAUDE.md`. At every level: before
a spec's PR comes out of draft and merges — a whole-spec consistency
sweep, not tied to any single task (see "After implementation" below).
At the product-owner level, two more, because the reviewer is standing
in for the person there: sign-off on `plan.md` and `tasks.md` before
implementation starts, and a review after every phase — per task only
for the tasks the planner marked `review: per-task`. Those stand-in
reviews are what let the person's own pauses drop to per-phase, and
they're scoped tightly (see "Keeping reviews cheap" below) precisely so
that cadence stays affordable.

### Step 1 — Is this routine?

Before Claude Code starts on a task, ask whether it's well-specified and
mechanical — matches an established pattern already in the codebase, no
real judgment call involved. If yes: dispatch it (see "The dispatch
loop" below) — no reviewer, no Plan Mode, no separate chat. This is
most tasks, most of the time, and should stay fast.

If the task involves any of the following, it's not routine — continue
to Step 2:
- Touches the data model, a core architecture choice, or something many
  other files will end up depending on.
- A genuine design or product tradeoff with no obviously correct answer.
- A design reference (a mock, an earlier decision) conflicts with
  something else, and reconciling them isn't mechanical.
- Would be expensive to unwind if it turns out wrong — not a five-minute
  fix.

**A task can trip the surface of one of these and still correctly be
routine, if the actual judgment call was already discharged upstream** —
in an approved `plan.md` section, or in an earlier review round — and
what remains at execution is genuine transcription into code, not an
open decision. The test is whether a real, undischarged judgment call
remains at the moment of execution, not whether the topic sounds
architecturally significant. A whole phase reading as "all routine" can
be the success condition of good upfront planning, not evidence the
triage is being skipped — but that's only true if the hard parts were
genuinely settled earlier, not merely unexamined now. Worth being able
to point to *where* a given decision was actually made, not just assert
that it was.

### Step 2 — Frame it in Plan Mode

Activate Plan Mode (`Shift+Tab` twice, or `/plan`) so nothing gets
touched while the question is open. Under the model policy the session
runs at the session tier, which is not the tier that resolves design
questions — so the session's job here is to *frame*, not to propose:
assemble a decision bundle with shell — the task line, the `plan.md`
section, the `spec.md` acceptance criteria, any conflicting excerpts —
and add the options it can see, without picking one. That bundle goes
to the subagent in Step 3. At the technical-lead level, read the
recommendation that comes back — sometimes that alone resolves it, or
needs one small correction you can just state. At the product-owner
level the person doesn't see it; it goes into `plan.md`, and only a
product question inside it — something `spec.md` doesn't settle —
comes back to the person.

### Step 3 — Decide: subagent, or subagent-plus-chat?

Ask whether this is **routine-but-real** (a genuine decision, but Claude
Code is well-positioned to reason through it) or hits one of two
specific triggers — not general "foundational" judgment, which is easy
to over-apply:

1. An aspect of the design or a feature in `spec.md`/`plan.md` turns out
   to be infeasible, or needs substantial rework to actually build.
2. A previously-unknown consideration surfaces where deciding it either
   way would materially change the project's direction — not an
   implementation detail with a reasonable default, a genuine fork.

**Routine-but-real** (the large majority of real decisions, including
plenty that feel weighty in the moment) → dispatch the
skeptical-reviewer at the top tier on the decision bundle from Step 2,
and resolve within the same session:

> "Decision review for T014 in specs/004-search. The bundle is
> scratch/T014-decision.md: the task line, the plan section, the
> acceptance criteria, and the options I can see. Recommend one, per
> your definition."

The subagent starts cold: it sees the invocation prompt, CLAUDE.md,
and a git status snapshot — nothing else from the session. Name the
spec directory and the bundle in the prompt; it can't infer them from
a conversation it never saw. Its report is a recommendation written
as the plan section would read; the session transcribes it into
`plan.md`, commits, and dispatches the task — now routine — on it.

At the technical-lead level, read the recommendation: if it's wrong
or needs a correction, state it and re-invoke once; if it's right, let
implementation proceed. At the product-owner level the same loop runs
without the person: the session transcribes, re-invokes at most once
if the recommendation doesn't fit the spec, and proceeds. The person
hears about it in the next pause report — or immediately, if the
recommendation is **needs the person**: an escalation trigger, or a
product question `spec.md` doesn't settle.

**One of the two triggers** → do the subagent review as well, but also
bring the specific question to a separate, fresh chat before finalizing
(see below).

### Step 4 — Escalating to a separate chat, narrowly

When a question does warrant leaving Claude Code, bring the *question*,
not the project state:

- State the decision or the disagreement plainly, in one or two
  sentences.
- Paste the specific relevant excerpt — the paragraph in question, the
  two conflicting claims, the actual code or test — not the whole file.
- Let that conversation resolve the one question in front of it.
- Take the answer back to Claude Code as a short, targeted instruction.

This is meant to be rare per project — reserved for what would already
earn the tightest review tier, not a routine step. If it's happening for
most tasks, something in Step 1's triage is being applied too
conservatively.

## Drafting plan.md and tasks.md: the planner

Once shipped code is what plans extend (see the skill's authorship
section), the session doesn't draft `plan.md` and `tasks.md` itself —
the spec session dispatches `sdd-planner`, once per spec, with a
per-call override to the top tier named in `CLAUDE.md` (explicit, so
the dispatch lands there whatever the session itself is running on).
The exploration a plan needs is the expensive part of planning; it
belongs in a discardable context bounded by a planning bundle, not in
the session that then carries it through every sign-off round:

```
{ echo "## Spec";                    cat specs/005-export/spec.md;
  echo "## Pattern: previous plan";  cat specs/004-search/plan.md;
  echo "## Pattern: previous tasks"; cat specs/004-search/tasks.md;
  echo "## Files";                   git ls-files <source dirs>;
} > scratch/005-planning.md
```

> "Draft plan.md and tasks.md for specs/005-export. Your bundle is
> scratch/005-planning.md: the spec, the previous spec's plan and
> tasks as the pattern, and the file listing. Read the code the spec
> touches, not the project. Write the two files to specs/005-export/
> marked Draft; don't commit. Report per your definition."

On return: commit the drafts to the spec branch with the PR still in
draft, log the planner's tokens in the tier log's planning rows, then
the sign-off — the skeptical-reviewer at the top tier on `spec.md`,
`CLAUDE.md`, the drafts, and only the existing files the plan claims to
extend. One sign-off and at most one re-review, like every other
invocation; a blocking finding still open after that is fixed by the
orchestrator directly and logged, not sent around a third time. Then
the spec-conformance summary to the person (product-owner level), or
the drafts themselves (technical lead).

A first spec, with no code to plan against, still goes through the
planner: its bundle is the spec, the constitution, and the skill's
templates as the pattern, and the sign-off is the same.

## The dispatch loop: who does the typing

Under the model policy the constitution sets (see the skill's "Model
tiering" section), the main session doesn't implement tasks itself; it
orchestrates. Per task:

1. Step 1 triage, as above. Routine → dispatch. Not routine → frame
   it in Plan Mode, decision review at the top tier (Steps 2–3),
   transcribe the recommendation into `plan.md`; then dispatch what
   remains.
2. Assemble a task bundle with shell — the same move as the review
   bundle, so the content never enters the orchestrator's context —
   and dispatch `sdd-implementer` on it. The bundle carries the task
   line, the `plan.md` section, the `spec.md` acceptance criteria, the
   files to touch, the file whose pattern to copy, and any recorded
   finding from earlier tasks that bears on this one:

   ```
   { echo "## Task";                grep -n "T014" specs/004-search/tasks.md;
     echo "## Plan section";        sed -n '/^## Data model/,/^## /p' specs/004-search/plan.md;
     echo "## Acceptance criteria"; sed -n '/^<criteria heading>/,/^## /p' specs/004-search/spec.md;
     echo "## Files";               echo "Models/SearchIndex.swift and its tests; pattern: Models/ItemStore.swift";
   } > scratch/T014-task.md
   ```

   > "Implement T014. Your bundle is scratch/T014-task.md — task, plan
   > section, acceptance criteria, files, pattern file. Don't read
   > plan.md, spec.md, or tasks.md in full. Report per your definition."

   The implementer's "read beyond the bundle" list in its report is how
   you learn what the next bundle should have named.
3. On return, for a task marked `review: per-task`: re-run the
   constitution's verification command yourself — the filtered one,
   never a raw build — then assemble the review bundle and invoke the
   skeptical-reviewer at its default tier (see "Keeping reviews
   cheap"). For every other task the implementer's verbatim filtered
   output is the verification, and the phase review is the check. Read
   the diff yourself only if something failed. Don't fold the
   reviewer's second-look notes into the code yourself, and don't do
   device or browser checks by hand — the first goes to the log or the
   next bundle, the second is the implementer's Verify criterion or the
   person's attestation.
4. If the reviewer says fix and re-review: dispatch the fix (the
   findings plus the task bundle), then one re-review scoped to the
   findings and the fix diff — and that is the end of the loop. One
   review, at most one re-review, per task. Anything still open after
   the re-review goes in the tier log and is left to the pre-merge
   sweep, not sent around again. An unbounded loop was the single
   largest cost in the first measured spec: a cold reviewer re-reading
   a whole task diff finds a new objection every round.
5. Commit, check the box in `tasks.md`, record findings in `plan.md` or
   `tasks.md` now. You are the only writer of `tasks.md` and the only
   one who commits.
6. Escape hatch: two failed verifications, or a "stopped" report on
   something you consider well-specified → do the task yourself and
   log the miss in the tier log.

One task at a time. The implementer's "stopped on a judgment call"
report is the cheapest escalation in the whole workflow — it costs one
subagent run — so treat it as the system working, not as a failure to
route around.

**When the walkthrough finds something.** At the phase pause the
person uses the app and reports back. A report that something is wrong
is a finding against an acceptance criterion, in the person's words —
not a task line, and not something to diagnose in the session. Restate
it (which criterion or task, what was seen, what the spec says),
confirm the restatement if it isn't obvious, and dispatch a diagnosis
bundle:

```
{ echo "## Report (person's words)"; cat scratch/report.txt;
  echo "## Restatement";             echo "Sorting by date shows newest last; spec criterion 3 says newest first. From T014.";
  echo "## Task";                    grep -n "T014" specs/004-search/tasks.md;
  echo "## Plan section";            sed -n '/^## Search results/,/^## /p' specs/004-search/plan.md;
  echo "## Acceptance criterion";    sed -n '/^3\./,/^4\./p' specs/004-search/spec.md;
  echo "## Files";                   git show --stat --format= <T014 commit> | sed '$d';
} > scratch/T014a-diagnosis.md
```

> "Diagnosis dispatch for a walkthrough finding on T014. Your bundle is
> scratch/T014a-diagnosis.md. Find the cause; fix it only if the fix
> is routine and inside the footprint, and add the test that would
> have caught it. Otherwise return the diagnosis and options. Report
> per your definition."

Route the return the same way as any other: a fix → verify, commit,
log it as `T014a` in `tasks.md`; options → a decision review at the
top tier (Steps 2–3), transcribe, dispatch; a product question → the
person, and a spec amendment before any code changes. The next pause
report tells the person what was reported, what was found, and what
changed, in their terms.

**One implementation session per spec; phase pauses stay in it.**
Cache re-sends — context size times turn count — were 97% of all
tokens on the measured sessions, but under the dispatch loop the
session's own context is bookkeeping, not exploration, and a phase
pause is where the person attests, not where the context has to go.
The report goes to the person; they use the app and say continue; the
same session goes on. `/compact` if the session has grown large;
never clear or compact mid-task, which just buys a re-read. The
session ends at the merge, and the next spec starts in a new one.

**End a session-ending pause with the prompt for the next session.**
The person shouldn't have to reconstruct the handoff; the report's
last item is the exact prompt to paste into the new session, in its
own fenced block. It is self-contained and points at files rather
than carrying state — anything decided at the pause goes into
`tasks.md` or `plan.md` first. The spec session ends this way when
`plan.md` and `tasks.md` are final:

> Implement specs/005-export. Read CLAUDE.md and specs/005-export/
> spec.md, plan.md, and tasks.md, then start from the first unchecked
> task (Phase 1). Involvement level: product owner. Dispatch per the
> constitution's model policy; skeptical-reviewer after each phase on
> a phase bundle; pause for me after each phase.

A merge ends the same way, with the prompt for the next spec session
if `ROADMAP.md` has an obvious next item — including the reminder to
switch that session to the top tier before starting. A phase pause
gets a prompt only when the person says they're stopping there; the
resume form is the same, from the first unchecked task.
If the next step is the person's decision, say that instead.

**Batch the bookkeeping.** After a task, the commit, the checkbox, and
the tier-log row are one shell command, not three tool calls; bundle
assembly and dispatch run back to back. The orchestrator re-sends its
whole context on every turn, so every turn removed is that re-send
removed.

## After implementation, not just before

The subagent is equally useful pointed at something already built,
not only a plan:

> "Have the skeptical-reviewer check this commit's claims against what
> actually got tested. The diff is in scratch/review-input.diff."

One mechanical constraint: the reviewer has no Bash, deliberately —
its read-only design is the point — so it cannot run `git diff`,
`git show`, or `git log` itself. When a review concerns a commit or a
diff, the invoking session must supply it: paste the diff into the
invocation prompt, or write it to a file and name that file. Without
that, the reviewer can only see the current state of the files, not
the change.

Good moments to do this: anything that felt uncertain while it was being
built, and before a spec's PR comes out of draft and merges. For that
pre-merge pass, the close-out edits come first — update ROADMAP.md and
the repo README (per the constitution's close-out step) *before*
invoking the sweep, so the sweep verifies the close-out instead of
pre-dating it. Then say explicitly that this is the pre-merge pass, so
the whole-spec sweep happens on purpose rather than as a guess about
scope:

> "This is the pre-merge whole-spec sweep for specs/004-search. Have
> the skeptical-reviewer sweep spec.md, plan.md, tasks.md, ROADMAP.md,
> DECISIONS.md, and the README for drift before this merges."

## Keeping reviews cheap

The reviewer is expensive when it reads the whole codebase to answer a
narrow question, and at the product-owner level it's invoked often
enough that this matters. Two levers: what it's given, and which tier
it runs at.

**Give it a bundle, not pointers.** For a per-task review, assemble one
scratch file containing everything the review needs — the uncommitted
working-tree diff, the task line, the `plan.md` section it implements,
and the `spec.md` acceptance criteria it serves — using shell, so the
orchestrator never loads that content into its own context:

```
{ echo "## Task";                grep -n "T014" specs/004-search/tasks.md;
  echo "## Plan section";        sed -n '/^## Data model/,/^## /p' specs/004-search/plan.md;
  echo "## Acceptance criteria"; sed -n '/^<criteria heading>/,/^## /p' specs/004-search/spec.md;
  echo "## Diff";                git add -A && git diff --cached;
} > scratch/T014-review.md
```

Stage first: plain `git diff` omits untracked files, and a bundle that
misses a new file costs a whole extra round. Staging is harmless here —
the orchestrator commits the task next anyway.

For a per-phase review — the cadence for mechanical phases — the recipe
is the same, just wider: `git diff <first phase commit>^..HEAD` in
place of `git diff`, every task line in the phase, and each plan
section they implement.

A reviewer with the whole question in front of it has no reason to go
looking, which is a stronger constraint than telling it not to. The
invocation then names the bundle and nothing else:

> "Per-task review of T014: everything you need is in
> scratch/T014-review.md. Don't read the codebase or grep for context
> beyond it unless a specific finding requires following a reference —
> and say so in your scope statement if you do."

Scope by invocation type:

- **Per-phase review** (every phase): the phase bundle, plus
  `CLAUDE.md` it already has. Nothing else. A phase-end review of
  well-specified work is a transcription check across several tasks,
  not a judgment call.
- **Per-task review** (only tasks the planner marked `review:
  per-task`): the task bundle, same rule.
- **Re-review**: the previous review's findings and the diff since that
  review — not the whole task diff again. This is the one case where a
  reviewer is handed prior findings on purpose: its job is to check
  them, not to audit afresh.
- **Plan/tasks sign-off**: `spec.md`, `CLAUDE.md`, and the draft
  `plan.md`/`tasks.md` — plus, for a project with shipped code, only
  the existing files the plan claims to extend or depend on.
- **Pre-merge sweep**: the whole document set for that spec plus the
  spec's full diff against main (`git diff main...HEAD`) — the
  documents and the change, not the codebase. This is the one
  invocation that's supposed to be broad, and it happens once per
  spec; the first measured sweep read the codebase and cost more than
  five tasks, which is what the documents-and-diff bound is for.

**Tier by invocation type.** The reviewer's definition defaults to
the implementation tier (`model: opus`), which is right for per-phase
and per-task reviews — checks of a diff against the plan sections it
implements, and the frequent case. Override up to the top tier named
in `CLAUDE.md` only where the reviewer is exercising judgment rather
than checking transcription: plan/tasks sign-off and the decision
reviews from Step 3. The pre-merge sweep stays at
the default tier — it's broad by design, which makes it the most
expensive single invocation, and the orchestrator adjudicates its
findings anyway. The default should be the frequent
case, because forgetting to override up costs a lesser review while
forgetting to override down costs the budget.

**Log it.** The subagent's return reports its token usage. Record each
reviewer invocation in the spec's tier log alongside the implementer
runs — invocation type, the resolved model name (`opus`, `fable` —
never "default," which is a pointer that can change under the log),
tokens. The reviewer's scope statement,
the first line of its report, says what it actually read; if the
tokens or the scope statement show it examining far more than the
bundle, the invocation was too loose, and the log will show which
invocation type is the outlier.

## Recording what a review decided

After acting on a review's findings, record each real decision in the
document it belongs to: a plan.md entry for a technical decision
(plan.md is a living record — this is exactly the content it exists
for), a spec.md correction if behavior was mis-stated, a CLAUDE.md
principle if the lesson generalizes, or a DECISIONS.md entry for
business or process context. Include a line on why it was resolved
that way, and reuse the review's own severity labels (blocking /
second look / solid) so the record and the reviews speak the same
language. This is what makes a discharged judgment call citable later
— Step 1's triage can point at where a decision was actually made
instead of just asserting that it was.

Record findings and resolutions, never coverage. A "reviewed on this
date, found clean" marker is a suppression list waiting to happen: the
sweep part of a review is drift detection, and any later change can
put two documents that agreed at the last review into contradiction —
a past clean verdict says nothing about the present. For the same
reason, never paste a previous review's verdict into a new reviewer
invocation. A fresh reviewer that starts from its predecessor's
agreement is anchored on exactly the failure mode it exists to catch;
give it the current documents and the current question, nothing else.

The main session does this writing, not the reviewer — the reviewer
has no write access, and that's part of its design, not a gap to work
around.

## When multiple findings surface at once

A subagent review, or a spec close-out pass, often surfaces several
things together — some routine-but-real, some genuinely needing the
person's own attestation, some hitting an actual escalation trigger.
Don't bundle all of it into one "come back and sort through this"
conversation just because it surfaced at the same time. Before bringing
anything to the person:

- Resolve every routine-but-real item first, inside Claude Code, via
  Plan Mode and the subagent — same as any other task, regardless of
  whether it happened to surface alongside something that does need
  the person.
- Separate what's left into two categories: things only the person can
  attest to (behavior they'd have to actually use the app to confirm —
  not determinable from the test suite or a diff), and things that hit
  one of the two escalation triggers.
- Bring only those two categories to chat, and say plainly which is
  which. A person's attention during an attestation pass shouldn't also
  get spent re-litigating a design-polish call that could have been
  settled without them.

## What "good" looks like over time

This workflow is working if: routine tasks move through Claude Code
without ever surfacing here; the subagent gets invoked at its fixed
checkpoints and only occasionally beyond them, scoped tightly enough
that its cost tracks the size of what it's reviewing, and mostly comes
back either clean or with something genuinely worth acting on; the
person's pauses are ones they actually read rather than click through;
and a separate chat gets used rarely enough
that each instance is memorable, not routine. If any of those stop being
true, that's worth revisiting — including reconsidering the triage in
Step 1, which is a judgment call that should improve with actual
practice, not something to treat as fixed on first use.
