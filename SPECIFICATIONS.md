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
├── BagMount.svelte          the pocket panel the display is set into, drawn
│   │                        as a product illustration, with the three keys
│   └── BagDisplay.svelte    the e-ink panel (Level 1), indicators only
│       └── ClubRack.svelte  the 14-slot grid
├── PhoneShell.svelte        phone body, status bar, tab switching
│   ├── PhoneRound.svelte    live round view, mirrors bag state
│   ├── PhoneSetup.svelte    choose which 14 clubs are loaded
│   └── PhoneSummary.svelte  post-round stats, hand-drawn SVG
├── PairingLine.svelte       dashed line between the devices, flashes on relay
├── SimProgress.svelte       simulation progress bar and step caption
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
    pulledAtHole: null       // hole number it was taken out on
  }
]

round = {
  active: false,
  hole: 1,                   // 1 to 18
  strokes: 0,                // the score so far, as a scorecard counts it
  log: [],                   // { hole, clubId, strokes } appended on each return
  toPin: null                // yards to the pin, or null if unknown. Phone only
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
1. `PhoneShell.svelte`: 296 by 640 (close to 19.5:9), drawn as a solid
   generic smartphone at the same fidelity as the bag mount. Revised in
   Phase 7; the original 300 by 620 outline with a minimal status strip is
   superseded.
   - Housing in flat tone steps, outside in: a `--phone-rule` metal edge
     band, a light chamfer hairline, the dark channel the glass sits in, a
     thin uniform `--oled-raised` bezel, then the `--oled` screen. No
     gradients, no gloss.
   - Concentric corners: each layer's radius is the screen's 30px plus the
     thickness outside it (40, 36, 34, 30).
   - Side keys that break the silhouette (volume up and down on the left, a
     longer key on the right), a hatched earpiece slot, a centred circular
     camera, and a light gesture bar at the bottom of the screen.
   - **The status bar is generic and drawn in our own palette**: the clock
     and the bag pairing indicator on the left, signal, wifi and battery on
     the right as inline stroke SVG. No one maker's design language: no
     branded cutout, typeface, control styling or accent colour. The app
     inside stays in the project's own design language.
2. Three tabs along the bottom: Round, Setup, Summary. Mono labels, a 2px
   `--flag` rule under the active one.
3. `PhoneRound.svelte` first: mirrors the bag's club count and alert, and adds
   what the bag cannot show, such as the list of clubs out with the hole each
   was pulled on, and a running shot count.
4. A notification band at the top of the phone that appears when `leftBehind`
   is non-empty, worded for someone who is not looking at the bag. It follows
   the shared alert preference: with alerts off there is no band, though the
   clubs-out list still marks the club as left behind.
5. `phone` state (`paired`, `tab`, `notifications`) lives in `App.svelte`.
   Setup and Summary are placeholder text until Phases 4 and 7.
6. Motion, from `MOTION.md`: tabs slide in the direction navigated (240ms in,
   140ms out), the band slides down and fades in and leaves by fading only,
   and the phone's count settles with a spring. All three check
   `prefers-reduced-motion` in JavaScript.

**Acceptance:** pulling a club on the test panel updates both devices at
once. The phone shows strictly more detail than the bag.

> **Decision 2: phone personality — RESOLVED**
>
> Jonathan confirmed the split, then widened it in Phase 4. The Round tab is
> a **companion** that also repeats the bag's three controls (Start/End
> Round, Next Hole, Alerts): either device can trigger them and both reflect
> the result. The Setup tab is a **remote control** for which clubs are
> loaded. Club loading and round history stay phone-only.

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

6. The three bag controls are mirrored on the phone's Round tab and call the
   same functions in `App.svelte`. The alert preference is changed there (and
   on the bag), not on the Setup tab, so the phone has one place for it. Alerts
   off silences both the left-behind and the away-from-bag notification.
7. A loaded club that is currently out of the bag cannot be deselected, and
   the Setup tab says why, so a club can never vanish from the rack mid-shot.
8. The test panel gains a "Walk away" slider, 0 to 100 m, for
   `bag.distanceFromGolfer`. Its "Advance hole" button is removed, since Next
   Hole now exists on both devices.
