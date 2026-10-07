<script>
  // The bag display does not float in space: it is a manufactured part set
  // into the side pocket of a golf bag. This component draws that mounting
  // as a technical product illustration, in the manner of a patent drawing
  // or a spec sheet, and slots the e-ink display into it.
  //
  // Realism here comes from form and depth, never texture. Four flat tones
  // from the palette say which surface sits proud and which is recessed:
  //   --paper-sunk   the bag's side wall, furthest back
  //   --paper        the pocket panel and the strap, sewn on top of it
  //   --ink-soft     recesses: the channel floor and the button bases
  //   --bezel        the display housing (in BagDisplay)
  // Everything else is ink line: stitches as dashed hairlines along the
  // seams, the zipper as a regular hatch, rivets, and parting lines where
  // separate parts meet. No gradients, no shadows, no fabric grain, so it
  // stays in the same ink-on-paper language as the rest of the page.
  //
  // WHY THE THREE CONTROLS ARE PHYSICAL BUTTONS AND NOT ON THE SCREEN
  // E-ink touch is slow: the panel takes a visible moment to redraw, so a
  // tap gives no instant feedback. Golfers wear gloves, which touch panels
  // read badly. And a bag is used in the rain, where water triggers and
  // blocks touch. A raised key with travel works in all three cases and can
  // be found by feel. So the screen holds indicators only, and the controls
  // are keys in the mount: Start/End Round below the screen, Next Hole and
  // Alerts on the strap edge where the hand already rests when carrying.
  let { round, alertsOn, onToggleRound, onNextHole, onToggleAlerts, children } = $props();
</script>

<div class="mount">
  <!-- The drawing is decoration for sighted users; the display and the
       buttons on top of it are the real content. -->
  <svg class="drawing" width="576" height="640" viewBox="0 0 576 640" aria-hidden="true">
    <!-- bag side wall -->
    <rect class="wall line" x="0.5" y="0.5" width="575" height="639" rx="16" />

    <!-- pocket panel, sewn onto the wall -->
    <rect class="proud line" x="12.5" y="12.5" width="471" height="615" rx="10" />
    <rect class="stitch" x="18.5" y="18.5" width="459" height="603" rx="6" />

    <!-- zipper along the top of the pocket: two rails, teeth as a hatch -->
    <line class="line" x1="36" y1="30.5" x2="448" y2="30.5" />
    <line class="line" x1="36" y1="42.5" x2="448" y2="42.5" />
    <line class="teeth" x1="38" y1="36.5" x2="434" y2="36.5" />
    <rect class="wall line" x="434.5" y="27.5" width="24" height="18" rx="3" />
    <circle class="line" cx="450.5" cy="36.5" r="3" />
    <line class="stitch" x1="24" y1="50.5" x2="472" y2="50.5" />

    <!-- webbing strap, sewn down at both ends with a box stitch and a rivet -->
    <rect class="proud line" x="492.5" y="8.5" width="72" height="623" rx="3" />
    <line class="stitch" x1="497.5" y1="14" x2="497.5" y2="626" />
    <line class="stitch" x1="559.5" y1="14" x2="559.5" y2="626" />

    <rect class="stitch" x="504.5" y="16.5" width="48" height="40" />
    <line class="stitch" x1="504.5" y1="16.5" x2="552.5" y2="56.5" />
    <line class="stitch" x1="552.5" y1="16.5" x2="504.5" y2="56.5" />
    <circle class="wall line" cx="528.5" cy="36.5" r="9" />
    <circle class="line" cx="528.5" cy="36.5" r="3.5" />

    <rect class="stitch" x="504.5" y="583.5" width="48" height="40" />
    <line class="stitch" x1="504.5" y1="583.5" x2="552.5" y2="623.5" />
    <line class="stitch" x1="552.5" y1="583.5" x2="504.5" y2="623.5" />
    <circle class="wall line" cx="528.5" cy="603.5" r="9" />
    <circle class="line" cx="528.5" cy="603.5" r="3.5" />
  </svg>

  <!-- The channel the display drops into. Its floor is the recess tone and
       its top and left edges carry a heavier line: with light from the top
       left, those are the edges that fall in shadow. -->
  <div class="channel">
    {@render children()}
  </div>

  <button class="key round-key" onclick={onToggleRound}>
    <span class="cap">{round.active ? "End Round" : "Start Round"}</span>
  </button>

  <button class="key strap-key next-key" onclick={onNextHole} disabled={!round.active}>
    <span class="cap">Next<br />Hole</span>
  </button>

  <button class="key strap-key alerts-key" aria-pressed={alertsOn} onclick={onToggleAlerts}>
    <span class="cap">Alerts<br />{alertsOn ? "On" : "Off"}</span>
  </button>
