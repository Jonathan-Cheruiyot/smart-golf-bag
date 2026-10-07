# Smart Golf Bag: Build Guide

Project 1, Interface to a Smart Object. Jonathan Cheruiyot.

This guide covers getting the code running in VS Code, what each file does,
how the code works, how it maps onto the Level 0 to Level 4 requirements, and
what is left to finish the project.

---

## 1. What is already done

**Level 0 is complete**, apart from one link. The page has the two devices on
a centre stage, a header strip with the project title, your name, a write-up
link placeholder, an Info button and the placement figure (Fig. 1), and a
test bar along the bottom with the simulation controls.

**Level 1 is complete.** The bag has three physical keys and four indicators
on an e-ink screen, built around one core feature: detecting when a club has
been left behind.

**Levels 2 to 4: two options are complete.**

- Option 2, a mock secondary device: the phone, with Round, Setup and Summary
  tabs, synced with the bag in both directions.
- Option 4, a simulation over time: Play round runs a scripted nine-hole
  round that drives both devices.

**Not done yet:** deploying to Vercel, the real write-up URL (the link is
still `#writeup`), the design section done by hand, the user evaluation, the
documentation page, and the demo video. Sections 8 and 9 cover those.

---

## 2. Setup in VS Code

The project is already a working Vite + Svelte project and is on GitHub at
https://github.com/Jonathan-Cheruiyot/smart-golf-bag.

### Running it on this machine

Open the project folder in VS Code with File > Open Folder, open a terminal,
and run:

```
npm run dev
```

That prints a URL like `http://localhost:5173/`. Open it in Chrome. You
should see the bag display on the left and the phone on the right.

### Getting it onto another machine

```
git clone https://github.com/Jonathan-Cheruiyot/smart-golf-bag.git
cd smart-golf-bag
npm install
npm run dev
```

`npm install` is needed once after cloning, because `node_modules` is not
stored in git.

### Avoid OneDrive

Keep the project out of OneDrive. OneDrive tries to sync every one of the
tens of thousands of files in `node_modules` as npm writes them, which makes
installs crawl and sometimes fail outright. GitHub is the backup.

### Install the VS Code extension

Search the Extensions panel for **Svelte for VS Code** and install it. You
get syntax highlighting and error checking in `.svelte` files.

---

## 3. File map

```
smart-golf-bag/
├── index.html              Entry page. Loads the IBM Plex fonts.
├── package.json            Dependencies and the dev/build/preview scripts.
├── vite.config.js          Vite setup. You will not edit this.
├── svelte.config.js        Svelte setup. You will not edit this.
├── CLAUDE.md               Design direction and code rules.
├── SPECIFICATIONS.md       The build phases, and every decision made.
├── MOTION.md               The rules for animation.
├── REQUIREMENTS.md         The assignment as a checklist.
├── QUESTIONNAIRE.md        The user evaluation protocol.
├── GUIDE.md                This file.
└── src/
    ├── main.js             Boots the app. Do not edit.
    ├── app.css             Design tokens (colours, fonts, spacing, motion)
    │                       and the few styles shared across components.
    ├── App.svelte          Root component. Holds ALL state and the page layout.
    └── lib/
        ├── BagMount.svelte       The bag's pocket and its three keys.
        ├── BagDisplay.svelte     The e-ink screen (Level 1).
        ├── ClubRack.svelte       The grid of club slots on that screen.
        ├── PhoneShell.svelte     The phone body, status bar and tabs.
        ├── PhoneRound.svelte     Phone tab: the live round.
        ├── PhoneSetup.svelte     Phone tab: choose today's 14 clubs.
        ├── PhoneSummary.svelte   Phone tab: shots chart and clubs used.
        ├── PairingLine.svelte    The dashed line between the devices.
        ├── TestPanel.svelte      The simulated sensors in the bottom bar.
        ├── SimProgress.svelte    Simulation progress bar and caption.
        ├── BagGraphic.svelte     Placement drawing (Level 0 requirement).
        ├── WorkbenchLabel.svelte   The small labels above each device.
        └── WorkbenchOverlay.svelte The pop-up sheet for Info and Fig. 1.
```

**The important structural idea:** all state lives in `App.svelte`, not in
the child components. The test bar, the bag and the phone all need to touch
the same data, so it lives in their shared parent and flows down as props.
This is worth being able to explain in your presentation, because it is the
main architectural decision in the project. It is also why the two devices
can never disagree: they are both drawing the same data.

Each component starts with a comment explaining why it is designed the way
it is. Those comments are written to be repeated out loud.

