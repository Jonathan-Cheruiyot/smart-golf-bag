<script>
  // The bag display is a printed instrument, not a screen. It lives in direct
  // sun on a small battery and gets a two-second glance from a walking
  // golfer, so it is drawn as an e-ink panel: paper ground, ink numerals, one
  // red for alarms, hairline rules, square corners, no shadows.
  //
  // Reading order, top to bottom, is the order of urgency at a glance:
  // where am I (hole), is everything here (the count), what is wrong (the
  // message), which one (the rack). The controls sit last because they are
  // pressed, not read.
  //
  // Motion: everything changes in one frame, as e-ink does. The only effect
  // is the refresh flash on the count, which inverts for 90ms when the number
  // changes and so draws the eye to exactly what changed.
  import { MediaQuery } from "svelte/reactivity";
  import ClubRack from "./ClubRack.svelte";

  let {
    clubs,
    round,
    alertsOn,
    inBagCount,
    clubsOut,
    leftBehind,
    onToggleRound,
    onNextHole,
    onToggleAlerts
  } = $props();

  // Two digits always, so the header does not shift between hole 9 and 10.
  let holeText = $derived(
    round.active ? String(round.hole).padStart(2, "0") : "--"
  );

  // One message at a time, highest priority first: left behind, club in
  // hand, all present, idle. With several clubs involved it gives the number
  // and lets the rack say which, to stay inside a two-second glance.
  let message = $derived.by(() => {
    if (alertsOn && leftBehind.length === 1) {
      const club = leftBehind[0];
      return { alert: true, text: `Left behind: ${club.name}, hole ${club.pulledAtHole}` };
    }
    if (alertsOn && leftBehind.length > 1) {
      return { alert: true, text: `Left behind: ${leftBehind.length} clubs` };
    }
    if (clubsOut.length === 1) {
      return { alert: false, text: `In hand: ${clubsOut[0].name}` };
    }
    if (clubsOut.length > 1) {
      return { alert: false, text: `In hand: ${clubsOut.length} clubs` };
    }
    if (round.active) {
      return { alert: false, text: `All ${clubs.length} clubs in bag` };
    }
    return { alert: false, text: "Ready. Press Start Round" };
  });

  // The e-ink refresh flash. Purely visual, so it is local state.
  const reducedMotion = new MediaQuery("(prefers-reduced-motion: reduce)");
  let flashing = $state(false);
  let previousCount;

  $effect(() => {
    const count = inBagCount;
    const changed = previousCount !== undefined && count !== previousCount;
    previousCount = count;
    // A flash is a blink, so it is skipped for people who ask for less motion.
    if (!changed || reducedMotion.current) return;

    flashing = true;
    const timer = setTimeout(() => (flashing = false), 90);
    return () => clearTimeout(timer);
  });
</script>

<!-- The bezel is the physical housing, so it alone is rounded. -->
<div class="bezel">
  <div class="screen">
    <div class="status">
      <span>Hole {holeText}</span>
      <span>{round.active ? "Round active" : "Idle"}</span>
    </div>

    <p class="count">
      <span class="number" class:flash={flashing}>{inBagCount}</span>
      <span class="of">/ {clubs.length}</span>
      <span class="count-label">clubs in bag</span>
    </p>

    <!-- role="status" so a screen reader announces the message when it
         changes, the way the golfer's eye is drawn to it. -->
    <p class="message" class:alert={message.alert} role="status">
      {message.text}
    </p>

    <ClubRack {clubs} hole={round.hole} />

    <div class="controls">
      <button onclick={onToggleRound}>
        {round.active ? "End Round" : "Start Round"}
      </button>
      <button onclick={onNextHole} disabled={!round.active}>Next Hole</button>
      <button aria-pressed={alertsOn} onclick={onToggleAlerts}>
        Alerts {alertsOn ? "On" : "Off"}
      </button>
    </div>
  </div>
</div>

<style>
  .bezel {
    padding: var(--space-4);
    border-radius: 16px;
    background: var(--bezel);
  }

  .screen {
    display: flex;
    flex-direction: column;
    gap: var(--space-5);
    padding: var(--space-5);
    background: var(--paper);
    color: var(--ink);
    font-family: var(--font-mono);
    font-variant-numeric: tabular-nums;
  }

  .status {
    display: flex;
    justify-content: space-between;
    padding-bottom: var(--space-3);
    border-bottom: 1px solid var(--ink);
    font-size: 12px;
    font-weight: 500;
    line-height: 16px;
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }

  .count {
    display: flex;
    align-items: baseline;
    gap: var(--space-3);
    margin: 0;
  }

  .number {
    /* Padding gives the inverted flash a deliberate block shape; the negative
       margin keeps the numeral on the left edge of the column. */
    margin-left: calc(var(--space-2) * -1);
    padding: 0 var(--space-2);
    font-size: 72px;
    font-weight: 600;
    line-height: 80px;
    /* The colour swaps in steps, it does not fade through grey. */
    transition:
      background-color var(--t-flash) var(--e-snap),
      color var(--t-flash) var(--e-snap);
  }

  .number.flash {
    background: var(--ink);
    color: var(--paper);
  }

  .of {
    font-size: 24px;
    color: var(--ink-soft);
  }

  .count-label {
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }

  /* Same box in every state, so the rack below never jumps. */
  .message {
    display: flex;
    align-items: center;
    height: 48px;
    margin: 0;
    padding: 0 var(--space-3);
    overflow: hidden;
    background: var(--paper-sunk);
    font-size: 14px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    white-space: nowrap;
  }

  /* The alert is a solid bar with reversed text: a different shape as well as
     a different colour, so it reads in grayscale too. It arrives in a 90ms
     step, the one concession MOTION.md allows, with no slide and no fade. */
  .message.alert {
    background: var(--flag);
    color: var(--paper);
    font-weight: 600;
    animation: alert-in var(--t-flash) var(--e-snap);
  }

  @keyframes alert-in {
    from {
      opacity: 0;
    }
  }

  .controls {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: var(--space-2);
  }

  /* 56px tall: comfortably over the 44px minimum for a gloved hand. */
  button {
    height: 56px;
    padding: 0;
    border: 1px solid var(--ink);
    border-radius: 0;
    background: var(--paper);
    color: var(--ink);
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    cursor: pointer;
  }

  /* No transitions: e-ink has no in-between frames. Hover is a tone change
     and a press inverts, both instantly. */
  button:hover:not(:disabled) {
    background: var(--paper-sunk);
  }

  button:active:not(:disabled) {
    background: var(--ink);
    color: var(--paper);
  }

  /* Ink, not red: on this screen red is reserved for alerts. */
  button:focus-visible {
    outline: 2px solid var(--ink);
    outline-offset: 2px;
  }

  /* Dashed and soft instead of faded, because e-ink has no half-opacity. */
  button:disabled {
    border-style: dashed;
    border-color: var(--ink-soft);
    color: var(--ink-soft);
    cursor: not-allowed;
  }
</style>
