# START HERE

Smart Golf Bag, Project 1. Where the project stands and how to work on it.

---

## Where things stand

The build is finished through all nine phases in `SPECIFICATIONS.md`, and the
code is on GitHub at https://github.com/Jonathan-Cheruiyot/smart-golf-bag.

Two build steps are left, and both are yours:

1. **Deploy to Vercel.** Go to vercel.com, sign in with GitHub, import the
   `smart-golf-bag` repo, accept the defaults.
2. **Add the write-up link.** Once the write-up is live, replace `#writeup`
   in `src/App.svelte` with its URL, then commit and push.

Everything after that is the design section, the user evaluation and the
documentation. See "Order of work" at the bottom.

---

## Step 1: Run it

Open the project folder in a terminal and run:

```
npm run dev
```

Open the localhost URL it prints. You should see the bag display on the left
and the phone on the right, with a bar of test controls along the bottom.

On a machine that does not have the project yet:

```
git clone https://github.com/Jonathan-Cheruiyot/smart-golf-bag.git
cd smart-golf-bag
npm install
npm run dev
```

---

## Step 2: Check it works

Try the main feature by hand:

1. Press the **Start Round** key under the bag's screen.
2. Press **7i** in the test bar to lift the 7 Iron out.
3. Press **Next Hole** on the bag's strap.

A red alert on the bag should name the club and the hole, and a moment later
the phone should show a notification. Press **7i** again to put it back.

Then press **Play round** in the test bar and watch a scripted nine holes
play by itself.

The **Info** button at the top right explains every test control.

---

## Step 3: Making changes with Claude Code

In the project folder, open a terminal and run:

```
claude
```

Then describe the change you want. It reads `CLAUDE.md` automatically, so it
already knows the design rules. Two habits that have worked on this project:

- Ask for one thing at a time, look at the result, then ask for the next.
- Ask it to commit and push when you are happy with a change.

---

## What each file is for

| File | Who reads it | What it does |
|---|---|---|
| `CLAUDE.md` | Claude Code, automatically | Design direction, color tokens, fonts, code conventions, things never to do |
| `SPECIFICATIONS.md` | Claude Code, and you | The 9 build phases as built, with every decision recorded |
| `MOTION.md` | Claude Code, and you | The rules for animation: the bag snaps, the phone slides, the page is still |
| `REQUIREMENTS.md` | You | The course requirements as a checklist, with what is done and what is left |
| `GUIDE.md` | You | How the code works, Svelte concepts, deployment, troubleshooting |
| `QUESTIONNAIRE.md` | You | User evaluation protocol to run now that the prototype works |

The code itself is in `src/`. `GUIDE.md` section 3 lists every file in it.

---

## The five decisions, and how they came out

`SPECIFICATIONS.md` marked five points where the build stopped to ask you.
You may be asked about these in the presentation.

1. **Device arrangement.** Bag and phone side by side on a centre stage, with
   project info in a slim header and the test controls in a quiet bottom bar.
2. **Phone personality.** A companion that mirrors and explains the bag, plus
   a remote for choosing clubs. Its Round tab also repeats the bag's three
   controls.
3. **Club set.** A standard 17-club set with no brands, 14 loaded at a time.
4. **Simulation pacing.** No speed control. It began at 1.2 seconds per
   step and is now 0.8, for a shot-by-shot round of about 57 seconds.
5. **URLs.** GitHub repo set. The write-up URL is still to come.

---

## If something breaks

Check `GUIDE.md` section 10. The common ones:

- **Blank page or red error overlay.** Read the terminal where `npm run dev`
  is running. It names the file and line.
- **"Failed to resolve import"** A file is in the wrong folder. Components go
  in `src/lib/`, not `src/`.
- **npm install hangs for minutes.** The project is in OneDrive. Move it out.
- **"Could not read package.json"** Your terminal is one folder too high.
  `cd` into the folder with `package.json` in it.

---

## Order of work

1. ~~Build phases 0 to 8 with Claude Code~~ Done
2. ~~Push to GitHub~~ Done. Deploy to Vercel: still to do
3. Do the design section by hand: affordances, 10-plus-10, vanilla sketch,
   storyboard, hybrid sketch
4. Run `QUESTIONNAIRE.md` with 3 people
5. Write the documentation page, including the AI section
6. Record the 2 to 3 minute demo video
7. Put the write-up URL into `src/App.svelte`, commit and push
