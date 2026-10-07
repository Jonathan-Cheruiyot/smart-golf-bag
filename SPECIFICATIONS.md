# SPECIFICATIONS.md

Build specification for the Smart Golf Bag interface.
Read `CLAUDE.md` first for design direction and code conventions.

This is a **living specification**. When a phase forces a change in approach,
update this file in the same commit rather than letting it drift from the
code.

---

## 1. The product idea

A golf bag that knows which clubs are in it, paired with a phone app that
knows where the bag is.

### The two-tier problem it solves

Golfers lose things in two different ways, at two different scales, and the
two need different interfaces:

| What gets left | Where it goes wrong | Which device catches it |
|---|---|---|
| A club | Left on the green, discovered holes later | **Bag screen**, because the golfer is standing right there |
| The bag itself | Left at the turn, the clubhouse, or the range | **Phone**, because the golfer has already walked away |

This pairing is the reason the project has a second device, and it is the
answer to the write-up question "why have a secondary device?" The bag can
only alert someone who is near it. The phone travels with the golfer.

### Division of labor

**Bag screen** is glanceable only. Club count, which club is out, alerts,
current hole. Three physical controls.

**Phone** handles everything requiring attention: which 14 clubs are loaded
today, alert preferences, round history, and the away-from-bag alert.

Governing rule, repeated from `CLAUDE.md`: nothing needing more than a
two-second glance goes on the bag screen.

---

## 2. Course requirements this satisfies

| Requirement | How |
|---|---|
| Level 0 | Device region and testing region, title, name, write-up link, placement graphic, info button, simulation buttons |
| Level 1 | Club presence tracking, three bag controls, four indicators, left-behind alert |
| Level 2, Option 2 | Mock secondary device with two-way sync |
| Level 3, Option 4 | Scripted round simulation driving both devices over time |
| Not one flat surface | Display on side pocket, buttons on strap, sensors in top cuff, stand sensor in base |

Two options from the Levels 2 to 4 list, which is within the "choose 1 to 3"
allowance.

---

## 3. Architecture

### Component tree

```
App.svelte                  owns ALL state, renders header strip, stage, test bar
├── WorkbenchLabel.svelte    small annotation label used around the page
├── WorkbenchOverlay.svelte  dialog sheet for the placement figure and the info text
├── BagDisplay.svelte        the e-ink panel (Level 1)
│   └── ClubRack.svelte      the 14-slot grid
├── PhoneShell.svelte        phone body, status bar, tab switching
│   ├── PhoneRound.svelte    live round view, mirrors bag state
│   ├── PhoneSetup.svelte    choose which 14 clubs are loaded
│   └── PhoneSummary.svelte  post-round stats, hand-drawn SVG
├── BagGraphic.svelte        placement drawing, four numbered zones; `compact`
│                            for the header figure, full size in the overlay
└── TestPanel.svelte         all simulation controls, in the bottom bar
```

### State model

All of this lives in `App.svelte` as `$state`. Do not scatter it.

```js
// One entry per club slot. `pulledAtHole` is what makes left-behind
// detection possible: a club in hand on the CURRENT hole is normal,
// the same club still out on a LATER hole was left behind.
clubs = [
  {
    id: 'i7',
    name: '7 Iron',
    short: '7i',
    category: 'iron',        // wood | hybrid | iron | wedge | putter
    loaded: true,            // is it in today's 14, set from the phone
    inBag: true,             // is it physically in its slot right now
    pulledAtHole: null,      // hole number it was taken out on
    usedOnHoles: []          // for the phone summary
  }
]

round = {
  active: false,
  hole: 1,                   // 1 to 18
  shots: 0,
  log: []                    // { hole, clubId } appended on each return
}

bag = {
  alertsOn: true,
  distanceFromGolfer: 0      // metres, driven by the test panel
}

phone = {
  paired: true,
  tab: 'round',              // round | setup | summary
  notifications: []          // { id, kind, text, hole }
}

sim = {
  running: false,
  step: 0
}
```

### Derived values

```js
loadedClubs   = clubs.filter(c => c.loaded)
clubsOut      = loadedClubs.filter(c => !c.inBag)
inBagCount    = loadedClubs.length - clubsOut.length
leftBehind    = clubsOut.filter(c => c.pulledAtHole < round.hole)
bagAbandoned  = bag.distanceFromGolfer > 30 && round.active
```

