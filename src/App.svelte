<script>
  import { untrack } from "svelte";
  // The page is a drafting table, and its hierarchy is the design argument:
  // the two devices are the centre stage and everything else serves them.
  // Project info is a slim header strip, the placement drawing is a small
  // figure in that strip that opens as an overlay, and the test panel is a
  // quiet bar along the bottom, so the simulated sensors never compete with
  // the thing they are simulating. Fixed widths because a real bag display is
  // a fixed size and the assignment waives responsiveness.
  //
  // All application state lives in this file. Components receive it as props
  // and change it only through the functions below, so the bag, the phone and
  // the test panel can never disagree about what is in the bag.
  import WorkbenchLabel from "./lib/WorkbenchLabel.svelte";
  import WorkbenchOverlay from "./lib/WorkbenchOverlay.svelte";
  import BagGraphic from "./lib/BagGraphic.svelte";
  import BagMount from "./lib/BagMount.svelte";
  import BagDisplay from "./lib/BagDisplay.svelte";
  import PairingLine from "./lib/PairingLine.svelte";
  import PhoneShell from "./lib/PhoneShell.svelte";
  import TestPanel from "./lib/TestPanel.svelte";
  import SimProgress from "./lib/SimProgress.svelte";

  // ===== STATE =====

  // One entry per club slot. pulledAtHole is what makes left-behind detection
  // possible: a club in hand on the CURRENT hole is normal, the same club
  // still out on a LATER hole was left behind.
  function club(id, name, short, category, loaded = true) {
    return {
      id,
      name,
      short,
      category, // wood | hybrid | iron | wedge | putter
      loaded, // is it in today's 14, set from the phone
      inBag: true, // is it physically in its slot right now
      pulledAtHole: null // hole number it was taken out on
    };
  }

  // The 17 clubs the golfer owns: a standard set, no brands. The rules allow
  // 14 in the bag, so three start the day at home and the phone's Setup tab
  // swaps them in, the way golfers trade a wood for a hybrid to suit a course.
  const LIMIT = 14;

  let clubs = $state([
    club("dr", "Driver", "DR", "wood"),
    club("w3", "3 Wood", "3W", "wood"),
    club("w5", "5 Wood", "5W", "wood"),
    club("h3", "3 Hybrid", "3H", "hybrid"),
    club("h4", "4 Hybrid", "4H", "hybrid", false),
    club("h5", "5 Hybrid", "5H", "hybrid", false),
    club("i4", "4 Iron", "4i", "iron"),
    club("i5", "5 Iron", "5i", "iron"),
    club("i6", "6 Iron", "6i", "iron"),
    club("i7", "7 Iron", "7i", "iron"),
    club("i8", "8 Iron", "8i", "iron"),
    club("i9", "9 Iron", "9i", "iron"),
    club("pw", "Pitching Wedge", "PW", "wedge"),
    club("gw", "Gap Wedge", "GW", "wedge"),
    club("sw", "Sand Wedge", "SW", "wedge"),
    club("w60", "60° Wedge", "60", "wedge", false),
    club("pt", "Putter", "PT", "putter")
  ]);

  // The course card: fixed data about the course, not something that is
  // sensed. The front nine is par 36, 3,235 yards.
  const COURSE = {
    name: "Tiger Village Golf Course",
    holes: [
      { yards: 485, par: 5 },
      { yards: 389, par: 4 },
      { yards: 364, par: 4 },
      { yards: 195, par: 3 },
      { yards: 390, par: 4 },
      { yards: 352, par: 4 },
      { yards: 510, par: 5 },
      { yards: 178, par: 3 },
      { yards: 372, par: 4 }
    ]
  };

  let round = $state({
    active: false,
    hole: 1, // 1 to 18
    strokes: 0, // the score so far, as a scorecard counts it
    log: [], // { hole, clubId, strokes } appended on each return
    // Yards from the golfer to the pin, or null when it is not known. The
    // PHONE's knowledge, never the bag's: the phone is in the golfer's
    // pocket, the bag is back on the cart path.
    toPin: null
  });

  let bag = $state({
    alertsOn: true,
    distanceFromGolfer: 0 // metres, driven by the test panel
  });

  let phone = $state({
    paired: true,
    tab: "round", // round | setup | summary
    notifications: [] // { id, kind, title, text, hole }
  });

  // Which overlay is open. Page chrome, not product state.
  let showInfo = $state(false);
  let showPlacement = $state(false);

  // ===== DERIVED VALUES =====

  let loadedClubs = $derived(clubs.filter((c) => c.loaded));
  let clubsOut = $derived(loadedClubs.filter((c) => !c.inBag));
  let inBagCount = $derived(loadedClubs.length - clubsOut.length);

  // The heart of the project: a club is left behind if it is still out of
  // the bag on a later hole than the one it was pulled on.
  let leftBehind = $derived(clubsOut.filter((c) => c.pulledAtHole < round.hole));

  // The bag itself has been left behind: the golfer is more than 30 metres
  // away mid-round. Only the phone can report this, because nobody is
  // standing at the bag to read its screen. This is why there are two devices.
  let bagAbandoned = $derived(bag.distanceFromGolfer > 30 && round.active);

  // ===== THE RELAY: bag to phone =====

  // When the bag raises a left-behind alert, the phone's notification follows
  // 200ms later. The delay makes the causal chain visible: the bag sensed it,
  // then told the phone. Both come from the one state change in leftBehind,
  // so they can never disagree. The away alert has no delay, because the
  // phone measures the distance itself and nothing is relayed.
  const RELAY_MS = 200;

  // True for those 200ms, while the alert is "in the air" between the two
  // devices. The pairing line on the stage flashes while it is.
  let relaying = $state(false);

  $effect(() => {
    const wanted = [];

    if (bag.alertsOn && leftBehind.length === 1) {
      const c = leftBehind[0];
      wanted.push({
        id: "left-behind",
        kind: "left-behind",
        hole: c.pulledAtHole,
        title: "Club left behind",
        // Worded for someone who is NOT looking at the bag: what is missing
        // and where they last had it, which is where to walk back to.
        text: `Your ${c.name} is not in the bag. You last had it on hole ${c.pulledAtHole}.`
      });
    } else if (bag.alertsOn && leftBehind.length > 1) {
      wanted.push({
        id: "left-behind",
        kind: "left-behind",
        hole: leftBehind[0].pulledAtHole,
        title: `${leftBehind.length} clubs left behind`,
        // Names only: the clubs-out list below gives the hole for each.
        text: "Not in the bag: " + leftBehind.map((c) => c.name).join(", ") + "."
      });
    }

    if (bag.alertsOn && bagAbandoned) {
      wanted.push({
        id: "away",
        kind: "away",
        hole: round.hole,
        title: "You left your bag",
        text: `Your bag is ${bag.distanceFromGolfer} m behind you.`
      });
    }

    // untrack: this block edits phone.notifications, and must not re-run
    // itself because of its own edit.
    return untrack(() => {
      // Anything no longer true clears at once.
      phone.notifications = phone.notifications.filter((n) =>
        wanted.some((w) => w.id === n.id)
      );

      const relayed = [];
      for (const w of wanted) {
        const showing = phone.notifications.find((n) => n.id === w.id);
        if (showing) {
          // Already on screen: keep it and refresh the wording in place.
          showing.title = w.title;
          showing.text = w.text;
          showing.hole = w.hole;
        } else if (w.kind === "away") {
          phone.notifications.push(w);
        } else {
          relayed.push(w);
        }
      }

      if (relayed.length === 0) return;
      relaying = true;
      const timer = setTimeout(() => {
        phone.notifications.unshift(...relayed);
        relaying = false;
      }, RELAY_MS);
      return () => {
        clearTimeout(timer);
        relaying = false;
      };
    });
  });

  // The card for the hole being played, or null when idle or past the nine
  // holes on the card. The bag prints this in its header.
  let holeCard = $derived(round.active ? (COURSE.holes[round.hole - 1] ?? null) : null);

  // ===== CONTROLS ON THE BAG, mirrored on the phone's Round tab =====

  // Standing on a tee, the distance to the pin is simply the hole's yardage.
  function teeOff() {
    round.toPin = COURSE.holes[round.hole - 1]?.yards ?? null;
  }

  function toggleRound() {
    round.active = !round.active;
    round.toPin = null;
    if (round.active) {
      round.hole = 1;
      round.strokes = 0;
      round.log = [];
      teeOff();
      for (const c of clubs) {
        // A club already in hand when the round starts counts as pulled on
        // hole 1, so it is not flagged until the golfer moves on without it.
        if (!c.inBag) c.pulledAtHole = 1;
      }
    }
  }

  function nextHole() {
    if (round.active && round.hole < 18) {
      round.hole += 1;
      teeOff();
    }
  }

  function toggleAlerts() {
    bag.alertsOn = !bag.alertsOn;
  }

  // ===== CONTROLS ON THE PHONE =====

  function setPhoneTab(tab) {
    phone.tab = tab;
  }

  // Phone to bag: the rack on the bag redraws as soon as this changes. The
  // two guards repeat the checks in PhoneSetup, so the rule holds no matter
  // what calls this.
  function toggleLoaded(id) {
    const c = clubs.find((c) => c.id === id);
    if (!c) return;
    if (!c.loaded && loadedClubs.length >= LIMIT) return;
    if (c.loaded && !c.inBag) return;
    c.loaded = !c.loaded;
  }

  // ===== SENSORS: called by the test panel, standing in for the slots =====

  function pullClub(id) {
    const c = clubs.find((c) => c.id === id);
    if (c && c.loaded && c.inBag) {
      c.inBag = false;
      c.pulledAtHole = round.hole;
    }
  }

  // A return is one use of a club: it went out, was played, and came back.
  // It is logged against the hole it was pulled on, not the hole it was
  // returned on, so a club left behind is still credited to the right hole.
  //
  // A use is not always one stroke. A two-putt is two strokes and one use of
  // the putter. The bag senses the club coming back, not the swing, so the
  // number of strokes is passed in. A press on the test panel counts as one;
  // the simulation says how many each of its steps represents.
  //
  // toPin is where the golfer now stands, in yards from the pin. On the real
  // product the phone's GPS would supply it; here the simulation does. A
  // manual press on the test panel has no position to give, so the phone
  // shows the distance as unknown until the next tee.
  function returnClub(id, strokes = 1, toPin = null) {
    const c = clubs.find((c) => c.id === id);
    if (c && !c.inBag) {
      if (round.active) {
        round.strokes += strokes;
        round.log.push({ hole: c.pulledAtHole, clubId: c.id, strokes });
        round.toPin = toPin;
      }
      c.inBag = true;
      c.pulledAtHole = null;
    }
  }

  function setDistance(metres) {
    bag.distanceFromGolfer = metres;
  }

  function resetBag() {
    for (const c of clubs) {
      c.inBag = true;
      c.pulledAtHole = null;
    }
    round.active = false;
    round.hole = 1;
    round.strokes = 0;
    round.log = [];
    round.toPin = null;
    bag.distanceFromGolfer = 0;
    sim.step = 0;
  }

  // ===== ROUND SIMULATION =====

  // A scripted nine holes that plays by itself, one step every 0.8 seconds,
  // so both devices can be watched telling the whole story. Every step calls
  // the SAME functions the buttons and sensors call (toggleRound, pullClub,
  // returnClub, nextHole, setDistance). There is no separate code path, so
  // what the simulation shows is what the real interface does.
  //
  // The round is a 6 handicap going out in 38 on a par 36. Each caption
  // narrates one shot, which is what makes it worth watching: the viewer
  // follows a golfer, and the two screens react to him.
  const STEP_MS = 800;

  const STANDARD_SET = ["dr", "w3", "w5", "h3", "i4", "i5", "i6", "i7", "i8", "i9", "pw", "gw", "sw", "pt"];

  // The opening words of each hole's first caption.
  const tee = (n) => `Hole ${n}, ${COURSE.holes[n - 1].yards} yds, par ${COURSE.holes[n - 1].par} - `;

  // Distances near the green are given in feet, as golfers do. toPin is kept
  // in yards, so a distance in feet is written as feet * FT.
  const FT = 1 / 3;

  // Every step declares how many strokes it represents and where it leaves
  // the golfer. Both are recorded when the club goes back in the bag,
  // because that is the moment the bag learns the shot is over, so the step
  // that pulls a club declares no strokes.
  //
  // One shot is two steps: the club comes out, then it goes back. Both carry
  // the same caption, so each shot stays on screen long enough to read.
  const shot = (id, toPin, caption, strokes = 1) => [
    { caption, strokes: 0, run: () => pullClub(id) },
    { caption, strokes, toPin, run: (step) => returnClub(id, step.strokes, step.toPin) }
  ];

  // A shot after which the club is NOT put back. These two are the story.
  // Its stroke is declared by the later step that finally returns the club.
  const shotAndLeave = (id, caption) => [{ caption, strokes: 0, run: () => pullClub(id) }];
  const recover = (id, toPin, caption, strokes = 1) => [
    { caption, strokes, toPin, run: (step) => returnClub(id, step.strokes, step.toPin) }
  ];

  const walk = (caption) => [{ caption, strokes: 0, run: () => nextHole() }];

  // Each full shot is hit with the club whose stock distance matches the
  // yardage left, using the 5 handicap column of a published distance chart:
  // driver 261, 3 hybrid 216, 4 iron 201, 6 iron 172, 7 iron 164, 9 iron 139,
  // pitching wedge 126, gap wedge 109, sand wedge 86. Where no club is exact
  // the result follows the club: the hybrid (216) is just short from 224 on
  // the 1st, the 4 iron (201) flies the 195 yard 4th, and the 6 iron (172)
  // comes up short on the 178 yard 8th.
  //
  // Each caption gives the distance hit and the distance left, and the two
  // always add up: 485 less a 261 drive is 224 to the pin.
  //
  // Strokes by hole: 5 4 4 4 4 5 4 3 5, which is 38.
  const SCRIPT = [
    {
      caption: `${COURSE.name}, front nine, par 36. A 6 handicap on the 1st tee.`,
      strokes: 0,
      run: () => toggleRound()
    },

    ...shot("dr", 224, tee(1) + "Driver, 261. 224 to the pin."),
    ...shot("h3", 8, "3 hybrid, 216, just short of the green. 24 feet to the pin."),
    ...shot("sw", 10 * FT, "Sand wedge chip. 10 feet to the pin."),
    ...shot("pt", 0, "Putter, two putts from 10 feet. Par.", 2),
    ...walk("Level par through 1. Walk to hole 2."),

    ...shot("dr", 126, tee(2) + "Driver, 263. 126 to the pin."),
    ...shot("pw", 20 * FT, "Pitching wedge, 126, on the green. 20 feet to the pin."),
    ...shot("pt", 0, "Putter, two putts from 20 feet. Par.", 2),
    ...walk("Level par through 2. Walk to hole 3."),

    // First leave-behind: the sand wedge is dropped beside the green for the
    // putt and never picked up.
    ...shot("dr", 109, tee(3) + "Driver, 255, pushed right. 109 to the pin."),
    ...shot("gw", 9, "Gap wedge, 100, short right of the green. 27 feet to the pin."),
    ...shotAndLeave("sw", "Sand wedge chip. 4 feet to the pin."),
    ...shot("pt", 0, "Putter, holes the 4 footer. Par save. The sand wedge stays on the fringe."),
    // Nobody presses anything here. The alert fires because the hole number
    // moved past the hole the club was pulled on.
    ...walk("Level par through 3. Walk to hole 4. The sand wedge is still by the 3rd green."),

    ...shot("i4", 8, tee(4) + "4 iron, 203, flies the green. 24 feet to the pin."),
    ...shot("gw", 12 * FT, "Reaches for the sand wedge: gone. Gap wedge chip instead, runs past. 12 feet to the pin."),
    ...shot("pt", 0, "Putter, two putts from 12 feet. Bogey.", 2),
    // The chip on the 3rd is counted now, when the bag gets the club back.
    ...recover("sw", 0, "Walks back to the 3rd green and puts the sand wedge in the bag."),
    ...walk("1 over through 4. Walk to hole 5."),

    ...shot("dr", 139, tee(5) + "Driver, 251. 139 to the pin."),
    ...shot("i9", 18 * FT, "9 iron, 139, on the green. 18 feet to the pin."),
    ...shot("pt", 0, "Putter, two putts from 18 feet. Par.", 2),
    ...walk("1 over through 5. Walk to hole 6."),

    ...shot("dr", 86, tee(6) + "Driver, 266. 86 to the pin."),
    ...shot("sw", 10, "Sand wedge, 86, spins back into the front bunker. 30 feet to the pin."),
    ...shot("sw", 12 * FT, "Sand wedge from the bunker. 12 feet to the pin."),
    ...shot("pt", 0, "Putter, two putts from 12 feet. Bogey.", 2),
    ...walk("2 over through 6. Walk to hole 7."),

    // Second leave-behind: the putter, forgotten in the moment after a birdie.
    ...shot("dr", 249, tee(7) + "Driver, 261. 249 to the pin."),
    ...shot("i7", 85, "7 iron lay-up, 164. 85 to the pin."),
    ...shot("sw", 5 * FT, "Sand wedge, 85. 5 feet to the pin."),
    ...shotAndLeave("pt", "Putter, holes the 5 footer. Birdie. The putter stays on the green."),
    ...walk("1 over through 7. Walk to hole 8 without the putter."),

    ...shot("i6", 10, tee(8) + "6 iron, 168, front edge of the green. 30 feet to the pin."),
    // The birdie putt on the 7th is counted now. He is still 30 feet away.
    ...recover("pt", 10, "Reaches for the putter: it is on the 7th green. His playing partner brings it over."),
    ...shot("pt", 0, "Putter, two putts from 30 feet. Par.", 2),
    ...walk("1 over through 8. Walk to hole 9."),

    ...shot("dr", 109, tee(9) + "Driver, 263. 109 to the pin."),
    ...shot("gw", 35 * FT, "Gap wedge, 109, on the green but wide. 35 feet to the pin."),
    ...shot("pt", 0, "Putter, three putts from 35 feet. Bogey. Out in 38, 2 over.", 3),

    // The second alert, at the larger scale: the bag itself is left behind,
    // and only the phone can say so.
    {
      caption: "Walks to the clubhouse. The bag is still by the 9th green.",
      strokes: 0,
      run: () => setDistance(60)
    },
    {
      caption: "The phone says the bag is 60 m behind him. Walks back for it.",
      strokes: 0,
      run: () => setDistance(0)
    },
    { caption: "Round over: 38 on a par 36.", strokes: 0, run: () => toggleRound() }
  ];

  let sim = $state({
    running: false,
    step: 0 // how many steps have played
  });

  let simCaption = $derived(sim.step > 0 ? SCRIPT[sim.step - 1].caption : "");

  // The interval's id. Not $state: nothing on screen depends on it.
  let simTimer = null;

  function simTick() {
    const step = SCRIPT[sim.step];
    step.run(step);
    sim.step += 1;
    if (sim.step >= SCRIPT.length) stopSim();
  }

  function playSim() {
    if (sim.running) return;
    // Put the stage back to a known start so the script always tells the
    // same story: clubs in, the standard 14 loaded, alerts on, Round tab up.
    resetBag();
    for (const c of clubs) c.loaded = STANDARD_SET.includes(c.id);
    bag.alertsOn = true;
    phone.tab = "round";

    sim.running = true;
    simTick();
    simTimer = setInterval(simTick, STEP_MS);
  }

  // Always clear the interval, whether the script finished or was stopped,
  // or it would keep firing in the background.
  function stopSim() {
    clearInterval(simTimer);
    simTimer = null;
    sim.running = false;
  }

  // And clear it if the page itself goes away mid-run.
  $effect(() => () => clearInterval(simTimer));
