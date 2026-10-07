# Project 1 Requirements: Interface to a Smart Object

Full requirements from the assignment, organized as a working checklist and
mapped to this project (Smart Golf Bag, Jonathan Cheruiyot).

Status key: **[done]** finished, **[partial]** started, **[todo]** not started.

---

## The assignment in brief

Design a mock-up of an interactive user interface for a physical object that
has been made digital and smart. Implemented in Svelte and JavaScript.

- **Digital:** the object integrates a digital display and digital controls.
- **Smart:** the object senses its usage or environment and provides features
  that support its user.
- **Mock-up:** nothing is physically built. The screen mock-up showcases the
  features and simulates use and behavior.

**The one hard constraint on object choice:** the interactive controls and
display cannot all be on one flat surface.

> **How the golf bag satisfies this:** the display is on the side pocket, the
> buttons are on the shoulder strap, the club sensors and slot lights are in
> the top cuff, and a stand sensor is in the base. Four distinct surfaces.
> `BagGraphic.svelte` is the drawing that shows this, and it should be called
> out explicitly in the write-up and the presentation.

---

## Grade breakdown

| Component | Weight |
|---|---|
| Design | 25% |
| Implementation | 50% |
| Documentation | 20% |
| Presentation | 5% |

---

## 1. Design (25%)

### 1a. Affordances and physical properties

Referencing the course slides on affordances and *The Design of Everyday
Things*, describe the physical properties of the object. Is it small, large,
portable, fixed? Can it go in a pocket? Does it stand up on a table?

- [ ] **[todo]** Written description

> **Draft for the golf bag:** Large and tall, heavy when full. Portable but
> awkward, carried by a shoulder strap or strapped to a cart. Stands on two
> fold-out legs, tilted back rather than upright. Used outdoors in sun glare,
> rain, and dirt, often with a glove on one hand. The top cuff, side pockets,
> strap, and base are all distinct surfaces a user already touches.

### 1b. Capturing user needs

Interview 3 people outside the class. Design interview questions that reveal
user needs and how the UI can address them. Report what you learned.

Where appropriate, have participants demonstrate how they typically use the
object. Photos or video are allowed, but identifying information must be
removed before posting.

- [ ] **[todo]** Interview 3 people
- [ ] **[todo]** Write up what you learned

> **Draft interview questions:**
> 1. Walk me through what you do with your bag during a round.
> 2. Have you ever left a club behind? How did you find out, and how long did
>    it take?
> 3. Do you carry, use a push cart, or ride? Does that change how you use the
>    bag?
> 4. What do you check on your phone or scorecard during a round?
> 5. Where on the bag would you expect to look for information?
> 6. What would make a screen on your bag annoying instead of useful?

### 1c. Smart and sensing feature assumptions

State what the object senses. Features must be feasible, but you do not have
to explain the technical implementation.

- [ ] **[todo]** Written list

> **Draft for the golf bag:** Each club divider senses whether its club is in
> the slot. A base sensor detects when the bag is set down. The bag knows the
> current hole, either by GPS or by the golfer pressing Next Hole.

### 1d. User needs and design requirements

A written list pairing each user need with the design requirement it creates.

- [ ] **[todo]** Final list, revised after the interviews

> **Draft, to be revised once interviews are done:**
>
> | User need | Design requirement |
> |---|---|
> | Know if a club got left behind | Alert names the missing club and the hole it was last used on |
> | Check the bag at a glance | Club count is the largest element on the display |
> | Read it in bright sun | High contrast panel, large type |
> | Find the empty slot quickly | Light under each empty slot, rack shown on screen |
> | Control it with a glove on | Large strap buttons that work by feel |
> | Skip alerts during practice | Alerts on/off control |

### 1e. Sketching

- [ ] **[todo]** 10-plus-10 sketches for 3 design challenges (if 10 full
      sketches is too much, 10 minutes per challenge is acceptable)