`leftBehind` is the heart of the project. It is one line and it is the thing
to explain in the presentation.

---

## 4. Build phases

Work these in order. After each one: `npm run build`, verify the acceptance
checks in the browser, then stop and report before continuing.

---

### Phase 0: Foundation

**Goal.** Design tokens and fonts in place, project compiles, nothing visual
yet.

**Do:**
1. Confirm `npm run build` succeeds on the existing project.
2. Write all tokens from `CLAUDE.md` into `src/app.css` as CSS custom
   properties on `:root`.
3. Add the IBM Plex font link to `index.html` (Mono 400/500/600, Sans 400/600,
   Serif 600).
4. Set the page background to `--table` and add the hairline graph grid as a
   repeating background on `<body>`, 24px squares in `--grid`. It should be
   barely visible, a drafting surface, not a feature.

**Acceptance:** page is warm paper with a faint grid, fonts load, build is
clean.

---

### Phase 1: Page shell and layout

**Goal.** The page layout, empty, with a hierarchy in which the two devices
dominate and everything else serves them.

**Do:**
1. Rebuild `App.svelte` as a single page with three bands, top to bottom:
   a slim header strip, the centre stage, and a quiet bottom bar.
2. Build `WorkbenchLabel.svelte`: a small `IBM Plex Mono` uppercase label with
   a hairline leader rule, used to annotate each device like a technical
   drawing. This is what makes the page read as a design document rather than
   a web app.
3. **Header strip**, 72px, with a hairline rule under it: the title in
   `IBM Plex Serif`, Jonathan's name, the write-up link, the Info button, and
   the placement graphic as a small figure ("Fig. 1, Placement"). The figure
   is always visible and is a real `<button>` that opens the full drawing in
   an overlay. The Info button opens its text in the same kind of overlay.
4. Build `WorkbenchOverlay.svelte` for both: a native `<dialog>` opened with
   `showModal()`, so focus trapping, Escape to close, and focus return come
   from the browser. Square corners, hairline border, no shadow. It opens
   instantly, because `MOTION.md` gives the harness one reveal on load and
   nothing after.
5. **Centre stage**: two labelled slots side by side, centred, with wide empty
   margins either side: "BAG DISPLAY, side pocket" at 480 by 620 and "PHONE,
   paired device" at 300 by 620, 96px apart.
6. **Bottom bar**, for the test panel: a hairline rule above it, the label in
   11px Mono in `--table-soft`, and a 48px-high slot for the controls. It is
   deliberately quieter than the devices, because the test controls are demo
   scaffolding and not part of the product. It must stay visible at the same
   time as the devices, with no scrolling.
7. Fixed widths. The assignment explicitly says the UI does not need to be
   responsive, and a real bag display is a fixed size.
8. The staged page reveal from `MOTION.md`: header and labels, then the bag,
   then the phone, then the test bar, 80ms apart.

**Acceptance:** header strip, two labelled empty device slots, and test bar
all visible together, nothing overlapping, no scrollbars at 1440 by 900. The
placement figure opens and closes by mouse and by keyboard.

> **Decision 1: device arrangement — RESOLVED**
>
> Jonathan chose (a), bag display and phone side by side, then revised the
> hierarchy around it: the devices moved to the centre and grew, project info
> became the header strip, the placement graphic became a header figure with
> an overlay, and the test panel moved to the bottom bar. The layout above is
> the result.

---

### Phase 2: Bag display, the e-ink instrument

**Goal.** Level 1 complete, in the new visual language.

**Do:**
1. `BagDisplay.svelte`. A dark `--bezel` housing with a rounded outer edge,
   containing a square-cornered `--paper` screen.
2. Inside, top to bottom:
   - A hairline-ruled header strip: `HOLE 04` on the left, round state on the
     right, in Mono at 12px, letterspaced.
   - The club count as the dominant element. Target 72px, Mono 600, tabular
     figures, `--ink`. The "/ 14" sits small beside it in `--ink-soft`.
   - A single message line, which is the only element permitted to use
     `--flag`.
   - `ClubRack.svelte`: a 7 by 2 grid of slots. A present club is a filled
     `--ink` block with the short code reversed out. An absent club is an
     outline. A left-behind club is an outline in `--flag` **plus a filled
     corner notch**, so the state survives in grayscale and for colorblind
     viewers.
   - Three controls at the bottom: Start/End Round, Next Hole, Alerts.
     Square, hairline-ruled, 44px minimum, Mono labels.
