# START HERE

Smart Golf Bag, Project 1. Everything you need to get building.

---

## Step 1: Have a project folder

If you already have a working Vite + Svelte project, skip to Step 2.

If not, open a terminal and run these one at a time:

```
mkdir C:\Users\cheru\dev
cd C:\Users\cheru\dev
npm init vite
```

At the prompts: name it `golf-bag`, choose **Svelte**, choose **JavaScript**.
Then:

```
cd golf-bag
npm install
npm run dev
```

Open the localhost URL it prints. You should see the default Vite counter
page. That confirms the scaffold works before you change anything.

---

## Step 2: Copy these files in

Unzip this bundle **into the project root**, the folder that contains
`package.json`. The folder structure already matches, so files land where
they belong:

```
golf-bag/
├── package.json          (already there, from Vite)
├── index.html            (already there, from Vite)
├── CLAUDE.md             <- new
├── SPECIFICATIONS.md     <- new
├── REQUIREMENTS.md       <- new
├── GUIDE.md              <- new
├── QUESTIONNAIRE.md      <- new
├── START-HERE.md         <- new, this file
└── src/
    ├── main.js           (already there, do not touch)
    ├── App.svelte        <- replaces the Vite one
    ├── app.css           <- replaces the Vite one
    └── lib/
        ├── ClubTracker.svelte   <- new
        └── BagGraphic.svelte    <- new
```

Say yes when it asks about replacing `App.svelte` and `app.css`.

Then delete `src/lib/Counter.svelte`, the leftover Vite demo component.

---

## Step 3: Check it runs

```
npm run dev
```

You should see the golf bag panel and the testing buttons. Try it: press
Start Round, pull the 7 Iron, press Advance a hole. A red alert should name
the club and the hole.

If that works, the foundation is solid and Claude Code has something real to
build on.

---

## Step 4: Open the Claude terminal in VS Code

In the project folder, open a terminal and run:

```
claude
```

Then paste this as your first message:

```
Read CLAUDE.md and SPECIFICATIONS.md, then build Phase 0 only.
Stop when it is done, tell me what you changed, and wait for me
before starting Phase 1.
```

That is the whole workflow. One phase at a time, you check it, then you say
go for the next one.

---

## What each file is for

| File | Who reads it | What it does |
|---|---|---|
| `CLAUDE.md` | Claude Code, automatically | Design direction, color tokens, fonts, code conventions, things never to do |
| `SPECIFICATIONS.md` | Claude Code, when you point it there | The 9 build phases with acceptance checks and 5 decision points |
| `REQUIREMENTS.md` | You | The course requirements as a checklist, with what is done and what is left |
| `GUIDE.md` | You | How the code works, Svelte concepts, deployment, troubleshooting |
| `QUESTIONNAIRE.md` | You, later | User evaluation protocol to run once the prototype works |

---

## The five decision points

Claude Code will stop and ask you at these points. They are in the spec, so
you do not have to remember them:

1. **Phase 1** How the two devices are arranged on the page
2. **Phase 3** Whether the phone is a companion or a remote control
3. **Phase 4** Which clubs you actually carry
4. **Phase 5** How fast the round simulation plays
5. **Phase 8** Your GitHub and write-up URLs

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

1. Build phases 0 to 8 with Claude Code
2. Do the design section by hand: affordances, 10-plus-10, vanilla sketch,
   storyboard, hybrid sketch
3. Push to GitHub, deploy to Vercel
4. Run `QUESTIONNAIRE.md` with 3 people
5. Write the documentation page, including the AI section
6. Record the 2 to 3 minute demo video
