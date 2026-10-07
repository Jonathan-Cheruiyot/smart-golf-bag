<script>
  // The Round tab is the bag display's companion. It repeats the two things
  // the bag shows (the count and whether something is wrong) so the golfer
  // can trust that both devices agree, then adds what a two-second glance at
  // the bag has no room for: exactly which clubs are out, the hole each one
  // was pulled on, and the running stroke count.
  //
  // TO PIN, the yards left to the hole, is here and deliberately not on the
  // bag. The phone travels with the golfer, so it can know where he is
  // standing. The bag is sitting back on the cart path and cannot know where
  // the ball is. The bag shows only the hole's printed yardage.
  //
  // It also repeats the bag's three controls. The golfer may be at the bag
  // or may have walked ahead with only the phone, and either device should
  // be able to start the round, move to the next hole, or silence alerts.
  // Both call the same functions in App, so they cannot drift apart.
  //
  // Motion: the count settles with a small spring when it changes. That is
  // the opposite of the bag's hard e-ink flash on purpose, because an OLED
  // phone really does move like this and an e-ink panel really does not.
  // When a club comes back, the rest of the clubs-out list glides up to
  // close the gap (flip) instead of jumping.
  import { Spring } from "svelte/motion";
  import { flip } from "svelte/animate";
  import { fade } from "svelte/transition";
  import { MediaQuery } from "svelte/reactivity";

  let {
    round,
    loadedCount,
    inBagCount,
    clubsOut,
    leftBehind,
    alertsOn,
    onToggleRound,
    onNextHole,
    onToggleAlerts
  } = $props();

  // The spring chases the count. Its distance from the real value becomes a
  // small vertical offset, so the numeral drops in and settles. Only
  // transform is animated, which is cheap, and the digit itself is always
  // the true value so nothing ever reads wrong mid-animation.
  const reducedMotion = new MediaQuery("(prefers-reduced-motion: reduce)");
  const settle = Spring.of(() => inBagCount, { stiffness: 0.2, damping: 0.45 });
  let flipMs = $derived(reducedMotion.current ? 0 : 260);
  let fadeMs = $derived(reducedMotion.current ? 0 : 140);
  let offset = $derived(
    reducedMotion.current ? 0 : (settle.current - inBagCount) * 8
  );

  // Yards from the fairway, feet near the green, as golfers say it. Unknown
  // (no position for this shot) reads as two dashes, never as a guess.
  let toPin = $derived.by(() => {
    if (round.toPin === null) return "--";
    if (round.toPin === 0) return "In the hole";
    if (round.toPin < 20) return `${Math.round(round.toPin * 3)} ft`;
    return `${Math.round(round.toPin)} yds`;
  });

  // Same precedence as the bag's message line, worded for the phone.
  let status = $derived.by(() => {
    if (alertsOn && leftBehind.length > 0) {
      const n = leftBehind.length;
      return { alert: true, text: `${n} ${n === 1 ? "club" : "clubs"} left behind` };
    }
    if (clubsOut.length > 0) {
      const n = clubsOut.length;
      return { alert: false, text: `${n} ${n === 1 ? "club" : "clubs"} in hand` };
    }
    if (round.active) return { alert: false, text: "All clubs in the bag" };
    return { alert: false, text: "Round not started" };
  });
</script>