3. Message precedence, highest first: left behind, club in hand, all present,
   idle. With alerts off, the alert line is suppressed but the rack still
   marks the slot, because the rack reports a fact and the alert is a nag.
4. `TestPanel.svelte`, first version, in the bottom bar: one toggle button per
   loaded club standing in for its slot sensor (press to lift out, press again
   to return), plus Advance hole and Reset. It is built here rather than later
   because this phase's acceptance check cannot be run without it. Later
   phases add the distance control (Phase 4) and the simulation (Phase 5).
5. `App.svelte` moves to the state model in section 3 for `clubs`, `round`
   and `bag`. `phone` and `sim` arrive with their own phases.
6. Motion, from `MOTION.md`: a 90ms inverted flash on the count when it
   changes (skipped under reduced motion), step easing on rack slot fills,
   and a 90ms step reveal on the alert line. Nothing else on the bag moves.

**Acceptance:** pull a club and the rack slot hollows out; advance a hole and
the slot gains the red notch and the alert line appears; the whole panel is
legible as a photocopy, with no color carrying meaning alone.

---

### Phase 3: Phone shell

**Goal.** The second device exists and mirrors the bag.

**Do:**
1. `PhoneShell.svelte`: 300 by 620, `--oled`, 36px corner radius, a thin
   `--phone-rule` bezel. A minimal status strip with a clock and a pairing
   indicator. **No fake iOS status bar with battery and signal icons**, which
   reads as a stock template.
2. Three tabs along the bottom: Round, Setup, Summary. Mono labels, a 2px
   `--flag` rule under the active one.
3. `PhoneRound.svelte` first: mirrors the bag's club count and alert, and adds
   what the bag cannot show, such as the list of clubs out with the hole each
   was pulled on, and a running shot count.
4. A notification band at the top of the phone that appears when `leftBehind`
   is non-empty, worded for someone who is not looking at the bag.

**Acceptance:** pulling a club on the test panel updates both devices at
once. The phone shows strictly more detail than the bag.

> **PAUSE AND ASK JONATHAN — Decision 2: phone personality**
>
> The phone can be either a **companion** that mirrors and explains, or a
> **remote control** that commands the bag. The spec assumes companion for
> the Round tab and remote for the Setup tab. Confirm that split, or pick one
> and apply it throughout. This changes what the Setup tab is allowed to do.

---

### Phase 4: Two-way sync

**Goal.** The write-up claim that interactions flow both directions is
actually true in the code.

**Do:**
1. `PhoneSetup.svelte`. The golfer picks which 14 clubs are loaded today from
   a longer list of around 18 owned clubs, since golfers swap a wood for a
   hybrid depending on the course. Enforce the 14-club USGA limit and show
   the count as `13 / 14 selected`. Over 14 is blocked with a clear reason,
   not a silent failure.
2. Changing the loaded set updates the bag's `ClubRack` immediately. This is
   phone to bag.
3. Alert preference lives on the phone, and the bag's Alerts button reflects
   it. Either device can change it, and both show the current value.
4. The bag's left-behind alert pushes a notification onto the phone. This is
   bag to phone.
5. **The away-from-bag alert**, the feature that justifies the second device:
   when `bagAbandoned` is true, the phone shows a distinct notification
   naming how far away the bag is. The bag screen shows nothing, because
   nobody is there to read it.

**Acceptance:** deselect a club on the phone and its rack slot disappears
from the bag; walk away on the test panel and only the phone reacts.

> **PAUSE AND ASK JONATHAN — Decision 3: his real club set**
>
> The list of about 18 owned clubs should be plausible. Ask what he actually
> carries, or whether to use a standard set. Do not invent brand names.

---

### Phase 5: Round simulation

**Goal.** Level 3, Option 4. A button plays a round and both devices update
live.

**Do:**
1. `sim` state plus a scripted sequence of roughly 20 steps covering nine
   holes: pull a club, return it, advance a hole, and on hole 3 deliberately
   fail to return the 7 Iron so the alert fires on its own.
2. Include one away-from-bag event, around the turn, so the phone alert also
   fires during the demo.
3. Roughly 1.2 seconds per step, with a visible progress indicator and a stop
   button. Use `setInterval` and always `clearInterval` when it finishes or is
   stopped.
4. The simulation must drive the **same functions** the manual buttons call,
   not a separate code path. Otherwise the demo proves nothing about the real
   interface.

