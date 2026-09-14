# Tasks: [Feature Name]

**Status**: Draft — pending sign-off
**Implements**: plan.md in this directory
**Foundational phases**: [0–1]. Review cadence is per-phase
everywhere; a task marked `review: per-task` gets its own review as
well. State both; an orchestrator left to guess guesses "all of them."

<!-- Before filling this in: the skill's "Building tasks.md" section has
the actual methodology — the "Verify:" criterion pattern, dependency-
ordered phases, when to sub-letter a task instead of renumbering. This
skeleton shows the shape; that section explains how to fill it well. -->

Ordered, small, independently verifiable. Each task should be completable
(and testable) on its own. If a session ends mid-list, resume by finding
the first unchecked task — don't re-verify everything above it unless
something looks off.

<!-- WARNING, and worth leaving this comment in the real file: once
implementation starts, this file gets written by more than one party —
whoever's steering adds scope and reshuffles tasks; the implementing
session (the orchestrator — never the sdd-implementer subagent) checks
boxes and adds findings. Never edit this file from a stale copy.
Prefer small, targeted edits over regenerating it wholesale — a full
replacement silently discards whatever the other party added since your
copy was taken. This is the single most common way a project like this
gets corrupted, and it's entirely avoidable by discipline. -->

Per the constitution: every implementation task ends with an actual build
and, where tests exist for what changed, an actual test run — reported,
not summarized.

---

## Phase 0 — Project scaffolding (one-time)

<!-- Foundational setup. Mistakes here are cheap to catch immediately
and expensive to unwind later — this phase (and the data-model phase
right after it) is where marking a task `review: per-task` is worth it
for anything a dozen later files will depend on. Mark sparingly; the
phase review covers the rest. At the technical-lead level, marked
tasks also pause for the person. -->

- [ ] **T001** — [...] *Verify: [concrete, checkable outcome].*
  *review: per-task*

## Phase 1 — [Data model / core architecture]

<!-- Still foundational. Same tight review cadence as Phase 0. -->

## Phase 2 onward — [feature work, roughly in dependency order]

<!-- Per-phase review here as everywhere — a wrong view or a wrong CRUD
field is cheap to fix after the fact. Re-tighten around anything that
turns out to be a genuine judgment call, even mid-phase. -->

## Final phase — Spec close-out

<!-- Keep this phase in every real tasks.md. Merge is gated on every
task being checked, which makes this checkbox the enforcement mechanism
for updates nothing else forces — no other task references these files,
and no test fails when they go stale. -->

- [ ] **T0XX** — Update `ROADMAP.md` (drop or annotate what this spec
  shipped; add any follow-ups it surfaced) and the repo README if
  user-facing behavior or setup changed. Then request the pre-merge
  whole-spec sweep. *Verify: ROADMAP.md no longer lists this spec's
  work as future; README matches actual behavior; the sweep came back
  clean or its findings were resolved.*

---

## Handoff note

Once this file is signed off (per the involvement level in the
constitution), hand it to the implementer with something like:

> Read [constitution file] and [spec/plan/tasks paths], then begin
> implementing starting at the first task. Involvement level is
> [product owner / technical lead]. Dispatch each routine task to the
> sdd-implementer on a task bundle, per the constitution's model
> policy; verify with the constitution's verification command, then
> commit. One review and at most one re-review per invocation. Have
> the skeptical-reviewer review after each phase with a phase bundle;
> tasks marked review: per-task get their own review as well[;
> technical lead only: and a pause for me after each marked task].
> Pause for me after each phase [or: "run through phases X–Y without
> pausing"], and whenever something unexpected bears on spec
> adherence.

Each phase pause is also where the carried context gets dropped:
compact, or start the next phase in a fresh session — either is fine —
resuming from the first unchecked task.

Every pause produces a report in this shape, in this order, in plain
language — short sentences, everyday words, no task IDs, agent names,
or tier names, for a reader who won't open plan.md. The person may not
be technical, and the report exists so they can act, not so the work
is documented:

1. **Why this pause** — a phase boundary, a spec-adherence question, or
   an escalation trigger. One line.
2. **What you can now do** — behavior that exists and can be tried,
   stated as a user would experience it, so attestation is possible.
3. **Where execution deviated from the spec, and why** — every place,
   per the "never silently" principle, not just the interesting ones.
4. **What needs your decision** — product questions only. Technical
   detail lives in plan.md and the commit log for anyone who wants it;
   it doesn't lead the report.
5. **To continue** — when the next step belongs in a fresh session,
   the exact prompt to paste after `/clear`, in its own fenced block:
   spec directory, files to read, where to resume (the first
   unchecked task, or the phase), involvement level, pause cadence,
   and any model switch the next session needs. Omit it only when
   nothing can proceed until item 4 is answered, and say so.

What the person reports back from the walkthrough — "this looks
wrong," "that didn't happen" — is a finding against an acceptance
criterion, not a task line. The orchestrator restates it, dispatches
a diagnosis bundle to the implementer, and routes the return (see the
constitution's model policy). A fix is logged here as a sub-lettered
task under the task it corrects; a product question goes back to the
person before any code changes.

## Tier log (recommended for the first spec under a model policy)

<!-- The constitution's model policy decides which tier runs each task
at dispatch time — there's no per-phase table to fill in. What's worth
recording here is the evidence: token usage from each subagent return
— implementer runs and reviewer invocations alike — any escape-hatch
miss (a task the orchestrator had to redo itself, and why),
and, if the lighter implementer is on, which tasks it took and whether they held
up. Compare the spec's total against a previous spec of similar size
before treating the policy as settled. Drop this section once a project
has that answer. The Tier column carries the resolved model name —
`opus`, `fable` — never "default" or "top tier": a definition's default
is a pointer that has already changed once, and a log entry has to
stay true after it changes again. -->

| Task / invocation | Tier | Tokens | Outcome / miss reason |
|---|---|---|---|
| Planning: draft (`sdd-planner`) | fable | [...] | plan.md + tasks.md drafted; foundational phases 0–1 |
| Planning: sign-off | fable | [...] | fix and re-review ×1, then signed off |
| T001 | opus | [...] | verified first try |
| T001 review | opus | [...] | signed off; scope statement matched the bundle |
| T014a (walkthrough finding) | opus | [...] | diagnosed and fixed in one dispatch; test added |
| Phase 2 review | opus | [...] | signed off; phase bundle |
| Pre-merge sweep | opus | [...] | signed off; documents + spec diff |