</script>

<main>
  <!-- ============ HEADER STRIP: PROJECT INFO AND PLACEMENT ============ -->
  <header class="strip reveal">
    <div class="titleblock">
      <h1>Smart Golf Bag</h1>
      <p class="byline">Jonathan Cheruiyot</p>
    </div>

    <div class="strip-actions">
      <!-- TODO: replace # with the URL of your write-up -->
      <a href="#writeup">Project write-up</a>
      <button
        class="harness-btn"
        aria-haspopup="dialog"
        onclick={() => (showInfo = true)}
      >
        Info
      </button>
      <!-- The placement drawing stays visible as a small figure so the
           "where does this live on the bag" answer is always one click away
           without taking space from the devices. -->
      <button
        class="harness-btn figure"
        aria-haspopup="dialog"
        onclick={() => (showPlacement = true)}
      >
        <BagGraphic compact />
        <span class="figure-caption">Fig. 1<br />Placement</span>
      </button>
    </div>
  </header>

  <!-- ================= CENTRE STAGE: THE TWO DEVICES ================= -->
  <section class="stage" aria-label="Device UI">
    <div class="bag-col">
      <WorkbenchLabel label="Bag display" note="side pocket" />
      <div class="reveal" style="--reveal-step: 1">
        <BagMount
          {round}
          alertsOn={bag.alertsOn}
          onToggleRound={toggleRound}
          onNextHole={nextHole}
          onToggleAlerts={toggleAlerts}
        >
          <BagDisplay
            clubs={loadedClubs}
            {round}
            alertsOn={bag.alertsOn}
            {inBagCount}
            {clubsOut}
            {leftBehind}
            {holeCard}
          />
        </BagMount>
      </div>
    </div>

    <div class="pair-col">
      <PairingLine paired={phone.paired} {relaying} />
    </div>

    <div class="phone-col">
      <WorkbenchLabel label="Phone" note="paired device" />
      <div class="reveal" style="--reveal-step: 2">
        <PhoneShell
          paired={phone.paired}
          tab={phone.tab}
          onTab={setPhoneTab}
          notifications={phone.notifications}
          {clubs}
          {round}
          loadedCount={loadedClubs.length}
          {inBagCount}
          {clubsOut}
          {leftBehind}
          alertsOn={bag.alertsOn}
          onToggleRound={toggleRound}
          onNextHole={nextHole}
          onToggleAlerts={toggleAlerts}
          onToggleLoaded={toggleLoaded}
        />
      </div>
    </div>
  </section>

  <!-- ================= BOTTOM BAR: TESTING UI ================= -->
  <section class="testbar reveal" style="--reveal-step: 3" aria-label="Test panel">
    <SimProgress
      running={sim.running}
      step={sim.step}
      total={SCRIPT.length}
      strokes={round.strokes}
      caption={simCaption}
    />
    <h2 class="testbar-label">
      Test panel
      <span class="testbar-note">simulated sensors</span>
    </h2>
    <TestPanel
      clubs={loadedClubs}
      distance={bag.distanceFromGolfer}
      running={sim.running}
      onPull={pullClub}
      onReturn={returnClub}
      onDistance={setDistance}
      onReset={resetBag}
      onPlay={playSim}
      onStop={stopSim}
    />
  </section>
