# Draw — Milestone Raffle

A raffle wheel for community events that run as a **ladder of milestones** rather than a single draw.

**→ [Open the app](https://manueleverafter.github.io/raffle)**

Built for stream "tap tap" style events: viewers push toward a goal, each milestone gets hit, people submit
proof, a mod verifies it, and every milestone draws its own winner. A grand prize at the end goes to whoever
took part in *all* of them.

---

## Why it exists

The wheel was never the slow part. The slow part was everything around it — reading through submissions,
checking who had taken part in what, and retyping the same names into a wheel over and over. The grand
prize was worst of all, because qualifying for it meant hand-checking every milestone for every person.

This does the bookkeeping. You still decide who is verified; the app remembers the rest.

## How you use it

1. **Pick a milestone** from the ladder on the right.
2. **Add the names** who submitted verified proof — paste a list or type them.
3. **Spin.** One winner per milestone.
4. Repeat down the ladder. The grand prize wheel fills itself in.

Names you add are remembered, so by the third milestone you are ticking checkboxes instead of retyping.

## What it handles

### The grand prize works itself out

Eligibility means "took part in every milestone", which is tedious to check by hand and easy to get wrong.
The app tracks it: someone who missed one shows as `5/6` and is excluded, someone with the full set is
marked eligible. Nothing to type.

### A live that ends early

Milestones are only counted once somebody is actually in them. If the stream stops at 600K, the grand prize
is judged on the milestones you *reached* — not the ones you had planned. The header shows
`6 of 20 reached` so it is clear what is being counted.

### Winners who pass

People decline prizes — they already won something, or they are only in it for fun. **Reroll** redraws the
milestone with that person excluded **from that milestone only**; they stay eligible everywhere else.

Passing is reversible. Anyone who stepped aside is listed with a **Put back** button, because somebody who
passed early may well want back in once the person they stepped aside for gives a prize up.

If everyone has passed and a reroll would leave the wheel empty, the app says so and offers to undo the
draw instead — which clears the passes and puts the full field back.

### Editing before you spin

Milestones are not fixed. Rename them, switch minor/major, add or delete, reset to the default ladder, or
rename the grand prize. Renaming is safe: milestones carry a stable internal id, so changing a label never
orphans the people already ticked into it.

## Is the draw actually fair?

Draws use `crypto.getRandomValues()` — a cryptographic RNG, not a seeded one — with rejection sampling, so
no name is even fractionally more likely than another.

You do not have to take that on faith. **Fairness check** runs 10,000 draws against the names on your wheel
right now and shows the spread, with a verdict based on standard deviation rather than eyeballing. If
somebody disputes a result, you can settle it in front of them with their own names.

Seat positions also reshuffle between rounds. That part is purely cosmetic — it changes nothing about the
odds — but it stops anyone reading patterns into where names sit.

Worth knowing: a name winning several times in a row is less unlikely than it feels. With 5 people, one
name taking 5 straight is about 1 in 625. Uncommon, not broken.

## Your data stays in your browser

There is no server and no account. Everything lives in your browser's local storage, which means:

- Nobody else can see your event, and you cannot see theirs.
- Two mods opening the site get **two separate events**. Ticks do not sync between them.
- Clearing site data wipes it.

Use **Save file** to export the event as JSON and **Load file** to restore it — on another machine, after a
browser clear, or to hand off to another mod. If the page is embedded somewhere that blocks downloads, the
backup is copied to your clipboard instead.

## Reference

### Buttons

| Control | Does |
| --- | --- |
| **Spin** | Draws a winner for the selected milestone |
| **Shuffle seats** | Rearranges the wheel. Cosmetic — the odds do not change |
| **Reroll** | Winner passed. Redraws, excluding them from this milestone only |
| **Undo draw** | Clears the winner *and* the passes, so the milestone starts fresh |
| **Put back** | Returns someone who passed to this milestone's wheel |
| **Tick all / Untick all** | Bulk-ticks the roster for this milestone. Respects the filter |
| **Edit** | Rename, re-tier, add or delete milestones; rename the grand prize |
| **Theme** | Cycles Auto → Light → Dark. Auto follows your device |
| **Fairness check** | 10,000 test draws against the current wheel |
| **Copy results** | All winners as text, ready to paste into a chat |
| **Save / Load file** | Export or restore the whole event as JSON |
| **New event** | Clears the roster, every tick, and every winner |

### Keyboard

| Key | Does |
| --- | --- |
| `Space` | Spin, or dismiss the winner card |
| `Esc` | Close any dialog |
| `Ctrl`/`Cmd` + `Enter` | Add the names you just pasted |

### Settings

**Skip people who already won** (off by default) leaves anyone holding a prize off later wheels. Left off,
everyone stays in every wheel they qualify for and you handle repeats case by case with reroll.

## Running it yourself

One file, no build step, no dependencies, no network calls. Either:

- Download `index.html` and open it in any browser — it works offline, or
- Fork this repo and turn on GitHub Pages (Settings → Pages → deploy from `main`)

This copy is served with `robots.txt` and a `noindex` tag, so it stays out of search results. Delete
`robots.txt` if you want yours indexed.

---

## Notes for anyone editing this

Things that broke during development and are easy to reintroduce.

**Escape non-ASCII characters in JavaScript.** Write `'—'`, not `'—'`. When the file is served without
an explicit charset the browser may read it as windows-1252 and render `â€"`. HTML entities like `&mdash;`
are fine in markup; the hazard is string literals in the script.

**`[hidden]` needs `!important`.** The attribute only carries user-agent weight, so any class setting
`display` silently overrides it and the element stays visible. There is a global rule near the top handling
this — keep it.

**`requestAnimationFrame` pauses in background tabs.** A spin started before an alt-tab would otherwise
hang forever with the button stuck disabled. `finish()` is idempotent and reachable three ways: the
animation ending, a `visibilitychange`, or a guard timer.

**Standalone pages get no CSS reset.** Embedded hosts often inject one; served raw, the browser's default
8px body margin shows as a pale band at every edge. `html, body` carry their own margin reset and
background for this reason.

**Canvas does not restyle itself.** Colors are read at paint time, so any theme change has to trigger a
redraw — there is a `MutationObserver` on `data-theme` plus a `prefers-color-scheme` listener.

**Sandboxed frames block `confirm()` and `navigator.clipboard`.** Both fail silently, which reads as a dead
button. Confirmations go through the in-page dialog (`ask()`), and copying falls back to a hidden textarea.

**Milestone ids are not labels.** Ticks are keyed by a stable id so renaming a milestone does not orphan the
people already in it. Keep them separate.

**Colour tokens are calibrated per background.** `--line` is tuned for borders on the panel background;
drawn on the darker page background the same value has far more contrast. That is what `--ring` is for.

## License

MIT — use it, change it, run it for your own community.
