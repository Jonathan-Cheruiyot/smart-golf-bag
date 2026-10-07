# Smart Golf Bag: Build Guide

Project 1, Interface to a Smart Object. Jonathan Cheruiyot.

This guide covers getting the code running in VS Code, what each file does,
how the current code maps onto the Level 0 and Level 1 requirements, and what
is left to finish the project.

---

## 1. What is already done

**Level 0 is complete.** The page is split into a Device UI region and a
Testing UI region. It has the project title, your name, a write-up link
placeholder, a graphic showing where the interface sits on the bag, and an
Info button that explains the simulation controls.

**Level 1 is complete.** The device panel has three controls and four
indicators, built around one core feature: detecting when a club has been
left behind.

**Not done yet:** Levels 2 to 4, the write-up link, hosting, and the
documentation page. Section 7 onward covers those.

---

## 2. Setup in VS Code

### If you do not have a project folder yet

Open a PowerShell terminal in VS Code and run these one at a time:

```
mkdir C:\Users\cheru\dev
cd C:\Users\cheru\dev
npm init vite
```

At the prompts:

- Project name: `golf-bag`
- Framework: **Svelte**
- Variant: **JavaScript** (not TypeScript, not SvelteKit)

Then:

```
cd golf-bag
npm install
npm run dev
```

That prints a URL like `http://localhost:5173/`. Open it in Chrome. You
should see the default Vite counter page, which confirms the scaffold works
before you change anything.

### Drop in the project files

In VS Code, use File > Open Folder and pick `C:\Users\cheru\dev\golf-bag`.
Then:

1. Put `App.svelte` in `src/`, replacing the one already there.
2. Put every component file (`BagMount.svelte`, `BagDisplay.svelte`,
   `PhoneShell.svelte` and the rest listed in section 3) in `src/lib/`.
3. Put `app.css` in `src/`, replacing the one already there.
4. Delete `src/lib/Counter.svelte`.

If `npm run dev` is still running, the page reloads by itself. If you stopped
it, run it again.

### Two things that commonly go wrong

**Downloaded files get renamed.** If your browser saved them as
`App (1).svelte`, rename them to exactly `App.svelte`,
`BagDisplay.svelte` and so on, or the imports will fail.

**Wrong folder.** The component files must be in `src/lib`, not `src`,
because `App.svelte` imports them from `./lib/`.

### Avoid OneDrive

Keep the project out of OneDrive and out of Downloads. OneDrive tries to sync
every one of the tens of thousands of files in `node_modules` as npm writes
them, which makes installs crawl and sometimes fail outright. Use GitHub for
backup instead, which the project requires anyway.

### Install the VS Code extension

Search the Extensions panel for **Svelte for VS Code** and install it. You
get syntax highlighting and error checking in `.svelte` files.

---

## 3. File map

```
golf-bag/
├── index.html              Entry page, created by Vite. You will not edit this.
├── package.json            Dependencies and the dev/build/preview scripts.
├── src/
│   ├── main.js             Boots the app, created by Vite. Do not edit.
│   ├── app.css             Global page styles.
│   ├── App.svelte          Root component. Holds ALL state and both regions.
│   └── lib/
│       ├── BagMount.svelte       The bag's pocket and its three keys.
│       ├── BagDisplay.svelte     The e-ink screen (Level 1).
│       ├── ClubRack.svelte       The grid of club slots on that screen.
│       ├── PhoneShell.svelte     The phone body, status bar and tabs.
│       ├── PhoneRound.svelte     Phone tab: the live round.
│       ├── PhoneSetup.svelte     Phone tab: choose today's 14 clubs.
│       ├── PhoneSummary.svelte   Phone tab: shots chart and clubs used.
│       ├── PairingLine.svelte    The dashed line between the devices.
│       ├── TestPanel.svelte      The simulated sensors in the bottom bar.
│       ├── SimProgress.svelte    Simulation progress bar and caption.
│       ├── BagGraphic.svelte     Placement drawing (Level 0 requirement).
│       ├── WorkbenchLabel.svelte   The small labels above each device.
│       └── WorkbenchOverlay.svelte The pop-up sheet for Info and Fig. 1.
└── GUIDE.md                This file.
```

