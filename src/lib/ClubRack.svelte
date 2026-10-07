<script>
  // The rack answers the one question the count cannot: WHICH club is
  // missing. One slot per loaded club, 7 across, so the golfer matches the
  // screen to the top of the bag at a glance.
  //
  // Three states, each readable without colour, so the panel still works as
  // a photocopy and for colourblind golfers:
  //   in the bag    a filled ink block, code reversed out
  //   out of bag    an empty outline
  //   left behind   an outline PLUS a filled corner notch
  // The red on a left-behind slot is reinforcement, never the only signal.
  //
  // Motion: slots swap fill with the e-ink step easing. A club leaving does
  // not fade out, it is simply gone on the next refresh.
  let { clubs, hole } = $props();
</script>

<ul class="rack">
  {#each clubs as club (club.id)}
    {@const out = !club.inBag}
    {@const left = out && club.pulledAtHole < hole}
    <li class="slot" class:out class:left>
      <span aria-hidden="true">{club.short}</span>
      <span class="sr">
        {club.name}:
        {left
          ? `left behind on hole ${club.pulledAtHole}`
          : out
            ? "out of the bag"
            : "in the bag"}
      </span>
    </li>
  {/each}
</ul>

<style>
  .rack {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: var(--space-2);
    margin: 0;
    padding: 0;
    list-style: none;
  }

  .slot {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    box-sizing: border-box;
    height: 64px;
    border: 1px solid var(--ink);
    background: var(--ink);
    color: var(--paper);
    font-family: var(--font-mono);
    font-size: 14px;
    font-weight: 500;
    transition:
      background-color var(--t-flash) var(--e-snap),
      color var(--t-flash) var(--e-snap),
      border-color var(--t-flash) var(--e-snap);
  }

  .slot.out {
    background: var(--paper);
    color: var(--ink);
  }

  .slot.left {
    border-width: 2px;
    border-color: var(--flag);
    color: var(--flag);
  }

  /* The corner notch: a filled triangle, so "left behind" has its own shape. */
  .slot.left::after {
    content: "";
    position: absolute;
    top: 0;
    right: 0;
    width: 16px;
    height: 16px;
    background: var(--flag);
    clip-path: polygon(0 0, 100% 0, 100% 100%);
  }

  /* Text for screen readers only: the slot states are otherwise visual. */
  .sr {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
  }
</style>