- [ ] **[todo]** Vanilla sketch of the interface (1)
- [ ] **[todo]** Storyboard sketch (1)
- [ ] **[todo]** Hybrid sketch showing the interface on a real object (1)

> **Shortcut for the hybrid sketch:** photograph a real golf bag and draw the
> four zones from `BagGraphic.svelte` on top of the photo.
>
> **Shortcut for the storyboard:** the four panel states already in the code
> are the plot. Golfer pulls a club, walks to the next tee, bag alerts, club
> goes back.

### 1f. Evaluate (user feedback)

Show the vanilla UI sketch to 3 people and record their feedback.

- [ ] **[todo]** Collect feedback from 3 people
- [ ] **[todo]** Write it up

---

## 2. Implementation (50%)

### Level 0: partition the page

Divide the page into two regions.

**Region 1, Device UI:** prototypes the interface you envision for the
object. Controls, indicators, settings. Leave space for future project goals.

**Region 2, project info and testing:** who you are, what the project is, and
buttons to test usage of the mock object.

Required elements:

- [x] **[done]** Project title
- [x] **[done]** Your name
- [ ] **[partial]** Link to the project write-up (placeholder `href="#writeup"`
      in `App.svelte`, needs the real URL)
- [x] **[done]** Graphic showing where the UI resides on the physical object
      (`BagGraphic.svelte`)
- [x] **[done]** Info button explaining the simulation controls
- [x] **[done]** Buttons that simulate use (Pull club, Return, Advance a hole,
      Reset bag)
- [x] **[done]** Space reserved for future goals (the dashed box)

Note: the UI does not need to be responsive. A fixed size is fine, since a
real smart object has a fixed display.

### Level 1: basic object UI

Focus on basic controls and usage. Produce a list of the controls, the
indicators, and how they connect to the design goals.

- [x] **[done]** Implemented in `ClubTracker.svelte`
- [x] **[done]** Controls and indicators list (below, reuse in the write-up)

**Controls**

| Control | What it does | Design goal it serves |
|---|---|---|
| Start / End Round | Begins tracking, resets hole and shot count | The bag should not nag during practice or in the trunk |
| Next Hole | Advances the hole, which triggers the left-behind check | The golfer decides when a hole is over, not the bag |
| Alerts On / Off | Silences the warning | Users need to mute alerts at the range |

**Indicators**

| Indicator | What it shows | Design goal it serves |
|---|---|---|
| Club count, large | How many of 14 are in the bag | Readable at a glance in sun glare |
| Club rack | Which specific club is out | Knowing one is missing is useless without knowing which |
| Left-behind alert | Club name plus the hole it was left on | Tells the golfer where to walk back to |
| Hole and shot count | Round context | Low priority, so it is small and at the top |

**Design choices**

- The club count is the largest element because it answers the most common
  question in the least time.
- A club in hand gets a neutral message, not a red alert. Alarming on normal
  use would train the golfer to ignore alerts entirely.
- The alert names the hole, not just the club, because the useful information
  is where to walk back to.
- Buttons are at least 44px tall, since golfers wear gloves.

### Levels 2 to 4: choose 1 to 3 options

- [ ] **[todo]** Pick at least one

**Option 1. Complex set of selections.** Design a UI for inputs the basic
interface cannot handle. Selections must be clearly visible with quick
feedback.

**Option 2. Mock secondary device.** Add a region showing an interactive
phone or watch UI. Define in the write-up why the secondary device exists and
how interactions sync both ways.

**Option 3. Display sensor or usage data.** Visualizations, SVG graphics, or
text showing captured data. Requires **4 selectable scenarios or user
profiles** in the testing area so the grader can see the data change.

**Option 4. Simulate the object in use over time.** A button activates a
scripted or timed simulation. All visual indicators update dynamically as it
runs.

**Option 5. Propose your own.** Must be interesting and sufficiently
challenging, and must involve new UI features. Run it by the professor first.

