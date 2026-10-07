<script>
  // The test panel stands in for the hardware this mock-up does not have.
  // Each club button is one slot sensor in the top cuff: press it to lift
  // that club out, press it again to put it back. A pressed button means the
  // club is out, which is what a real sensor would report. The slider is the
  // distance between the golfer's phone and the bag, which a real phone would
  // estimate from the strength of the bag's radio signal.
  //
  // It is deliberately quiet, small muted type in a bar at the bottom,
  // because these controls are scaffolding for the demo and must not compete
  // with the two devices they drive.
  //
  // Play round runs the scripted simulation. While it plays, the manual
  // sensors are disabled so a stray click cannot derail the story; the two
  // devices stay live.
  let {
    clubs,
    distance,
    running,
    onPull,
    onReturn,
    onDistance,
    onReset,
    onPlay,
    onStop
  } = $props();
</script>

<div class="panel">
  <div class="group" role="group" aria-label="Club slot sensors. Pressed means lifted out.">
    <span class="caption" aria-hidden="true">Lift or<br />return</span>
    {#each clubs as club (club.id)}
      <button
        class="sensor"
        aria-label={club.name}
        aria-pressed={!club.inBag}
        disabled={running}
        onclick={() => (club.inBag ? onPull(club.id) : onReturn(club.id))}
      >
        {club.short}
      </button>
    {/each}
  </div>

  <div class="group">
    <label class="caption short" for="distance">Walk<br />away</label>
    <input
      id="distance"
      type="range"
      min="0"
      max="100"
      step="5"
      value={distance}
      disabled={running}
      oninput={(event) => onDistance(Number(event.target.value))}
    />
    <output class="readout" for="distance">{distance} m</output>
  </div>

  <div class="group">
    <button class="play" onclick={running ? onStop : onPlay}>
      {running ? "Stop" : "Play round"}
    </button>
    <button onclick={onReset} disabled={running}>Reset</button>
  </div>
</div>

<style>
  .panel {
    display: flex;
    flex: 1;
    align-items: center;
    justify-content: space-between;
    gap: var(--space-4);
  }

  .group {
    display: flex;
    align-items: center;
    gap: var(--space-1);
  }

  .caption {
    width: 72px;
    font-family: var(--font-mono);
    font-size: 11px;
    line-height: 16px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--table-soft);
  }

  .caption.short {
    width: 48px;
  }

  /* Fixed width so the bar does not shift when the label changes to Stop. */
  .play {
    width: 112px;
  }

  /* 44px tall so the thumb is a full-size target. The colour comes from the
     page's soft ink so the slider stays as quiet as the buttons. */
  input {
    width: 120px;
    height: 44px;
    margin: 0;
    accent-color: var(--table-soft);
    cursor: pointer;
  }

  .readout {
    width: 40px;
    font-family: var(--font-mono);
    font-size: 11px;
    font-variant-numeric: tabular-nums;
    letter-spacing: 0.08em;
    text-align: right;
    color: var(--table-soft);
  }

  /* 44px minimum in both directions, even though the bar is quiet. */
  button {
    min-width: 44px;
    height: 44px;
    padding: 0 var(--space-3);
    border: 1px solid var(--table-soft);
    border-radius: 0;
    background: var(--table);
    color: var(--table-soft);
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    cursor: pointer;
    transition: background-color var(--t-quick) var(--e-enter);
  }

  .sensor {
    padding: 0;
    text-transform: none;
  }

  button:hover:not(:disabled) {
    background: var(--grid);
  }

  /* Lifted out reads as a filled button, so the state is a shape change and
     not only a colour change. */
  .sensor[aria-pressed="true"],
  .sensor[aria-pressed="true"]:hover {
    background: var(--table-soft);
    color: var(--table);
  }

  button:active:not(:disabled) {
    transform: translateY(1px);
    transition: none;
  }

  button:focus-visible,
  input:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: 2px;
  }

  button:disabled,
  input:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }
</style>