9. Motion, from `MOTION.md`: `animate:flip` on the Round tab's clubs-out
   list, and the cross-device relay. Jonathan chose the 200ms relay: the bag's
   alert appears first and the phone notification follows 200ms later, staged
   by one effect in `App.svelte` from the single `leftBehind` change. The
   away alert is not delayed, because the phone measures distance itself. The
   pairing-line flash between the two waits for Phase 6, when the line exists.

**Acceptance:** deselect a club on the phone and its rack slot disappears
from the bag; walk away on the test panel and only the phone reacts.

> **Decision 3: his real club set — RESOLVED**
>
> A standard 17-club set with no brands: driver, 3 and 5 wood, 3, 4 and 5
> hybrid, 4 through 9 iron, pitching, gap and sand wedge, 60° wedge, putter.
> The lob wedge was dropped because it is the same club as the 60° wedge.
> The day starts with 14 loaded; the 4 and 5 hybrid and the 60° wedge start
> at home.

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

5. As built (revised after Phase 8): 72 steps at 0.8 seconds, about 57
   seconds. It is a 6 handicap going out in 38 on a par 36 at Tiger Village
   Golf Course, and each caption narrates one shot, for example "Hole 3, 364
   yds, par 4 — Driver, pushed right". Every full shot uses the club whose
   stock distance matches the yardage left, from a 5 handicap distance chart.
   - A shot is two steps with the same caption, club out then club back, so
     each one stays on screen for 1.6 seconds.
   - The sand wedge is left by the green on hole 3; the alert fires on hole
     4, he chips with the gap wedge instead, and collects it before hole 5.
   - The putter is left on the green on hole 7 after a birdie; the alert
     fires on hole 8 and it is returned before he putts.
   - After hole 9 he walks to the clubhouse without the bag, which fires the
     away-from-bag alert, then walks back and the round ends.
   - The hole layout is the one Jonathan supplied: pars 5 4 4 3 4 4 5 3 4,
     yards 485 389 364 195 390 352 510 178 372, par 36 and 3,235 yards. It
     lives in one `HOLES` list in `App.svelte`.
   - **Strokes, not club uses.** Each step declares how many strokes it
     represents, and `returnClub(id, strokes)` logs them when the club goes
     back in the bag. A two-putt is two strokes and one use of the putter. A
     manual return on the test panel counts as one stroke. Strokes by hole
     are 5 4 4 4 4 5 4 3 5, which is 38; the round has 30 club uses. The
     phone's Summary tab, its chart, the Round tab and the test bar all read
     the same `round.strokes` and `round.log`.
   - **Distance.** The bag's header prints the hole's par and yardage from
     the course card, for example "HOLE 04  PAR 3  195 YDS": static data,
     not shot tracking. The phone's Round tab has a TO PIN line that updates
     after each shot, in yards from the fairway and feet near the green. It
     is on the phone because the phone travels with the golfer; the bag sits
     on the cart path and cannot know where the ball is. Each script step
     declares where it leaves the golfer, and each caption gives the distance
     hit and the distance left, for example "Hole 1, 485 yds, par 5 -
     Driver, 261. 224 to the pin." A manual return on the test panel has no
     position, so TO PIN then reads "--" until the next tee.
   - A stroke on a club that was left behind is counted when the club comes
     back, so the running total is one behind between the 3rd green and the
     5th tee, and between the 7th green and the 8th green. It is credited to
     the hole the club was pulled on.
   - The original script (21 steps at 1.2 seconds, a 7 Iron left on hole 3)
     is superseded.
6. Play first resets the bag, loads the standard 14, turns alerts on and
   opens the phone's Round tab, so the script always tells the same story.
   The manual sensors, slider and Reset are disabled while it plays.
7. `SimProgress.svelte` is the progress indicator: a bar drawn over the test
   bar's top rule, plus a one-line caption above it naming the step. Per
   `MOTION.md` the bar tweens, using `transform`.
8. The phone's Round tab was tightened so that three clubs out, both alerts
   and the controls fit without scrolling.

**Acceptance:** press play, walk away from the keyboard, and both devices
tell a complete story including both alert types.

> **Decision 4: simulation pacing — RESOLVED**
>
> No speed control. First set at 1.2 seconds per step for a 25 second run;
> revised to 0.8 seconds per step when the script became a shot-by-shot
> round, for a run of about 57 seconds.

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