**Acceptance:** press play, walk away from the keyboard, and both devices
tell a complete story including both alert types.

> **PAUSE AND ASK JONATHAN — Decision 4: simulation pacing**
>
> 1.2 seconds per step makes a nine-hole run about 25 seconds, which fits a
> 2 to 3 minute demo video. Confirm, or ask whether he wants a speed control.

---

### Phase 6: Placement graphic

**Goal.** The Level 0 requirement, and the proof the interface is not on one
flat surface.

**Do:**
1. Reuse the existing `BagGraphic.svelte` but restyle it to the drafting
   aesthetic: hairline `--table-ink` strokes on `--table`, no fills except
   the screen, numbered callouts in `--flag` with leader lines.
2. Four zones: side pocket display, strap buttons, top cuff sensors and slot
   lights, base stand sensor.
3. Add a fifth callout for the phone, drawn as a small outline beside the bag
   with a dashed pairing line to it, which visually makes the two-device
   argument.
4. The drawing appears in two places since the Phase 1 revision: `compact` in
   the header strip (bag only, 27 by 48, no callout text) and full size in
   the overlay. Both must be restyled, and the compact one must stay legible
   at that size.

**Acceptance:** a reader who has never seen the project can tell where each
part of the interface physically lives.

---

### Phase 7: Phone summary and polish

**Goal.** Finish the phone and clean up.

**Do:**
1. `PhoneSummary.svelte`: post-round stats from `round.log`, drawn as
   hand-written inline SVG. Shots per hole as a small column chart in `--ink`
   on the phone's dark ground, plus most-used club and total shots. No chart
   library.
2. Accessibility pass against the checklist in `CLAUDE.md`. Specifically
   verify `--ink-soft` on `--paper` and `--phone-soft` on `--oled` meet
   contrast, and darken them if not.
3. Keyboard pass: Tab through every control in order, confirm visible focus
   rings that are not the browser default blue.
4. Fill in the Info button content so it explains every test control.
5. Remove dead code and any leftover template files.

**Acceptance:** the page can be operated entirely by keyboard, and every
control has a visible focus state.

---

### Phase 8: Ship

**Goal.** Public links for the write-up.

**Do:**
1. `npm run build` clean, no warnings.
2. Confirm `.gitignore` excludes `node_modules` and `dist`.
3. Commit and push to GitHub.
4. Deploy to Vercel by importing the repo. Vite is detected automatically.
5. Replace the `href="#writeup"` placeholder in `App.svelte` with the real
   write-up URL.
6. Update `REQUIREMENTS.md` checkboxes to reflect what is now done.

> **PAUSE AND ASK JONATHAN — Decision 5: URLs**
>
> Ask for the GitHub repo URL and the write-up URL. Do not guess or invent
> them, and do not leave a fake URL in the code.

---

## 5. Screenshots to capture at the end

For the documentation, which requires plenty of screenshots showing different
actions:

1. Clean state, both devices, 14 of 14
2. Club in hand, bag neutral, phone showing which club and which hole
3. Left behind, red alert on both devices
4. Phone Setup tab mid-edit, showing the 14-club limit being enforced
5. Away-from-bag alert, phone only, bag calm
6. Phone Summary tab with the round chart
7. The placement graphic on its own

---

## 6. Things Claude Code must not do

- Do not invent interview findings, user quotes, or statistics. Those come
  from Jonathan's real interviews and go in the write-up, not the code.
- Do not add a backend, a database, or `localStorage`. Everything is
  in-memory, which is correct for a mock-up.
- Do not make the layout responsive. The assignment explicitly waives it.
- Do not add dependencies without asking.
- Do not write the AI documentation section of the write-up. Jonathan has to
  write that himself and it has to be honest.

---

## 7. Open questions to resolve with Jonathan

Collected from the phase markers above, in the order they come up:

1. ~~**Device arrangement** on the page (Phase 1)~~ Resolved, see Phase 1
2. **Phone personality**, companion versus remote control (Phase 3)
3. **His real club set**, or use a standard 18 (Phase 4)
4. **Simulation pacing**, and whether to add a speed control (Phase 5)
5. **GitHub and write-up URLs** (Phase 8)

Two more that are not blocking but improve the result if he answers early:

6. Did his user interviews surface a need the current feature set misses? If
   so it may belong on the phone rather than the bag.
7. Does he want the bag display to show a hole number from a real course
   layout, or stay generic 1 through 18?
