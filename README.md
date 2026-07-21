# Draw — Milestone Raffle

A raffle wheel for community events that run as a **ladder of milestones** rather than a single draw.

**→ [Open the app](https://manueleverafter.github.io/raffle)**

Built for stream "tap tap" style events: viewers hit a milestone, submit proof, a mod verifies it, and each
milestone draws its own winner. A grand prize at the end goes to whoever took part in *every* milestone.

## What it does

- **A milestone ladder.** Defaults to every 50K up to 1M — 20 milestones, majors at each 250K. Pick one,
  add the names who submitted verified proof, spin. One winner per milestone.
- **Grand prize eligibility works itself out.** Instead of hand-checking who took part in everything, the
  app tracks it. Someone who missed one milestone shows as `5/6` and is excluded automatically.
- **It understands a live that ends early.** A milestone only counts once someone is in it, so if the stream
  stops at 600K the grand prize is judged on the milestones you actually reached — not all 20.
- **Winners can pass.** When someone declines a prize, reroll excludes them **from that milestone only** —
  they stay eligible everywhere else. Passing is reversible, so anyone who stepped aside can be put back.
- **A roster that grows.** Names you type are remembered. By the third milestone you are ticking checkboxes
  instead of retyping.
- **Editable before you spin.** Rename milestones, change minor/major, add or delete them, rename the grand
  prize. Renaming is safe — it never orphans people already ticked in.

## Fairness

Draws use `crypto.getRandomValues()` — a cryptographic RNG, not a seeded one — with rejection sampling so
no name is even fractionally more likely than another.

You do not have to take that on faith. **Fairness check** runs 10,000 draws against the names on your wheel
right now and shows the spread, so you can settle an argument in front of the room with real numbers.

Seat positions also reshuffle between rounds. That part is cosmetic — it changes nothing about the odds —
but it stops anyone reading patterns into the layout.

## Your data stays in your browser

There is no server and no account. Everything lives in your browser's local storage, which means:

- Nobody else can see your event, and you cannot see theirs.
- Two mods opening the site get **two separate events**. Ticks do not sync between them.
- Clearing site data wipes it.

Use **Save file** to export the whole event as JSON, and **Load file** to restore it — on another machine,
after a browser clear, or to hand off to another mod.

## Keyboard

| Key | Does |
| --- | --- |
| `Space` | Spin, or dismiss the winner card |
| `Esc` | Close any dialog |
| `Ctrl`/`Cmd` + `Enter` | Add the names you just pasted |

## Running it yourself

One file, no build step, no dependencies. Either:

- Download `index.html` and open it in any browser, or
- Fork this repo and turn on GitHub Pages (Settings → Pages → deploy from `main`)

Open it in a browser and it works offline.

## License

MIT — use it, change it, run it for your own community.
