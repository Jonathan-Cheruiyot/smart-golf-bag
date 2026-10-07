# CLAUDE.md

Steering document for Claude Code on the Smart Golf Bag project.
Read this before writing any code. `SPECIFICATIONS.md` has the build phases.

---

## What this project is

A UI mock-up of a smart golf bag for a university HCI course (Project 1:
Interface to a Smart Object). Svelte 5 + Vite, no backend, no real sensors.
Everything is simulated from a testing panel on the same page.

Author: Jonathan Cheruiyot. He presents this live and answers questions, so
every non-obvious decision needs a one-line reason he can repeat out loud.

---

## The design concept, in one sentence

**The bag is a printed instrument. The phone is a screen.**

The two devices deliberately use different visual languages because their
physical situations are different, and that contrast is the design argument
of the project. Do not unify them into one generic look.

### Why the bag screen looks like e-ink

A display bolted to a golf bag lives in direct sun, runs for days on a small
battery, and gets glanced at for under two seconds while walking. That drives
a transflective, e-ink-style panel: warm paper white, pure ink black, one red
for alarms, no backlight glow, no gradients, no shadows, enormous numerals,
hairline rules. It should look like a yardage book printed onto a device,
not like a web dashboard.

### Why the phone looks like a phone

The phone is held close, indoors or in a pocket, on an OLED screen, with the
user's full attention. It carries everything that takes more than a glance:
configuration, history, and remote alerts. Dark ground, fine type, denser
information.

### The rule that keeps them honest

**Nothing that requires more than a two-second glance may appear on the bag
screen.** If a feature needs reading, scrolling, or a decision, it belongs on
the phone. Apply this rule when deciding where to put anything new.

---

## Design tokens

Define these once as CSS custom properties in `src/app.css` and use them
everywhere. Do not hardcode hex values inside components.

### Bag display (e-ink)

```css
--paper:        #E6E2D7;  /* panel ground */
--paper-sunk:   #DAD5C7;  /* inset areas, rules */
--ink:          #15171A;  /* primary text, numerals */
--ink-soft:     #68645A;  /* labels, secondary (4.56:1 on --paper) */
--flag:         #B23A2E;  /* alerts ONLY */
--bezel:        #2A2B28;  /* the physical housing around the screen */
```

### Phone (OLED)

```css
--oled:         #0A0B0C;
--oled-raised:  #16181A;
--phone-text:   #F2F0EA;
--phone-soft:   #8A8578;
--phone-rule:   #26282A;
--flag:         #B23A2E;  /* same red, shared product family */
--fairway:      #4F8A5B;  /* confirmations only, used sparingly */
```

### Page (the drafting-table harness around both devices)

```css
--table:        #F2EFE7;
--grid:         #E2DCCD;  /* hairline graph rule, 1px, very low contrast */
--table-ink:    #1A1C1E;
--table-soft:   #6B675C;
```

### Type

One family, three registers, loaded from Google Fonts:

- `IBM Plex Mono` — all device chrome, labels, numerals. Tabular figures mean
  the club count does not jitter when it changes.
- `IBM Plex Sans` — phone body text and page UI.
- `IBM Plex Serif` — page headings only, which gives the printed-document
  voice to the harness without touching the devices.

Plex was drawn for technical and industrial contexts, which is the right
register here.

### Geometry

- The bag bezel is rounded (it is a physical housing). Everything **inside**
  the bag screen is square cornered. E-ink does not do soft corners.
- The phone body is rounded. Elements inside it get at most 4px.
- Rules are 1px hairlines, not borders on cards.
- Spacing scale: 4, 8, 12, 16, 24, 32, 48.

---

## Motion

`MOTION.md` governs every animation in this project. Read it before adding
any transition, and apply its per-phase table as you build each phase.

The one-line version: **motion expresses material.** The bag display is
e-ink and snaps with no easing. The phone is OLED and moves smoothly. The
page harness reveals once on load and is then still. Never apply one global
transition style across all three.

---

## Hard prohibitions

These make student work look machine-generated. Do not use any of them.

- Gradient backgrounds of any kind, especially purple-to-blue
- Drop shadows on cards. Use a hairline rule or a tone change instead
- Emoji anywhere in the UI. Icons are inline stroke SVG, drawn in this file's
  palette, or they are text labels
- Inter, Roboto, Arial, or system-ui as a display face
- Rounded cards floating on a light gray page, the default dashboard look
- Lorem ipsum or invented statistics. If a number is unknown, write a
  bracketed placeholder like `[TBD]` and flag it
- Generic names like `Card`, `Widget`, `Container` for components. Name them
  after what they are: `BagDisplay`, `PhoneShell`, `ClubRack`

---

## Accessibility, which is graded in an HCI course

- Real `<button>`, `<a href>`, `<input>` with a `<label>`. Never a click
  handler on a `<div>` or `<span>`, because Tab skips it.
- `aria-label` on any icon-only control.
- Touch targets at least 44px. The golfer is wearing a glove.
- Text contrast at least 4.5:1, or 3:1 above 24px. Check `--ink-soft` on
  `--paper` and `--phone-soft` on `--oled` specifically.
- Never use color alone to carry meaning. The left-behind state must be
  readable from the text and the shape, not only from the red.

---

## Code conventions

- **Svelte 5 runes only.** `$state`, `$derived`, `$props`, `onclick`. Never
  `export let`, `$:`, or `on:click`. The two syntaxes cannot be mixed in one
  component, and the whole project is runes.
- **All application state lives in `App.svelte`.** Child components receive
  state as props and receive callback functions as props to change it. Do not
  introduce stores, context, or local state that duplicates parent state. The
  one exception is purely visual local state, such as which phone tab is open,
  which may live in the phone component.
- Plain JavaScript, not TypeScript.
- Comments explain *why*, not *what*. Jonathan has to defend this code in a
  presentation, so comment the design reasoning at the top of each component.
- One component per file, named in PascalCase, in `src/lib/`.
- No new dependencies without asking. Vanilla Svelte and CSS can do all of
  this. Charts, if any, are hand-written inline SVG.

---

## Build stepwise

Work one phase at a time from `SPECIFICATIONS.md`. After each phase:

1. Run `npm run build` and confirm it compiles with no warnings.
2. Run the dev server and manually verify the phase's acceptance checks.
3. Stop and report what you did before starting the next phase.

Do not build three phases at once. If a phase turns out to need a decision
that is not in the spec, stop and ask rather than guessing.

---

## When to stop and ask Jonathan

`SPECIFICATIONS.md` marks specific points with **PAUSE AND ASK JONATHAN**.
Beyond those, stop and ask whenever:

- A requirement in the spec contradicts something already built
- A design choice would be hard to reverse later
- You are about to invent content that should be real, such as interview
  findings, his course name, or a URL
- Something would take significantly longer than the phase suggests

Ask one clear question with two or three concrete options, not an open-ended
one. He is a beginner with Svelte, so frame options by what the user sees,
not by implementation detail.