---

## 4. How the code works

The project uses Svelte 5. Note that the class tutorial PDF references the
older Svelte 4 REPL, so if you see `export let` and `$:` and `on:click` in
course materials, that is Svelte 4 syntax. The two styles cannot be mixed
inside one component. Everything here is consistent Svelte 5.

### The pieces of Svelte you need to understand

**`$state`** makes a value reactive. When it changes, anything displaying it
redraws.

```js
let round = $state({ active: false, hole: 1, strokes: 0, log: [], toPin: null });
```

**`$props`** receives values from a parent component.

```js
let { clubs, hole } = $props();
```

**`$derived`** computes a value from other state. It recalculates on its own
whenever its inputs change, so you never update it manually.

```js
let loadedClubs = $derived(clubs.filter((c) => c.loaded));
let clubsOut = $derived(loadedClubs.filter((c) => !c.inBag));
```

**`{#each}` and `{#if}`** loop and branch in the markup.

```svelte
{#each clubs as club (club.id)}
  <li class="slot">{club.short}</li>
{/each}

{#if clubsOut.length === 0}
  <p class="empty">None. Every club is in its slot.</p>
{/if}
```

The `(club.id)` tells Svelte which item is which, so that when the list
changes it moves the right elements instead of redrawing all of them.

**`$effect`** runs code when state changes, for things that are not just
drawing. It is used sparingly: for the 200ms relay of an alert from the bag
to the phone, for the count's flash on the e-ink screen, and for the phone's
clock.

### Passing functions down

`BagMount.svelte` has the bag's buttons but does not own the data they
change. So `App.svelte` passes functions down as props, and the child calls
them (shortened here to the one button):

```svelte
<!-- In App.svelte -->
<BagMount {round} onNextHole={nextHole} />
```

```svelte
<!-- In BagMount.svelte -->
<button onclick={onNextHole}>Next Hole</button>
```

`{round}` is shorthand for `round={round}`.

The phone's Round tab has the same three buttons and is given the same
functions. That is the whole of "either device can control it": two sets of
buttons, one function.

### The left-behind logic

This is the heart of the project, and it is the thing worth explaining in
your demo. Each club record carries a `pulledAtHole` field:

```js
{
  id: "i7",
  name: "7 Iron",
  short: "7i",
  category: "iron",
  loaded: true,        // is it in today's 14
  inBag: true,         // is it in its slot right now
  pulledAtHole: null   // the hole it was taken out on
}
```

When a club is pulled, the bag records which hole it happened on. A club is
only treated as lost if it was pulled on an **earlier** hole than the current
one:

```js
let leftBehind = $derived(clubsOut.filter((c) => c.pulledAtHole < round.hole));
```

That distinction is the whole design idea. A club in your hand on the current
hole is normal and should not trigger an alarm. The same club still missing
after you have walked to the next tee means you left it on the green.

### The second alert

The bag itself can be left behind too, and only the phone can say so,
because nobody is standing at the bag to read its screen:

```js
let bagAbandoned = $derived(bag.distanceFromGolfer > 30 && round.active);
```

This one line is the answer to "why is there a second device".

---

## 5. Level 0 requirements, mapped to the code

| Requirement | Where it is |
|---|---|
| Page split into regions | `App.svelte`: `<header class="strip">` for project info, `<section class="stage">` for the two devices, `<section class="testbar">` for testing |
| Project title and your name | `<h1>` and `.byline` in the header strip |
| Link to the write-up | The `<a href="#writeup">` tag. **Replace `#writeup` with your real URL.** |
| Graphic showing where the UI sits | `BagGraphic.svelte`. The small Fig. 1 button in the header opens the full drawing, with six numbered callouts |
| Info button explaining the controls | The Info button, which opens a `WorkbenchOverlay` |
| Buttons to simulate use | `TestPanel.svelte`: one button per club to lift or return it, the Walk away slider, Play round, Reset |
| Space reserved for future goals | Reserved in the first layout, since filled by the phone and the simulation. There is no empty box on the page now |

The project also requires that the interface not be confined to one flat
surface. `BagGraphic.svelte` is what shows this: the display is set into the
side pocket with the Start/End Round key below it, the Next Hole and Alerts
keys are on the strap edge, the sensors and lights are in the top cuff, a
stand sensor is in the base, and the phone travels with the golfer. Call this
out explicitly in your write-up and your presentation.

---

## 6. Level 1: controls, indicators, and design goals

Use these tables directly in your documentation. The project asks for exactly
this mapping.

### Controls