> **Recommendation: Option 4.** It fits a golf bag better than the others and
> demos well. Script a nine-hole round that pulls clubs, returns them,
> advances holes, and deliberately leaves one club behind so the alert fires
> on its own while the grader watches. `GUIDE.md` section 7 has a code sketch.
>
> Option 3 is the natural second if you want two. Note that it carries the
> extra requirement of 4 scenarios or user profiles.

---

## 3. Documentation (20%)

Written for someone encountering the project for the first time. Must be
publicly available on your portfolio page.

- [ ] **[todo]** Describe the project
- [ ] **[todo]** Present the design work from section 1
- [ ] **[todo]** Describe the interface in detail, explaining features and
      controls
- [ ] **[todo]** Plenty of screenshots showing different actions users can
      perform
- [ ] **[todo]** Explain the implementation: libraries, code structure
- [ ] **[todo]** Future work, including anything attempted but not finished,
      with screenshots of the progress
- [ ] **[todo]** **AI documentation: describe how you used AI, if you used it**
- [ ] **[todo]** 2 to 3 minute demo video with voiceover, linked from the page
- [ ] **[todo]** Link to the source code on GitHub
- [ ] **[todo]** Link to the publicly hosted application

### On the implementation section

> Svelte 5 with Vite. Three components: `App.svelte` holds all application
> state and renders both regions; `ClubTracker.svelte` is the device panel
> and receives state as props; `BagGraphic.svelte` is a static SVG showing
> interface placement. State lives in the parent because the testing buttons
> and the device panel both act on the same data. Reactivity uses Svelte 5
> runes: `$state` for the club array and round status, `$derived` for
> computed values such as which clubs are out and which were left behind.

### On the AI documentation section

This is a new requirement in this version of the spec, so do not skip it.
Be specific and honest about what AI did and what you did. Note which parts
you understand well enough to defend, since you have to present this live.

### Screenshots worth taking

1. Clean state: 14 of 14, all clubs accounted for
2. Club in hand: 13 of 14, neutral message, one hollow slot
3. Left behind: red alert naming the club and the hole
4. Info box open
5. Whatever Level 2 feature you build

### Demo video outline

Open with the project name and your name. State the problem in one sentence:
golfers leave clubs behind and have no way to know until several holes later.
Show the two regions and say what each is for. Walk through pulling a club
and advancing a hole so the alert fires live. Point at the placement graphic
and explain the four surfaces. Close with what you would build next.

Record with a screen capture tool that also captures audio, such as
QuickTime. Host on your webpage or YouTube, but it must be linked from your
webpage.

---

## 4. Presentation (5%)

- [ ] **[todo]** 5 to 6 minute talk to a small group, plus 1 to 2 minutes of
      questions

Slides, video, or a live demo are all allowed.

> A live demo is the strongest option here, because the left-behind alert
> firing in real time is more convincing than a screenshot of it. Have the
> app already running on localhost before you start so nothing has to load.
> Keep a recorded video as a backup in case the laptop misbehaves.

---

## Remaining work, in priority order

**Chosen approach: build first, validate after.** The design section is being
done by hand in parallel, and the user research runs against the finished
prototype rather than before it. See the method note at the end of
`QUESTIONNAIRE.md` for how to describe this honestly in the write-up.

### Build

1. Work `SPECIFICATIONS.md` phases 0 through 8 with Claude Code
2. Push to GitHub and deploy to Vercel
3. Paste the real write-up URL into `App.svelte`, replacing `#writeup`

### Design section, done by hand alongside the build

4. Affordances write-up
5. 10-plus-10 sketches, vanilla sketch, storyboard, hybrid sketch

### After the prototype works

6. Run `QUESTIONNAIRE.md` with 3 people. This covers both the user-needs
   requirement and the sketch-feedback requirement, asked against the built
   interface instead of a sketch
7. Revise the user needs and design requirements table with what they said
8. Write the documentation page, including the AI section and a one-sentence
   note on the method ordering
9. Record the demo video
