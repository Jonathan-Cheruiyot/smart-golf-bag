<script>
  // ===== PROPS =====
  // These values are passed in from App.svelte. The bag's state lives in
  // App so that the Testing UI buttons can change it.
  let {
    clubs,
    hole,
    roundActive,
    alertsOn,
    shots,
    onStartStop,
    onNextHole,
    onToggleAlerts
  } = $props();

  // ===== DERIVED VALUES =====
  // $derived recomputes automatically whenever clubs or hole change.
  let clubsOut = $derived(clubs.filter((c) => !c.inBag));
  let inBagCount = $derived(clubs.length - clubsOut.length);

  // A club counts as "left behind" if it was pulled on an EARLIER hole
  // and never put back. This is the core feature of the interface.
  let leftBehind = $derived(clubsOut.filter((c) => c.pulledAtHole < hole));

  // Turn a list of clubs into a readable string like "7 Iron, Putter"
  let leftBehindNames = $derived(leftBehind.map((c) => c.name).join(", "));
  let inHandNames = $derived(clubsOut.map((c) => c.name).join(", "));
</script>

<div class="panel">
  <!-- Status bar: small, always-on indicators -->
  <div class="status">
    <span>HOLE {roundActive ? hole : "--"}</span>
    <span class:live={roundActive}>{roundActive ? "ROUND ACTIVE" : "IDLE"}</span>
    <span>{shots} shots</span>
  </div>

  <!-- Primary indicator: the club count, the biggest thing on screen -->
  <div class="count" class:warn={inBagCount < clubs.length}>
    <span class="big">{inBagCount}</span>
    <span class="small">of {clubs.length} clubs in the bag</span>
  </div>

  <!-- Message area. Only one of these shows at a time. -->
  {#if alertsOn && leftBehind.length > 0}
    <div class="msg alert">
      Left behind on hole {leftBehind[0].pulledAtHole}: <strong>{leftBehindNames}</strong>
    </div>
  {:else if clubsOut.length > 0}
    <div class="msg notice">In hand: <strong>{inHandNames}</strong></div>
  {:else if roundActive}
    <div class="msg ok">All {clubs.length} clubs accounted for</div>
  {:else}
    <div class="msg notice">Ready. Press Start Round to begin tracking.</div>
  {/if}

  <!-- Club rack: one slot per club, so the golfer can see WHICH is missing -->
  <div class="rack">
    {#each clubs as club}
      <div
        class="slot"
        class:out={!club.inBag}
        class:missing={!club.inBag && club.pulledAtHole < hole}
      >
        {club.short}
      </div>
    {/each}
  </div>

  <!-- Controls that live on the physical bag -->
  <div class="controls">
    <button onclick={onStartStop}>
      {roundActive ? "End Round" : "Start Round"}
    </button>
    <button onclick={onNextHole} disabled={!roundActive}>Next Hole</button>
    <button class:on={alertsOn} onclick={onToggleAlerts}>
      Alerts: {alertsOn ? "On" : "Off"}
    </button>
  </div>
</div>

<style>
  .panel {
    width: 340px;
    flex-shrink: 0;
    box-sizing: border-box;
    background: #14301f;
    border: 6px solid #0b1d13;
    border-radius: 20px;
    padding: 18px;
    color: #eaf3ec;
    display: flex;
    flex-direction: column;
    gap: 14px;
  }

  .status {
    display: flex;
    justify-content: space-between;
    font-family: ui-monospace, monospace;
    font-size: 12px;
    letter-spacing: 0.08em;
    color: #a9c7b3;
    padding-bottom: 10px;
    border-bottom: 1px solid #2c5a40;
  }

  .status .live {
    color: #7ee0a6;
  }

  .count {
    display: flex;
    align-items: baseline;
    gap: 8px;
  }

  .count .big {
    font-size: 60px;
    font-weight: 700;
    line-height: 1;
  }

  .count .small {
    font-size: 15px;
    color: #a9c7b3;
  }

  .count.warn .big {
    color: #ffd37a;
  }

  .msg {
    padding: 12px 14px;
    border-radius: 10px;
    font-size: 15px;
    line-height: 1.4;
    min-height: 22px;
  }

  .msg.notice {
    background: #1d4530;
  }

  .msg.ok {
    background: #1d4530;
    color: #9fefc0;
  }

  .msg.alert {
    background: #6b2620;
    border: 1px solid #ff8a7a;
    color: #ffede9;
  }

  .rack {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 6px;
  }

  .slot {
    height: 52px;
    box-sizing: border-box;
    border-radius: 6px;
    background: #2c5a40;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: ui-monospace, monospace;
    font-size: 12px;
  }

  .slot.out {
    background: transparent;
    border: 1px dashed #7faa90;
    color: #7faa90;
  }

  .slot.missing {
    border: 2px dashed #ff8a7a;
    color: #ff8a7a;
  }

  .controls {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;
  }

  .controls button {
    height: 48px;
    border: none;
    border-radius: 10px;
    background: #2c5a40;
    color: #ffffff;
    font-size: 13px;
    font-weight: 600;
    cursor: pointer;
  }

  .controls button:hover:not(:disabled) {
    background: #3a7a55;
  }

  .controls button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  .controls button.on {
    background: #3a7a55;
  }
</style>