**The important structural idea:** all state lives in `App.svelte`, not in
the child components. The testing buttons and the device panel both need to
touch the same data, so it lives in their shared parent and flows down as
props. This is worth being able to explain in your presentation, because it
is the main architectural decision in the project.

---

## 4. How the code works

The project uses Svelte 5, which is what `npm init vite` installs today. Note
that the class tutorial PDF references the older Svelte 4 REPL, so if you see
`export let` and `$:` and `on:click` in course materials, that is Svelte 4
syntax. The two styles cannot be mixed inside one component. Everything here
is consistent Svelte 5.

### The four pieces of Svelte you need to understand

**`$state`** makes a value reactive. When it changes, anything displaying it
redraws.

```js
let hole = $state(1);
```

**`$props`** receives values from a parent component.

```js
let { clubs, hole, onNextHole } = $props();
```

**`$derived`** computes a value from other state. It recalculates on its own
whenever its inputs change, so you never update it manually.

```js
let clubsOut = $derived(clubs.filter((c) => !c.inBag));
```

**`{#each}` and `{#if}`** loop and branch in the markup.

```svelte
{#each clubs as club}
  <div class="slot">{club.short}</div>
{/each}

{#if leftBehind.length > 0}
  <div class="alert">Left behind: {leftBehindNames}</div>
{/if}
```

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

### The left-behind logic

This is the one piece of real logic in the project, and it is the thing worth
explaining in your demo. Each club record carries a `pulledAtHole` field:

```js
{ name: "7 Iron", short: "7i", inBag: true, pulledAtHole: null }
```

When a club is pulled, the bag records which hole it happened on. A club is
only treated as lost if it was pulled on an **earlier** hole than the current
one:

```js
let leftBehind = $derived(clubsOut.filter((c) => c.pulledAtHole < hole));
```

That distinction is the whole design idea. A club in your hand on the current
hole is normal and should not trigger an alarm. The same club still missing
after you have walked to the next tee means you left it on the green.

---

## 5. Level 0 requirements, mapped to the code

| Requirement | Where it is |
|---|---|
| Page split into two regions | `App.svelte`, `<section class="device">` and `<section class="testing">` |
| Space reserved for future goals | The dashed "Reserved for Level 2 features" box |
| Project title and your name | `<h1>` and `.byline` in the testing region |
| Link to the write-up | The `<a href="#writeup">` tag. **Replace the `#` with your real URL.** |
| Graphic showing where the UI sits | `BagGraphic.svelte`, four numbered zones |
| Info button explaining the controls | The Info button and `.info-box` in the testing region |
| Buttons to simulate use | Pull club, Return, Advance a hole, Reset bag |

The project also requires that the interface not be confined to one flat
surface. `BagGraphic.svelte` is what satisfies this: the display is on the
side pocket, the buttons are on the strap, the sensors and lights are in the
top cuff, and a stand sensor is in the base. Call this out explicitly in your
write-up and your presentation.

---

## 6. Level 1: controls, indicators, and design goals

Use this table directly in your documentation. The project asks for exactly
this mapping.

### Controls

| Control | What it does | Design goal it serves |
|---|---|---|
| Start / End Round | Begins tracking, resets hole and shot count | The bag should not nag during practice or in the trunk |
| Next Hole | Advances the hole, which triggers the left-behind check | The golfer, not the bag, decides when a hole is over |
| Alerts On / Off | Silences the warning | Users need to mute alerts at the range |

### Indicators

| Indicator | What it shows | Design goal it serves |
|---|---|---|
| Club count, large | How many of 14 are in the bag | Needs to be readable at a glance in sun glare |
| Club rack | Which specific club is out | Knowing one is missing is useless without knowing which |
| Left-behind alert | Club name plus the hole it was left on | Tells the golfer where to walk back to |
| Hole and shot count | Round context | Low-priority, so it is small and at the top |

### Design choices worth defending

- The club count is the largest element because it answers the most common
  question in the least time.
- A club in hand gets a neutral message, not a red alert. Alarming on normal
  use would train the golfer to ignore the alert entirely.
- The alert names the hole, not just the club, because the useful information
  is where to go, not what is gone.
- Buttons are at least 44px tall, since golfers wear gloves.

---

## 7. Choosing a Level 2 goal

The project asks for 1 to 3 options from the Levels 2 to 4 list. Two fit this
object especially well.