</main>

<WorkbenchOverlay
  title="Info"
  open={showInfo}
  onClose={() => (showInfo = false)}
>
  <div class="info-text">
    <p>
      This page is a mock-up of a golf bag that knows which clubs are in it,
      and a phone app that knows where the bag is. Nothing here is a real
      sensor. The bar along the bottom of the page stands in for the
      hardware, and everything on the two devices reacts to it.
    </p>

    <h3>Test controls</h3>
    <dl>
      <dt>Club buttons (DR, 3W, 7i and so on)</dt>
      <dd>
        Each one is the sensor in one slot at the top of the bag. Press it to
        lift that club out; press it again to put it back, which counts as
        one stroke during a round. A filled button means the club is out.
      </dd>
      <dt>Walk away</dt>
      <dd>
        How far you are from the bag, in metres. Past 30 metres during a
        round, the phone warns that you have left the bag itself. The bag
        shows nothing, because nobody is there to read it.
      </dd>
      <dt>Play round</dt>
      <dd>
        Plays a scripted nine holes by itself, about 55 seconds, with a
        caption above the bar narrating each shot. A sand wedge and then the
        putter get left on a green, and at the end the bag itself is left
        behind, so it shows both alerts. While it plays the button reads
        Stop, and the other test controls are switched off.
      </dd>
      <dt>Reset</dt>
      <dd>
        Puts every club back, ends the round, clears the round's history and
        walks you back to the bag.
      </dd>
    </dl>

    <h3>To see the main feature by hand</h3>
    <p>
      Press the Start Round key under the bag's screen, lift the 7 Iron, then
      press Next Hole on the strap without returning it. The bag flags the
      club you walked away from and names the hole you left it on, and a
      moment later the phone is told.
    </p>

    <h3>On the devices</h3>
    <p>
      The bag has three physical keys: Start or End Round, Next Hole, and
      Alerts. The phone's Round tab repeats them. Its Setup tab chooses
      today's 14 clubs, and its Summary tab shows the round's shots.
    </p>
  </div>
