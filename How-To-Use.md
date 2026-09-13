# Getting Started

This is the human guide to running a Solowright project: how to start
one, what the files are for, what the day-to-day flow looks like, and
what your part of it is. The AI-facing side — `SKILL.md`, the
templates in `assets/`, the three subagents — is installed once per
machine (see `README.md`, "Installing"); this file is for you.

## Starting a project

1. **Install the skill once**, if you haven't: `README.md` covers it.
   The three subagents (`skeptical-reviewer`, `sdd-implementer`,
   `sdd-planner`) go in `~/.claude/agents/`; they are per-machine, not
   per-project.
2. **Create an empty repository** and open Claude Code in it.
3. **Say "Start a new Solowright project."** The session opens on the
   project's default model and will ask you to switch it to the top
   tier for this conversation — that's the one manual step in the
   whole workflow. It then scaffolds the repo from the skill's
   templates (`CLAUDE.md`, `.claude/settings.json`, the first spec's
   folder, a design brief if there's a UI, a `.gitignore`, a PR
   template) and commits. Nothing to copy by hand.
4. **Have the conversations, in this order.** Not every one deserves
   equal time: the idea, the spec's user flows, and the design
   direction are where iteration pays off. The constitution's technical
   choices are comparatively fungible — if you don't have a strong
   preference, say so and expect a quick recommendation, not a menu.
   - **Idea** — audience, purpose, what makes this distinctive, the
     core loop. Settle this before any framework or hosting talk. If
     you'd rather think this through in chat first, do, and bring the
     conclusions here.
   - **Constitution** — platform, architecture, testing philosophy,
     dependency policy, and your involvement level (product owner is
     the default: you own the spec and try the app; you don't approve
     technical plans). Fills in `CLAUDE.md`. Should move quickly.
   - **First spec** — push on ambiguity here; it's nearly free now and
     expensive after code exists. Expect it to be bigger than later
     specs. Anything decided only in conversation isn't decided: the
     planner will see the documents, not the chat.
   - **Design brief, if there's a UI** — fill in `design/brief.md` and
     hand it to Claude Design; iterate on screens until they're right.
     Deliverables: screens exported as images plus a `tokens.md`. If
     there's no UI, the scaffold skips the `design/` folder.
5. **Approve the spec.** The same session then dispatches the planner
   to draft `plan.md` and `tasks.md`, has the reviewer sign them off,
   and gives you a spec-conformance summary in plain language: which
   acceptance criteria the plan serves, where it deviates from the
   spec and why, and any product question that needs your call. You
   approve the *what*; the *how* is already reviewed. (At the
   technical-lead level you read and approve the plan yourself.)
6. **Start implementation in a new session.** The spec session ends
   with the exact prompt to paste; that new session opens on the
   project's default model and runs the whole spec.

Every spec after the first follows the same path minus the scaffold
and the constitution: a spec session of its own for the conversation
and planning, then an implementation session through to the merge.

## Implementation, and your part in it

The implementation session orchestrates rather than types. Each task
goes to the implementer subagent with a bundle — the task, its plan
section, its acceptance criteria — which builds, runs the project's
verification command, and reports back. The session commits. After
each phase the skeptical reviewer checks the phase's work, and then
you get a **pause report**: what you can now try, where execution
deviated from the spec and why, and what needs a decision. You use the
app and say continue — or report what's wrong, in your own words, and
the session dispatches a diagnosis for it.

You are pulled in beyond that for two things only: something in the
design turns out infeasible or needs real rework, or a previously
unknown consideration surfaces that would materially change the
project's direction. Both are meant to be uncommon.

At the merge, the session ends with the prompt for the next spec, if
`ROADMAP.md` has one.

## The files, at a glance

| File | Written by | Purpose |
|---|---|---|
| `CLAUDE.md` | You + Claude, in the first session | The constitution: platform, rules, involvement level, model policy. Read automatically every session. |
| `.claude/settings.json` | The scaffold | Which model and effort a session opens with. Never hand-edited. |
| `spec.md` | You + Claude, in a spec session | What and why. No implementation detail. |
| `plan.md` | The planner; signed off by the reviewer; updated during implementation | Technical design — the record of *why*, kept current as decisions get made or reversed. |
| `tasks.md` | The planner; then the implementation session, task by task | Ordered execution steps and the tier log. The one multi-writer file. |
| `design/brief.md` | You + Claude, in the first session | Visual/interaction direction, if there's a UI. |
| `design/tokens.md` | Output of the design process | The actual colors/type/spacing settled on. |
| `DECISIONS.md` | You + Claude, as needed | Business/product/process context that doesn't fit the structured docs. |
| `ROADMAP.md` | You + Claude, as needed | Backlog of future specs. Deliberately unordered. |

`DECISIONS.md` and `ROADMAP.md` don't need to exist on day one — create
them the first time something needs a home.

## What runs where

Three roles, three model tiers, named once in `CLAUDE.md`: the top
tier decides (the spec conversation, the planner, the sign-off, and
decision reviews), the implementation tier builds and checks (the
implementer, phase reviews, the pre-merge sweep), and the session tier
orchestrates. Every dispatch logs which model ran and what it cost in
`tasks.md`'s tier log, so the policy stays a measured choice. The full
table is "The flow at a glance" in `SKILL.md`; the reasoning is in
`references/design-record.md`.

## A worked example

**Setting**: partway through implementing a personal website. The
current task is a projects/portfolio page.

**A real decision, resolved without reaching you.** The session's
triage finds the task isn't routine: there's a genuine layout call
(grid versus list, how much detail per project) that `plan.md` doesn't
settle. It doesn't decide. It frames a decision bundle — the task
line, the plan section, the acceptance criteria, the options it can
see — and dispatches the skeptical reviewer at the top tier for a
recommendation. The reviewer reads the site's existing `Card`
component, recommends reusing it in a grid, and flags one thing the
options missed: `spec.md` calls out the zero-projects state, and none
of the options handle it. The session transcribes the recommendation
into `plan.md`, adds the empty state, and dispatches the implementer.
What reaches you is the phase's pause report, same as any other.

**When it does reach you.** Later on the same page, the implementer
stops: `brief.md` specifies a masonry layout, the framework has no
clean way to do true masonry without a third-party library, and
`CLAUDE.md` says no new dependencies without asking. That is a design
decision needing rework, not a judgment call to resolve quietly. The
session restates it in your terms and asks you, before anything is
built either way.
