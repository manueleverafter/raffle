# Changelog

What changed, and why. Newest first.

---

## Hardening pass

Three audits of the whole file plus reproduction of every claim before fixing it. Grouped by what the work
was protecting against.

### The draw itself

**A spin can no longer be corrupted by anything that happens during it.** The rail stayed live for the full
5.2 seconds, and two things could go wrong, both reproduced:

- Clicking a different milestone mid-spin filed the winner against *that* milestone. A 50K participant was
  recorded as the winner of 300K, which they had never entered.
- Ticking anyone mid-spin reshuffled the segments under the running animation, so the pointer came to rest
  on a different name than the one announced — the worst possible failure for an app whose pitch is a
  verifiable draw.

The wheel's contents are now frozen for the duration, the milestone is captured before the animation starts,
the rail is visibly locked, and the mutating handlers refuse to run while `spinning` is true.

**Commas no longer split a pasted line.** `Doe, Jane` became two entrants, each holding their own odds.
Lines are separated by line breaks and semicolons only.

### Losing an event

**Storage failures were silent.** `save()` swallowed every error, so a quota problem, blocked storage, or a
private window meant a whole event could be run in the belief it was saved, discovered only on refresh.
Every write is now read back, and a failure raises a banner offering an immediate export that clears itself
once a save succeeds.

**A malformed backup used to be permanent.** The file was written to storage *before* anything tried to
render it, so a bad file threw on load, then threw again on every subsequent load — unopenable without
devtools. Imports are now coerced to a known shape, proven to render, and only then persisted; a failure
rolls back and says so. Saved data gets the same treatment on the way in.

**Damaged saved data is set aside** under its own key rather than being overwritten by the first click, so
it stays recoverable by hand.

**Undo always confirms** and names the winner it is about to delete. It sits one button away from Reroll and
used to destroy a result on a single click with no way back.

**A second tab no longer clobbers the first.** A `storage` event means another tab owns the event, so this
one stops saving and says so.

### Telling the truth about the wheel

The rule deciding who could be drawn was written out three times — once to build the wheel, once to count
it, once to explain it — and they drifted. That is why the UI kept insisting names were available after
quietly filtering them out. `poolBreakdown()` is now the single account, returning the survivors *and* the
reason anyone was removed.

Downstream of that:

- An empty wheel names its actual cause — everyone passed, everyone already won, or nobody added yet —
  instead of telling you to add names you can see on screen.
- The eligibility list marks held names `HELD — WON 50K` rather than showing them as available.
- The ladder row and the `N of M reached` header can no longer disagree.
- Reroll and Undo dialogs report what will actually be restored.

**Putting the last passer back clears the empty result behind them.** That shell kept a milestone
permanently "reached" with nobody in it, which could leave the grand prize impossible to award.

**Opening a dialog while one is pending settles the first** rather than orphaning its promise, so a
confirmation can no longer answer a question the operator never saw.

### Shape and speed

**Schema versioning.** The storage key used to carry the version, so bumping it silently orphaned every
saved event — which had already happened once. The version now travels inside the payload, migrations are an
append-only list, saves under the old key are read once and carried forward, and a file from a newer build
is refused with an explanation instead of half-adopted.

**`reached()` is memoized per render pass.** It was recomputed once per roster row, making the cost
quadratic in roster size. Ten filter keystrokes against 300 names now cost 5ms in total.

**The spin tick is throttled by time** as well as by segment, so a large wheel no longer spawns thousands of
oscillators during the one animation that has to stay smooth.

### Reaching everyone

**Phones never worked.** There was no `viewport` meta, so the 940px breakpoint could not fire and every
phone rendered a shrunken desktop layout. Added, along with the rules it reveals: 16px fields so iOS stops
zooming on every tap, the rail ordered above the wheel, 40px touch targets on destructive buttons, and no
nested scroll traps.

**The wheel is a canvas and was invisible to screen readers.** It now carries a label naming who is on it.
Dialogs trap Tab and hand focus back. Light-mode secondary text was under the contrast floor while carrying
load-bearing information.

**The fairness panel explains that it is stale** instead of silently emptying mid-conversation.

### Smaller

- Ticking someone no longer discards half-typed text, the roster filter, scroll position, or focus.
- Draws record a timestamp.
- New event resets the skip rules instead of silently inheriting them; names are kept.

---

## Per-tier skip rules

One global "skip people who already won" switch was too blunt. Minor, major, and the grand prize each have
their own, so an event can spread the small prizes around while leaving the grand prize open to everyone who
qualified.

It was also silently misleading: with the rule on, the eligibility panel still printed ELIGIBLE beside
people who could not be drawn. Held names are now marked with the milestone they won, and the wheel reports
how many are held back.

---

## Bulk roster controls

Tick all / Untick all, acting on whatever the filter is showing rather than the whole roster — narrowing to
a few names and hitting the button cannot quietly change rows scrolled out of view.

---

## Passing on a prize made visible and reversible

The wheel silently shrank when people passed, with nothing on screen explaining why, and a milestone where
everyone had passed was a dead end with no way back.

Passes are now listed with a **Put back** button each, the winner banner says `after 2 passed (cedy,
manuel)`, and a reroll that would empty the wheel explains itself and offers to undo the draw instead.

---

## Rebuilt around a milestone ladder

The original two-list model did not match how the events actually run. Replaced with a ladder of milestones,
each drawing its own winner, and a grand prize derived from participation.

- Grand-prize eligibility is computed, not hand-checked.
- Eligibility counts only the milestones the live **actually reached**, so a stream ending early is handled
  without any manual adjustment.
- Milestones are editable before a draw — rename, re-tier, add, delete, reorder the ladder, rename the grand
  prize. Ids are stable and separate from labels, so renaming never orphans anyone already ticked in.
- Prize amounts were removed: rewards are often merch or items, not money.

---

## Hosting

Published to GitHub Pages. Serving the file standalone surfaced two things the embedded copy had hidden: no
CSS reset, so the browser's default body margin showed as a pale band at every edge; and no charset, so
unescaped non-ASCII in the script rendered as mojibake. Both fixed at the source.

`robots.txt` and a `noindex` tag keep the site out of search results; the link still works for anyone who
has it.

---

## Fairness

Draws moved from `Math.random()` to `crypto.getRandomValues()` with rejection sampling. A **Fairness check**
runs 10,000 draws against the current wheel so a disputed result can be settled with real numbers rather
than assurances.

Seat positions reshuffle between rounds. That is cosmetic — it changes nothing about the odds — but it stops
anyone reading patterns into where names sit.

Context for the original report: with 5 names, one name winning 5 spins in a row is about 1 in 625.
Uncommon, not evidence of a broken wheel. The picker measured uniform across 600,000 draws.

---

## First version

A wheel with minor, major, and MVP tiers, where MVP drew from the intersection of the other two lists.
Superseded by the milestone ladder once the real event mechanics were clear.

Three bugs shipped in this period were caused by writing code without ever opening the page:
`confirm()` and `navigator.clipboard` are blocked in sandboxed frames, and the `hidden` attribute loses to
any class that sets `display`. Everything since has been verified in a browser before being called done.