</WorkbenchOverlay>

<WorkbenchOverlay
  title="Fig. 1, placement on the bag"
  open={showPlacement}
  onClose={() => (showPlacement = false)}
>
  <BagGraphic />
</WorkbenchOverlay>

<style>
  /* 1344 is 56 squares of the 24px graph grid. With 24px side padding the
     page is 1392 wide, which leaves room for a scrollbar in a 1440 window. */
  main {
    box-sizing: content-box;
    width: 1344px;
    margin: 0 auto;
    padding: var(--space-5);
    color: var(--table-ink);
  }

  /* Slim on purpose: the title block of a drawing sheet, not a hero banner. */
  .strip {
    display: flex;
    align-items: center;
    justify-content: space-between;
    box-sizing: border-box;
    height: 72px;
    padding-bottom: var(--space-3);
    border-bottom: 1px solid var(--table-soft);
  }

  .titleblock {
    display: flex;
    align-items: baseline;
    gap: var(--space-4);
  }

  h1 {
    margin: 0;
    font-family: var(--font-serif);
    font-size: 32px;
    font-weight: 600;
    line-height: 48px;
  }

  .byline {
    margin: 0;
    font-size: 16px;
    color: var(--table-soft);
  }

  .strip-actions {
    display: flex;
    align-items: center;
    gap: var(--space-4);
  }

  a {
    /* Padding makes the link a 44px target without moving the text. */
    padding: var(--space-3) 0;
    color: var(--table-ink);
    text-underline-offset: 4px;
  }

  a:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: 2px;
  }

  .figure {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    height: 60px;
    padding: 0 var(--space-3) 0 var(--space-2);
    text-align: left;
  }

  .figure-caption {
    line-height: 16px;
  }

  /* The devices are centred with wide margins either side: the empty table
     around them is what makes them read as the subject of the page. */
  .stage {
    display: flex;
    justify-content: center;
    padding: var(--space-5) 0;
  }

  /* The mount is the same 640px height as the phone, so the two devices
     stand level on the table. */
  .bag-col {
    width: 576px;
  }

  /* The 96px gap between the devices, with the pairing line across it at
     the height of the bag's club count, so the line appears to leave the
     bag from the number the phone mirrors. */
  .pair-col {
    width: 96px;
    padding-top: 212px;
  }

  /* 296 by 640, close to the 19.5:9 of a current phone. */
  .phone-col {
    width: 296px;
  }

  /* Quieter than everything above it: smaller type and the soft ink, because
     these controls are scaffolding for the demo, not part of the product. */
  .testbar {
    /* Anchors the simulation readout, which is drawn over the top rule. */
    position: relative;
    display: flex;
    align-items: center;
    gap: var(--space-4);
    padding-top: var(--space-3);
    border-top: 1px solid var(--table-soft);
  }

  .testbar-label {
    flex-shrink: 0;
    width: 136px;
    margin: 0;
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    line-height: 16px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--table-soft);
  }

  .testbar-note {
    display: block;
    font-weight: 400;
    text-transform: none;
  }

  .info-text {
    max-width: 520px;
    font-size: 14px;
    line-height: 20px;
  }

  .info-text p,
  .info-text dl {
    margin: 0 0 var(--space-3);
  }

  .info-text p:last-child {
    margin: 0;
  }

  .info-text h3 {
    margin: var(--space-4) 0 var(--space-2);
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }

  .info-text dt {
    font-weight: 600;
  }

  .info-text dd {
    margin: 0 0 var(--space-2);
    color: var(--table-soft);
  }
</style>
