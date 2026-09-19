# 0002. Responsive layout via marker attributes and !important, not a class refactor

- **Status:** Proposed
- **Date:** 2026-09-19
- **Deciders:** @parul (proposed during the responsive pass, flagged for review)

## Context

`index.html` is a single hand-authored page whose entire layout lives in inline
`style` attributes — every grid, every padding value, every width. It had no
media queries at all, so at 375px 172 elements sat outside the viewport.

Inline styles outrank any stylesheet rule regardless of selector specificity.
That is the whole problem: there is no way to add a breakpoint that changes a
value which is already written inline, short of removing it from the markup or
escalating with `!important`.

The page is also rendered through a React runtime (`support.js`) that reparses
the markup, so anything structural carries a re-render risk (see ADR 0001 for
how that already bit us once).

## Options considered

### A. Marker attributes + a central responsive block using `!important`
Add `data-rsp="hero|case|split|cards3|stats|…"` to the dozen containers that
need breakpoints, and override them from the existing `<style>`. Roughly 40
lines of CSS and a dozen touched attributes.

### B. Refactor inline styles to CSS classes
Lift the inline styles into real rules, then write ordinary media queries with
no `!important` anywhere. The "correct" answer in isolation.

### C. Duplicate mobile markup
Render a second set of containers and toggle between them by breakpoint.

## Decision

Option A. Layout containers carry `data-rsp` markers; a responsive block at the
end of the existing `<style>` overrides them at 1024 / 900 / 760 / 640 / 380,
using `!important` throughout.

## Reasoning

B is what this would be if the page were being written today, and it is worth
saying plainly that A leaves `!important` on about forty declarations — normally
a smell worth removing, not adding.

What decides it is blast radius against benefit. B means rewriting every style
attribute on a 700-line page whose visual design is finished and approved, in a
file with no test coverage, rendered through a runtime that reparses attributes.
The failure mode is silent visual regression somewhere in the middle of a page
nobody diffs pixel by pixel. The benefit is stylistic: the rendered result is
identical either way. That is a bad trade on a personal site whose desktop
layout is already exactly as wanted.

A also confines the change to one reviewable block. The diff is +112/−22, and
every override is in one place rather than scattered through the markup, which
is arguably easier to reason about than the inline styles it overrides.

C was never seriously in play: two copies of the content to keep in sync, and
double the DOM for a page that is mostly text.

The `!important` here is therefore load-bearing, not lazy — it is the only
mechanism that beats an inline style. The stylesheet says so in a comment, so
the next person does not "clean it up" and silently revert the page to
desktop-only.

## Consequences

Good: small diff, one place to look, desktop rendering provably unchanged, no
risk to the finished design.

Bad: a stylesheet where `!important` is normal, which blunts it as a signal —
if a future override genuinely needs to shout, it has no louder voice left.
New responsive work must follow the same pattern (add a marker, add a rule) or
it simply will not apply, and that is not guessable from reading the markup.

Adding a breakpoint now means touching two places: the marker in the HTML and
the rule in the `<style>`. The JS deliberately does not hardcode any breakpoint
— the nav's resize guard reads the burger's computed `display` instead — so at
least the CSS stays the single source of truth for where the breakpoints sit.

## Revisit trigger

Reopen if this page gains a build step, a component framework, or a second page
that wants to share styling. At that point the class refactor stops being a
risky rewrite of finished work and becomes a normal migration, and B wins.
