# Plan: [Feature Name]

**Status**: Draft — pending review
**Implements**: spec.md in this directory

<!-- This document matters more over time than spec.md does. Treat it
as a living record of technical decisions and their reasoning, not a
one-time handoff — update it whenever a real decision gets made,
reversed, or corrected during implementation, not just before coding
starts. A plan.md that only reflects the pre-implementation guess is
less useful than one that's been kept current. -->

## Data model / core types

<!-- The actual types, not the conceptual entities from spec.md. If
there's a real, checkable claim here (e.g. "this schema is compatible
with X"), that claim should have a test that verifies it — see the
parent skill's "Test the architectural claim" principle. Name that test
here once it exists, so the claim and its proof are linked. -->

## Architecture / screens / components

<!-- Whatever the project's shape actually is — view models, API
routes, components. For each nontrivial piece: what it does, and any
non-obvious reasoning behind how it's built, especially anything that
diverges from a design reference or an earlier assumption. State
divergences explicitly with the reason, not silently. -->

## [Cross-cutting decisions as they come up]

<!-- This is where the "one small addition here saves a migration
later" kind of decision lives — a field added now because retrofitting
it after real data exists is expensive, a naming convention adopted
project-wide once, a pattern established once and referenced everywhere
else it applies. Add sections here as real decisions get made; don't
try to anticipate them all upfront. -->

## Known limitations

<!-- Deliberate v1 trade-offs, stated plainly with the reasoning for why
they're acceptable now and what would need to happen to revisit them.
This is different from a non-goal (spec.md) — it's a real gap in what
got built, accepted on purpose, not a feature that was never in scope. -->

## Testing strategy

<!-- What gets tested how, and what's deliberately not automated (with
the reason) vs. what's manual and why. -->

## File structure

<!-- Actual directory layout. Update this when reality diverges from
the original plan — a plan that still shows an old structure after
implementation changed it is worse than no diagram at all. -->

## Resolved decisions

<!-- Same purpose as spec.md's version, but for technical rather than
product decisions. -->
