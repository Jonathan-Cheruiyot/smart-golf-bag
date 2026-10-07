<script>
  // The simulation's readout: which step of the scripted round is playing,
  // and one line saying what the golfer just did. Without the caption the
  // demo is two screens changing by themselves; with it, a viewer can follow
  // cause and effect without anyone narrating.
  //
  // It takes no room of its own. The bar is drawn over the test bar's top
  // rule and the caption sits in the empty margin just above it, so the
  // devices keep all their space. Both are positioned against the test bar,
  // which App sets to position: relative.
  //
  // Motion: the bar may tween, the one harness animation after load that
  // MOTION.md allows. It scales with transform, never width, because
  // transform is cheap for the browser to animate.
  let { running, step, total, caption } = $props();

  let stateWord = $derived(
    running ? "Playing" : step === total ? "Finished" : "Stopped"
  );
</script>

{#if step > 0}
  <p class="caption">
    <span class="step">{stateWord} {String(step).padStart(2, "0")} / {total}</span>
    {caption}
  </p>
{/if}

<div
  class="track"
  role="progressbar"
  aria-label="Round simulation progress"
  aria-valuemin="0"
  aria-valuemax={total}
  aria-valuenow={step}
  aria-valuetext={step > 0 ? `Step ${step} of ${total}: ${caption}` : "Not started"}
>
  <div class="bar" style="transform: scaleX({step / total})"></div>
</div>

<style>
  .caption {
    position: absolute;
    bottom: 100%;
    left: 0;
    margin: 0 0 var(--space-1);
    font-family: var(--font-mono);
    font-size: 11px;
    line-height: 16px;
    letter-spacing: 0.04em;
    white-space: nowrap;
    color: var(--table-ink);
  }

  .step {
    margin-right: var(--space-3);
    font-variant-numeric: tabular-nums;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--table-soft);
  }

  .track {
    position: absolute;
    top: -2px;
    right: 0;
    left: 0;
    height: 3px;
  }

  .bar {
    height: 100%;
    background: var(--table-ink);
    transform-origin: left;
    transition: transform var(--t-slow) var(--e-draft);
  }
</style>