| Control | What it does | Design goal it serves |
|---|---|---|
| Start / End Round (key below the screen) | Begins tracking, resets hole and shot count | The bag should not nag during practice or in the trunk |
| Next Hole (key on the strap edge) | Advances the hole, which triggers the left-behind check | The golfer, not the bag, decides when a hole is over |
| Alerts On / Off (key on the strap edge) | Silences the warnings on both devices | Users need to mute alerts at the range |

All three are physical keys in the mount, not touch targets on the screen:
e-ink touch is slow, golfers wear gloves, and the bag is used in rain.

### Indicators

| Indicator | What it shows | Design goal it serves |
|---|---|---|
| Club count, large | How many of today's clubs are in the bag | Needs to be readable at a glance in sun glare |
| Club rack | Which specific club is out | Knowing one is missing is useless without knowing which |
| Message line, with the left-behind alert | Club name plus the hole it was left on | Tells the golfer where to walk back to |
| Hole and round state | Round context | Low-priority, so it is small and at the top |

### Design choices worth defending

- The club count is the largest element because it answers the most common
  question in the least time.
- A club in hand gets a neutral message, not a red alert. Alarming on normal
  use would train the golfer to ignore the alert entirely.
- The alert names the hole, not just the club, because the useful information
  is where to go, not what is gone.
- Nothing that takes more than a two-second glance is on the bag. The shot
  count, the club setup and the round history are on the phone.
- The bag looks like e-ink and the phone looks like a phone on purpose. They
  are different objects in different situations, and they move differently
  too: the bag snaps, the phone slides.
- A left-behind club is never shown by colour alone. Its rack slot gets an
  outline and a corner notch, so it reads in grayscale.
- Buttons are at least 44px, since golfers wear gloves.

---

## 7. Levels 2 to 4: what was built

The project asks for 1 to 3 options from the Levels 2 to 4 list. Two are
built.

### Option 2: the phone, a mock secondary device

The write-up has to say why the second device exists and how the two sync.

**Why it exists.** Golfers lose things at two scales. A club left on the
green is caught by the bag, because the golfer is standing right there. The
bag itself left at the turn can only be caught by the phone, because the
golfer has already walked away.

**How they sync, in both directions.**

- Phone to bag: ticking a club on or off in the Setup tab redraws the bag's
  rack at once.
- Bag to phone: when the bag raises a left-behind alert, the phone's
  notification follows 200ms later, and the line between them flashes.
- Either way: Start/End Round, Next Hole and Alerts work from the bag's keys
  or the phone's Round tab, and both show the result.

### Option 4: simulate a round over time

Play round steps through a scripted nine holes in 72 steps, one every 0.8
seconds, about 57 seconds in all. It is a 6 handicap going out in 38 on a
par 36, and the caption narrates each shot. The sand wedge is left by the
green on hole 3 so the alert fires on its own on hole 4; the putter is left
on hole 7 after a birdie; and after hole 9 he walks to the clubhouse without
the bag, so the phone's second alert fires too.

Each full shot uses the club whose distance matches the yardage left, from
the 5 handicap column of a published distance chart.

**Strokes and club uses are different numbers.** The round is 38 strokes but
only 30 club uses, because a two-putt is two strokes and one use of the
putter. The bag senses a club coming back, not a swing, so each step of the
script declares how many strokes it represents. The phone shows strokes as
the headline and "Club uses" as a separate stat.

**Distance is split between the devices on purpose.** The bag's header
prints the hole's par and yardage, which is fixed course data. The yards
left TO PIN are on the phone only, because the phone travels with the golfer
and the bag is back on the cart path and cannot know where the ball is.

The script is a list of steps in `App.svelte`. Each step calls the same
functions the buttons call. A shot is two steps with the same caption, club
out and then club back:

```js
// id, yards left to the pin afterwards, caption, strokes
const shot = (id, toPin, caption, strokes = 1) => [
  { caption, strokes: 0, run: () => pullClub(id) },
  { caption, strokes, toPin, run: (step) => returnClub(id, step.strokes, step.toPin) }
];

const SCRIPT = [
  // ...the round starts...
  ...shot("dr", 126, tee(2) + "Driver, 263. 126 to the pin."),
  ...shot("pw", 20 * FT, "Pitching wedge, 126, on the green. 20 feet to the pin."),
  ...shot("pt", 0, "Putter, two putts from 20 feet. Par.", 2),
  ...walk("Level par through 2. Walk to hole 3."),
  // ...and so on
];

function simTick() {
  const step = SCRIPT[sim.step];
  step.run(step);
  sim.step += 1;
  if (sim.step >= SCRIPT.length) stopSim();
}

function playSim() {
  // ...reset the stage first...
  sim.running = true;
  simTick();
  simTimer = setInterval(simTick, STEP_MS);
}

function stopSim() {
  clearInterval(simTimer);
  simTimer = null;
  sim.running = false;
}
```