<div class="round">
  <div class="meta">
    <span>Hole {round.active ? String(round.hole).padStart(2, "0") : "--"}</span>
    <span>{round.active ? "Round active" : "Idle"}</span>
  </div>

  <div class="headline">
    <p class="count">
      <span class="number" style="transform: translateY({offset}px)">{inBagCount}</span>
      <span class="of">/ {loadedCount} in bag</span>
    </p>
    <p class="to-pin">
      <span class="to-pin-label">To pin</span>
      <span class="to-pin-value">{toPin}</span>
    </p>
  </div>

  <!-- The square marker fills when something is wrong, so the state is a
       shape as well as a colour. -->
  <p class="status" class:alert={status.alert}>
    <span class="marker" aria-hidden="true"></span>
    {status.text}
  </p>

  <section aria-labelledby="out-heading">
    <h3 id="out-heading">Clubs out</h3>
    {#if clubsOut.length === 0}
      <p class="empty">None. Every club is in its slot.</p>
    {:else}
      <ul>
        {#each clubsOut as club (club.id)}
          {@const left = club.pulledAtHole < round.hole}
          <li class:left animate:flip={{ duration: flipMs }} transition:fade={{ duration: fadeMs }}>
            <span class="name">{club.name}</span>
            <span class="where">
              {left ? "Left behind" : "In hand"}, hole {club.pulledAtHole}
            </span>
          </li>
        {/each}
      </ul>
    {/if}
  </section>

  <section class="shots">
    <h3>Strokes this round</h3>
    <p class="shot-count">{round.strokes}</p>
  </section>

  <section>
    <div class="controls" role="group" aria-label="Bag controls">
      <button onclick={onToggleRound}>
        {round.active ? "End" : "Start"}<br />round
      </button>
      <button onclick={onNextHole} disabled={!round.active}>Next<br />hole</button>
      <button aria-pressed={alertsOn} onclick={onToggleAlerts}>
        Alerts<br />{alertsOn ? "on" : "off"}
      </button>
    </div>
  </section>
</div>

<style>
  .round {
    display: flex;
    flex-direction: column;
    /* Tight on purpose: the tab has to hold three clubs out, both alerts and
       the controls at once without scrolling. */
    gap: var(--space-2);
  }

  .meta {
    display: flex;
    justify-content: space-between;
    font-family: var(--font-mono);
    font-size: 11px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  .count {
    display: flex;
    align-items: baseline;
    gap: var(--space-2);
    margin: 0;
    font-family: var(--font-mono);
    font-variant-numeric: tabular-nums;
  }

  .number {
    display: inline-block;
    font-size: 44px;
    font-weight: 600;
    line-height: 48px;
  }

  .of {
    font-size: 14px;
    color: var(--phone-soft);
  }

  /* The count on the left, the distance on the right, on one line so the
     tab gets no taller. */
  .headline {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;
  }

  .to-pin {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    margin: 0 0 var(--space-1);
    font-family: var(--font-mono);
  }

  .to-pin-label {
    font-size: 11px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  .to-pin-value {
    font-size: 16px;
    font-weight: 500;
    font-variant-numeric: tabular-nums;
    line-height: 24px;
  }

  .status {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    margin: 0;
    font-size: 14px;
    line-height: 20px;
  }

  .marker {
    box-sizing: border-box;
    width: 10px;
    height: 10px;
    border: 1px solid var(--phone-soft);
  }

  .status.alert {
    font-weight: 600;
  }

  .status.alert .marker {
    border-color: var(--flag);
    background: var(--flag);
  }

  section {
    padding-top: var(--space-2);
    border-top: 1px solid var(--phone-rule);
  }

  h3 {
    margin: 0 0 var(--space-2);
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    padding: var(--space-1) 0 var(--space-1) var(--space-2);
    border-left: 2px solid var(--phone-rule);
    font-size: 14px;
    line-height: 20px;
  }

  li + li {
    margin-top: var(--space-1);
  }

  .where {
    font-family: var(--font-mono);
    font-size: 11px;
    color: var(--phone-soft);
  }

  /* Left behind: the words say it, the bar and the weight back it up. */
  li.left {
    border-left-color: var(--flag);
  }

  li.left .name {
    font-weight: 600;
  }

  li.left .where {
    color: var(--phone-text);
  }

  .empty {
    margin: 0;
    font-size: 14px;
    line-height: 20px;
    color: var(--phone-soft);
  }

  /* One line, label left and number right, to leave room for the controls. */
  .shots {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
  }

  .shots h3 {
    margin: 0;
  }

  .shot-count {
    margin: 0;
    font-family: var(--font-mono);
    font-size: 16px;
    font-weight: 500;
    font-variant-numeric: tabular-nums;
  }

  .controls {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: var(--space-2);
  }

  /* Rounded by 4px and softly ruled: these are phone buttons, and they are
     deliberately not the square inked keys of the bag. */
  button {
    height: 48px;
    padding: 0;
    border: 1px solid var(--phone-soft);
    border-radius: 4px;
    background: var(--oled-raised);
    color: var(--phone-text);
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    line-height: 14px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    cursor: pointer;
    transition:
      background-color var(--t-quick) var(--e-enter),
      border-color var(--t-quick) var(--e-enter);
  }

  button:hover:not(:disabled) {
    border-color: var(--phone-text);
  }

  button:active:not(:disabled) {
    transform: translateY(1px);
    transition: none;
  }

  button:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: 2px;
  }

  button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }
</style>