</div>

<style>
  .mount {
    position: relative;
    width: 576px;
    height: 640px;
  }

  .drawing {
    display: block;
  }

  .line {
    stroke: var(--ink);
    stroke-width: 1;
  }

  .wall {
    fill: var(--paper-sunk);
  }

  .proud {
    fill: var(--paper);
  }

  circle.line,
  line.line {
    fill: none;
  }

  /* Stitching: a precise dashed hairline, set in from the seam it follows. */
  .stitch {
    fill: none;
    stroke: var(--ink-soft);
    stroke-width: 1;
    stroke-dasharray: 4 3;
  }

  /* A thick line dashed very short is a row of evenly spaced ticks, which
     is exactly what zipper teeth are. */
  .teeth {
    stroke: var(--ink);
    stroke-width: 8;
    stroke-dasharray: 1 3;
  }

  .channel {
    position: absolute;
    top: 60px;
    left: 24px;
    box-sizing: border-box;
    width: 448px;
    padding: 4px;
    border: 1px solid var(--ink);
    border-top-width: 3px;
    border-left-width: 3px;
    border-radius: 14px;
    background: var(--ink-soft);
  }

  /* A key is two parts: a base fixed to the bag and a cap that sits 4px
     above it. The gap between them is what makes it read as pressable. */
  .key {
    position: absolute;
    padding: 0;
    border: 0;
    border-radius: 6px;
    background: none;
    cursor: pointer;
  }

  .key::before {
    content: "";
    position: absolute;
    inset: 4px 0 0;
    border: 1px solid var(--ink);
    border-radius: 6px;
    background: var(--ink-soft);
  }

  .cap {
    position: absolute;
    inset: 0 0 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid var(--ink);
    border-radius: 6px;
    background: var(--paper-sunk);
    color: var(--ink);
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    line-height: 16px;
    letter-spacing: 0.08em;
    text-align: center;
    text-transform: uppercase;
  }

  /* Wide and low, centred under the screen: the one key used with the bag
     standing in front of you. */
  .round-key {
    top: 544px;
    left: 148px;
    width: 200px;
    height: 52px;
  }

  /* On the strap edge, one above the other, where a thumb falls. Both are
     comfortably over the 44px minimum for a gloved hand. */
  .strap-key {
    left: 500px;
    width: 56px;
    height: 68px;
  }

  .strap-key .cap {
    font-size: 11px;
    letter-spacing: 0.04em;
  }

  .next-key {
    top: 196px;
  }

  .alerts-key {
    top: 288px;
  }

  /* Hover is a tone change only. */
  .key:hover:not(:disabled) .cap {
    background: var(--paper);
  }

  /* The cap travels 1px down, instantly: a physical key has no latency
     between the press and the movement. */
  .key:active:not(:disabled) .cap {
    transform: translateY(1px);
  }

  /* Ink, not red: on the bag, red is reserved for alerts. */
  .key:focus-visible {
    outline: 2px solid var(--ink);
    outline-offset: 3px;
  }

  /* Dashed and soft instead of faded, to match the e-ink screen beside it. */
  .key:disabled {
    cursor: not-allowed;
  }

  .key:disabled .cap {
    border-style: dashed;
    border-color: var(--ink-soft);
    color: var(--ink-soft);
  }
</style>
