<script>
  // The dashed line across the gap between the two devices. It is the same
  // pairing line as in the placement figure, brought onto the stage so the
  // connection between bag and phone is something you can see.
  //
  // It does two jobs. At rest it drifts slowly, which says "these two are
  // talking". And when the bag raises a left-behind alert it flashes once,
  // in the 200ms between the bag's alert and the phone's notification, so
  // the eye follows the alert across: bag, line, phone. That makes the
  // cause visible instead of two screens changing at once.
  //
  // The flash is a solid, heavier, red line: a change of shape as well as
  // colour. It is skipped for people who ask for reduced motion.
  let { paired, relaying } = $props();

  const uid = $props.id();
</script>

{#if paired}
  <svg
    width="96"
    height="24"
    viewBox="0 0 96 24"
    role="img"
    aria-label="The bag and the phone are paired"
  >
    <!-- Draws on once after the devices have appeared, by growing a solid
         mask line over the dashed one. -->
    <mask id="{uid}-draw" maskUnits="userSpaceOnUse" x="0" y="0" width="96" height="24">
      <line class="pair-draw" x1="0" y1="12" x2="96" y2="12" stroke-width="24" pathLength="1" />
    </mask>
    <line class="pair-line" class:relaying x1="0" y1="12" x2="96" y2="12" mask="url(#{uid}-draw)" />
  </svg>
{/if}

<style>
  svg {
    display: block;
  }

  /* Waits for the staged reveal to bring in the phone before drawing. */
  .pair-draw {
    animation-delay: 560ms;
  }

  @media (prefers-reduced-motion: no-preference) {
    .pair-line.relaying {
      stroke: var(--flag);
      stroke-width: 3;
      stroke-dasharray: none;
    }
  }
</style>