`setInterval` runs a function repeatedly on a timer, and `clearInterval`
stops it. The point to make in the presentation: there is no separate
"simulation mode". The script presses the same buttons you would, so what it
shows is what the real interface does.

---

## 8. GitHub and hosting

The project requires both a source code link and a publicly hosted app link.

### GitHub

The repo is already set up and pushed:
https://github.com/Jonathan-Cheruiyot/smart-golf-bag

To save and upload later changes:

```
git add .
git commit -m "Describe what changed"
git push
```

The `.gitignore` file keeps `node_modules` and `dist` out of the repo, so do
not delete it.

### Deploy the public app (still to do)

The simplest route for a Vite project:

1. Go to vercel.com and sign in with GitHub.
2. Import the `smart-golf-bag` repo.
3. Accept the defaults. Vercel detects Vite automatically.
4. You get a public URL in about a minute.

After that, every `git push` redeploys the site by itself.

Netlify works the same way. Either is fine. GitHub Pages also works but needs
a `base` setting in `vite.config.js`, so it is more fiddly.

### The write-up link (still to do)

Once the write-up is live, open `src/App.svelte`, find
`<a href="#writeup">`, replace `#writeup` with the real URL, then commit and
push.

---

## 9. Documentation checklist

The write-up is 20% of the grade and has to be public on your portfolio page.
It needs:

- [ ] Description of the project
- [ ] Your design work: affordances, interviews with 3 people, smart feature
      assumptions, user needs and design requirements, 10-plus-10 sketches,
      vanilla sketch, storyboard, hybrid sketch, and the feedback you
      collected from 3 people
- [ ] Detailed description of the interface, features, and controls
- [ ] Plenty of screenshots showing different actions
- [ ] How you implemented it: Svelte 5 with Vite and no other libraries,
      the component structure in section 3, state in the parent and props
      flowing down
- [ ] Future work, including anything you did not finish
- [ ] AI documentation: how you used AI, written by you and honest
- [ ] A 2 to 3 minute demo video with voiceover, linked from the page
- [ ] Link to the GitHub repo
- [ ] Link to the hosted app

### Screenshots worth taking

1. Clean state, both devices, 14 of 14
2. Club in hand: bag neutral, phone showing which club and which hole
3. Left behind: red alert on both devices
4. Phone Setup tab mid-edit, showing the 14-club limit being enforced
5. Away-from-bag alert: phone only, bag calm
6. Phone Summary tab with the round chart
7. Fig. 1, the placement graphic, open
8. Info overlay open

For 5 and 6, Play round gets you there: stop it on step 70 of 72 for the
away alert, or let it finish and open the Summary tab.

### Demo video outline

Open with the project name and your name. Explain the problem in one
sentence: golfers leave clubs behind and have no way to know until holes
later. Show the two devices and the test bar and say what each is for. Lift
a club and press Next Hole so the alert fires live and travels to the phone.
Open Fig. 1 and explain the surfaces the interface is spread across. Press
Play round and let it run. Close with what you would build next.

### The reduced-motion test

Worth one sentence in the write-up, because most projects skip it. On
Windows: Settings, Accessibility, Visual effects, turn off Animation effects.
Reload the page and confirm everything still works and nothing disappears.

---

## 10. Troubleshooting

**Page is blank, or a red error overlay appears.** Read the message in the
terminal where `npm run dev` is running. It usually names the file and line.

**"Failed to resolve import ./lib/BagDisplay.svelte"** (or any other
component) The file is in the wrong folder or has the wrong name. Components
must be in `src/lib`, with the exact names in section 3.

**npm install hangs for many minutes.** The project is probably inside
OneDrive. Move it somewhere else and run `npm install` again.

**"Could not read package.json"** Your terminal is in the parent folder, not
the project folder. `cd` into the folder that contains `package.json`.

**Changes do not show up.** Check that `npm run dev` is still running and
that you saved the file. A hard refresh with Ctrl+Shift+R clears a stale
cache.

**Port already in use.** An old dev server is still running. Vite will offer
a different port, or you can close the other terminal.

**The simulation's story goes wrong.** Something was pressed on the bag or
the phone while it was playing. Press Reset, then Play round again.