### Option 4: simulate a round over time (recommended)

This is the strongest fit for a golf bag, and it demos well. Add a "Play
simulated round" button that steps through a scripted nine holes: pulls a
club, returns it, advances the hole, and on one hole deliberately leaves a
club behind so the alert fires on its own while the grader watches.

Sketch of the approach in `App.svelte`:

```js
let simRunning = $state(false);

// Each step is one scripted event, played on a timer.
const script = [
  { action: "pull", club: "Driver" },
  { action: "return", club: "Driver" },
  { action: "nextHole" },
  { action: "pull", club: "7 Iron" },
  { action: "nextHole" }   // 7 Iron deliberately left behind
];

function runSimulation() {
  simRunning = true;
  let i = 0;
  const timer = setInterval(() => {
    if (i >= script.length) {
      clearInterval(timer);
      simRunning = false;
      return;
    }
    const step = script[i];
    if (step.action === "pull") { selected = step.club; pullClub(); }
    if (step.action === "return") returnClub(step.club);
    if (step.action === "nextHole") nextHole();
    i = i + 1;
  }, 1200);
}
```

`setInterval` runs a function repeatedly on a timer, and `clearInterval`
stops it. Everything on screen updates by itself because the simulation
changes the same state the panel already reads.

### Option 3: round statistics

Track which clubs were used per hole and display a small summary, such as a
bar showing shots per hole or most-used clubs. This option requires 4
selectable scenarios or user profiles in the testing area, so it is more work
than Option 4, but it fills the reserved space well and gives you something
visual.

### What to do with the reserved box

Whatever you choose goes in the dashed "Reserved for Level 2 features" box in
the device region. That box exists so the layout does not have to change when
you add the feature.

---

## 8. GitHub and hosting

The project requires both a source code link and a publicly hosted app link.

### Push to GitHub

```
git init
git add .
git commit -m "Smart golf bag UI, Level 0 and Level 1"
git branch -M main
git remote add origin https://github.com/Jonathan-Cheruiyot/smart-golf-bag.git
git push -u origin main
```

Create the empty repo on github.com first. The Vite scaffold already includes
a `.gitignore` that keeps `node_modules` out, so do not delete that file.

### Deploy the public app

The simplest route for a Vite project:

1. Go to vercel.com and sign in with GitHub.
2. Import the `smart-golf-bag` repo.
3. Accept the defaults. Vercel detects Vite automatically.
4. You get a public URL in about a minute.

Netlify works the same way. Either is fine. GitHub Pages also works but needs
a `base` setting in `vite.config.js`, so it is more fiddly.

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
- [ ] How you implemented it: Svelte 5 with Vite, component structure,
      state in the parent and props flowing down
- [ ] Future work, including anything you did not finish
- [ ] A 2 to 3 minute demo video with voiceover, linked from the page
- [ ] Link to the GitHub repo
- [ ] Link to the hosted app

### Screenshots worth taking

1. Clean state: 14 of 14, all clubs accounted for
2. Club in hand: 13 of 14, neutral message, one hollow slot
3. Left behind: red alert naming the club and hole
4. Info box open

### Demo video outline

Open with the project name and your name. Explain the problem in one
sentence: golfers leave clubs behind and have no way to know until holes
later. Show the two regions and say what each is for. Walk through pulling a
club and advancing a hole so the alert fires live. Point at the placement
graphic and explain the four surfaces. Close with what you would build next.

---

## 10. Troubleshooting

**Page is blank, or a red error overlay appears.** Read the message in the
terminal where `npm run dev` is running. It usually names the file and line.

**"Failed to resolve import ./lib/BagDisplay.svelte"** (or any other
component) The file is in the wrong folder or has the wrong name. It must be
exactly `src/lib/BagDisplay.svelte`.

**npm install hangs for many minutes.** The project is probably inside
OneDrive. Move it somewhere else and run `npm install` again.

**"Could not read package.json"** Your terminal is in the parent folder, not
the project folder. `cd` into the folder that contains `package.json`.

**Changes do not show up.** Check that `npm run dev` is still running and
that you saved the file. A hard refresh with Ctrl+Shift+R clears a stale
cache.

**Port already in use.** An old dev server is still running. Vite will offer
a different port, or you can close the other terminal.
