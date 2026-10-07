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
  import BagDisplay from "./lib/BagDisplay.svelte";
  import PhoneShell from "./lib/PhoneShell.svelte";
  import TestPanel from "./lib/TestPanel.svelte";

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
      pulledAtHole: null, // hole number it was taken out on
      usedOnHoles: [] // for the phone summary
    };
  }

  // The 18 clubs the golfer owns: a standard set, no brands. The rules allow
  // 14 in the bag, so four start the day at home and the phone's Setup tab
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
    club("lw", "Lob Wedge", "LW", "wedge", false),
    club("w60", "60° Wedge", "60", "wedge", false),
    club("pt", "Putter", "PT", "putter")
  ]);

  let round = $state({
    active: false,
    hole: 1, // 1 to 18
    shots: 0,
    log: [] // { hole, clubId } appended on each return
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
        text: leftBehind.map((c) => `${c.name} (hole ${c.pulledAtHole})`).join(", ") + " are not in the bag."
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
      const timer = setTimeout(() => phone.notifications.unshift(...relayed), RELAY_MS);
      return () => clearTimeout(timer);
    });
  });

  // ===== CONTROLS ON THE BAG, mirrored on the phone's Round tab =====

  function toggleRound() {
    round.active = !round.active;
    if (round.active) {
      round.hole = 1;
      round.shots = 0;
      round.log = [];
      for (const c of clubs) {
        c.usedOnHoles = [];
        // A club already in hand when the round starts counts as pulled on
        // hole 1, so it is not flagged until the golfer moves on without it.
        if (!c.inBag) c.pulledAtHole = 1;
      }
    }
  }

  function nextHole() {
    if (round.active && round.hole < 18) round.hole += 1;
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

  // A return is what counts as a shot: the club went out, was used, and came
  // back. It is logged against the hole it was pulled on, not the hole it was
  // returned on, so a club left behind is still credited to the right hole.
  function returnClub(id) {
    const c = clubs.find((c) => c.id === id);
    if (c && !c.inBag) {
      if (round.active) {
        round.shots += 1;
        round.log.push({ hole: c.pulledAtHole, clubId: c.id });
        if (!c.usedOnHoles.includes(c.pulledAtHole)) {
          c.usedOnHoles.push(c.pulledAtHole);
        }
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
      c.usedOnHoles = [];
    }
    round.active = false;
    round.hole = 1;
    round.shots = 0;
    round.log = [];
    bag.distanceFromGolfer = 0;
  }
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
        <BagDisplay
          clubs={loadedClubs}
          {round}
          alertsOn={bag.alertsOn}
          {inBagCount}
          {clubsOut}
          {leftBehind}
          onToggleRound={toggleRound}
          onNextHole={nextHole}
          onToggleAlerts={toggleAlerts}
        />
      </div>
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
    <h2 class="testbar-label">
      Test panel
      <span class="testbar-note">simulated sensors</span>
    </h2>
    <TestPanel
      clubs={loadedClubs}
      distance={bag.distanceFromGolfer}
      onPull={pullClub}
      onReturn={returnClub}
      onDistance={setDistance}
      onReset={resetBag}
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
      The bar along the bottom stands in for the bag's sensors. Each club
      button is one slot in the top of the bag: press it to lift that club
      out, press it again to put it back.
    </p>
    <p>
      To see the main feature, press Start Round on the bag display, lift the
      7 Iron, then press Next Hole without returning it. The bag flags the
      club you walked away from and names the hole you left it on.
    </p>
    <p>
      The Walk away slider is how far you are from the bag. Move it past 30
      metres during a round and the phone warns you that you left the bag
      itself. The bag shows nothing, because nobody is there to read it.
    </p>
    <p>Reset puts every club back, ends the round and walks you back to the bag.</p>
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
    gap: 96px;
    padding: var(--space-5) 0;
  }

  .bag-col {
    width: 480px;
  }

  /* 300 by 620 is the phone body size fixed in the spec. */
  .phone-col {
    width: 300px;
  }

  /* Quieter than everything above it: smaller type and the soft ink, because
     these controls are scaffolding for the demo, not part of the product. */
  .testbar {
    display: flex;
    align-items: center;
    gap: var(--space-5);
    padding-top: var(--space-3);
    border-top: 1px solid var(--table-soft);
  }

  .testbar-label {
    flex-shrink: 0;
    width: 144px;
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
    max-width: 480px;
    font-size: 14px;
    line-height: 20px;
  }

  .info-text p {
    margin: 0 0 var(--space-3);
  }

  .info-text p:last-child {
    margin: 0;
  }
</style>