5. A second pairing line, `PairingLine.svelte`, sits on the stage across the
   gap between the bag display and the phone. Jonathan chose this so that the
   relay flash from `MOTION.md` is visible: the bag's alert appears, the line
   turns solid red for the 200ms relay, then the phone's notification
   arrives.
6. Motion, from `MOTION.md`: both pairing lines draw themselves on once (the
   stage line after the load reveal, the figure's line when the overlay
   opens) and then drift. `MOTION.md` writes the drift with `linear` but also
   bans `linear`; the drift uses `--e-draft`, which reads as a slow pulse.

**Acceptance:** a reader who has never seen the project can tell where each
part of the interface physically lives.

---

### Phase 7: Phone summary and polish

**Goal.** Finish the phone and clean up.

**Do:**
1. `PhoneSummary.svelte`: post-round stats from `round.log`, drawn as
   hand-written inline SVG. Shots per hole as a small column chart in
   `--phone-text` on the phone's dark ground (the spec first said `--ink`,
   which is near-black and would be invisible there). The chart and the
   headline number count strokes; club uses is a separate, labelled stat,
   and the list of clubs used gives both for each club.
   No chart library. With nothing logged it says so; it never shows invented
   numbers.
2. Accessibility pass against the checklist in `CLAUDE.md`. Specifically
   verify `--ink-soft` on `--paper` and `--phone-soft` on `--oled` meet
   contrast, and darken them if not.
3. Keyboard pass: Tab through every control in order, confirm visible focus
   rings that are not the browser default blue.
4. Fill in the Info button content so it explains every test control.
5. Remove dead code and any leftover template files. Done:
   `ClubTracker.svelte`, the first prototype's panel, is deleted.

6. **Addition: the bag display is mounted in the bag.** `BagMount.svelte`
   draws the side pocket around the display as a technical product
   illustration, in the manner of a patent drawing: 576 by 620, the same
   height as the phone.
   - Depth from four flat tones only: `--paper-sunk` for the bag wall,
     `--paper` for the pocket panel and strap sewn onto it, `--ink-soft` for
     recesses, `--bezel` for the display housing. No gradients, shadows,
     bitmap textures or fabric grain.
   - The display drops into a channel with a heavier shadow line on its top
     and left edges. Stitches are dashed hairlines along the seams, the
     zipper is a regular hatch, the strap has a box stitch and a rivet at
     each end, and parting lines mark where parts meet.
   - **The three controls move off the e-ink screen** and become physical
     keys in the mount: Start/End Round below the screen, Next Hole and
     Alerts on the strap edge. Each is a real `<button>` drawn as a cap
     sitting 4px above a base, which travels 1px down on `:active`. The
     reason, to repeat out loud: e-ink touch is slow, golfers wear gloves,
     and the bag is used in rain. This supersedes item 2 of Phase 2, which
     put the controls on the screen.
   - The e-ink screen then holds only indicators: hole, round state, club
     count, message line, club rack.
   - Fig. 1 has six callouts to match: display, round key, strap edge keys,
     top cuff, base, phone.

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

> **Decision 5: URLs — PARTLY RESOLVED**
>
> GitHub repo: https://github.com/Jonathan-Cheruiyot/smart-golf-bag, pushed.
> The write-up URL does not exist yet, so step 5 is open: `href="#writeup"`
> stays in `App.svelte` as an honest placeholder and Jonathan replaces it in
> a final commit once the write-up is live. Step 4, the Vercel deploy, is
> also his to do, since it needs his Vercel account.

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
2. ~~**Phone personality**, companion versus remote control (Phase 3)~~
   Resolved, see Phase 3
3. ~~**His real club set**, or use a standard 18 (Phase 4)~~ Resolved, see
   Phase 4
4. ~~**Simulation pacing**, and whether to add a speed control (Phase 5)~~
   Resolved, see Phase 5
5. **GitHub and write-up URLs** (Phase 8). GitHub resolved; the write-up
   URL is still to come

Two more that are not blocking but improve the result if he answers early:

6. Did his user interviews surface a need the current feature set misses? If
   so it may belong on the phone rather than the bag.
7. Does he want the bag display to show a hole number from a real course
   layout, or stay generic 1 through 18?
