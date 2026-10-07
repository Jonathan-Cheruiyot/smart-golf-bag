<script>
  // Level 0 requirement: a drawing of where this interface sits on the real
  // object. It is drawn like a technical figure on the drafting table:
  // hairline ink strokes, no fills except the screen, numbered callouts in
  // red with leader lines.
  //
  // The six callouts are the argument that the interface is not one flat
  // surface: screen in the side pocket, a key below it, two more on the
  // strap, sensors in the top cuff, a stand sensor in the base, and the phone
  // beside the bag, joined to it by a dashed pairing line. That line is the
  // two-device idea in one stroke.
  //
  // `compact` is the small figure in the header strip: the bag alone, with
  // the fine detail left out so it stays legible at 27 by 48. The button
  // around it carries the label, so there it is hidden from screen readers.
  let { compact = false } = $props();

  // Unique per instance, so two drawings on one page never share a mask.
  const uid = $props.id();

  // cx, cy: centre of the numbered balloon. The leader runs from x1, y1 at
  // the balloon's edge to x2, y2 on the part it names.
  const zones = [
    {
      n: 1,
      title: "Side pocket display",
      text: "The e-ink screen, set into the pocket panel. Indicators only: hole, round state, club count, message, club rack.",
      cx: 40, cy: 230, x1: 55, y1: 240, x2: 128, y2: 282
    },
    {
      n: 2,
      title: "Round key",
      text: "Start or End Round. A physical key below the screen, because e-ink touch is slow, golfers wear gloves, and the bag is used in rain.",
      cx: 40, cy: 440, x1: 56, y1: 432, x2: 152, y2: 394
    },
    {
      n: 3,
      title: "Strap edge keys",
      text: "Next Hole and Alerts. Raised so they work by feel with the bag on your shoulder.",
      cx: 392, cy: 250, x1: 378, y1: 260, x2: 320, y2: 304
    },
    {
      n: 4,
      title: "Top cuff sensors and slot lights",
      text: "Each divider senses its club. A light under an empty slot turns red.",
      cx: 312, cy: 86, x1: 298, y1: 94, x2: 262, y2: 116
    },
    {
      n: 5,
      title: "Base stand sensor",
      text: "Detects when the bag is set down, so the screen wakes when it can be seen.",
      cx: 312, cy: 612, x1: 298, y1: 604, x2: 252, y2: 576
    },
    {
      n: 6,
      title: "Phone, paired",
      text: "Travels with the golfer. Carries setup, history, and the alert for when the bag itself is left behind.",
      cx: 478, cy: 224, x1: 478, y1: 242, x2: 478, y2: 270
    }
  ];
</script>

<div class="wrap">
  <svg
    width={compact ? 27 : 350}
    height={compact ? 48 : 400}
    viewBox={compact ? "0 0 360 640" : "0 0 560 640"}
    role={compact ? undefined : "img"}
    aria-hidden={compact}
    aria-label={compact
      ? undefined
      : "Golf bag with five numbered interface zones, and the paired phone beside it"}
  >
    <g class="ink">
      <!-- legs, starting at the body's edge so no line shows through it -->
      <line x1="110" y1="300" x2="56" y2="600" />
      <line x1="110" y1="336" x2="84" y2="612" />

      <!-- top cuff, body, base -->
      <rect x="100" y="108" width="160" height="40" rx="12" />
      <rect x="110" y="148" width="140" height="404" rx="24" />
      <rect x="110" y="552" width="140" height="28" rx="12" />

      <!-- strap -->
      <path d="M250 176 C 334 236, 334 420, 250 478" />

      <!-- side pocket -->
      <rect x="128" y="240" width="104" height="170" rx="12" />

      {#if !compact}
        <!-- club shafts and heads -->
        <line x1="140" y1="34" x2="140" y2="108" />
        <line x1="164" y1="18" x2="164" y2="108" />
        <line x1="190" y1="40" x2="190" y2="108" />
        <line x1="214" y1="26" x2="214" y2="108" />
        <rect x="130" y="22" width="20" height="12" rx="3" />
        <rect x="155" y="6" width="18" height="12" rx="3" />
        <rect x="205" y="14" width="18" height="12" rx="3" />

        <!-- one sensor and light per slot in the cuff -->
        {#each [122, 146, 170, 194, 218, 242] as x}
          <circle cx={x} cy="128" r="5" />
        {/each}

        <!-- the two keys on the strap edge, and the round key below the screen -->
        <circle cx="312" cy="310" r="8" />
        <circle cx="312" cy="346" r="8" />
        <rect x="154" y="384" width="52" height="14" rx="3" />

        <!-- the phone, drawn as an outline beside the bag -->
        <rect x="440" y="270" width="76" height="150" rx="10" />
        <line x1="466" y1="282" x2="490" y2="282" />
      {/if}
    </g>

    <!-- the screen: the one filled shape, so the eye finds it first -->
    <rect class="screen" x="140" y="254" width="80" height="120" />

    {#if !compact}
      <!-- The pairing line draws itself when the figure appears, then drifts
           slowly to show a live connection. The mask is what draws it on:
           a solid line that grows over the dashed one. -->
      <mask id="{uid}-draw" maskUnits="userSpaceOnUse" x="232" y="380" width="208" height="40">
        <line class="pair-draw" x1="232" y1="400" x2="440" y2="400" stroke-width="40" pathLength="1" />
      </mask>
      <line class="pair-line" x1="232" y1="400" x2="440" y2="400" mask="url(#{uid}-draw)" />

      <g class="callouts">
        {#each zones as zone}
          <line x1={zone.x1} y1={zone.y1} x2={zone.x2} y2={zone.y2} />
          <circle cx={zone.cx} cy={zone.cy} r="18" />
          <text x={zone.cx} y={zone.cy}>{zone.n}</text>
        {/each}
      </g>
    {/if}
  </svg>

  {#if !compact}
    <ol class="zones">
      {#each zones as zone}
        <li>
          <span class="num" aria-hidden="true">{zone.n}</span>
          <span>
            <strong>{zone.title}</strong><br />
            {zone.text}
          </span>
        </li>
      {/each}
    </ol>
  {/if}
</div>

<style>
  .wrap {
    display: flex;
    gap: var(--space-5);
    align-items: flex-start;
  }

  svg {
    flex-shrink: 0;
  }

  /* non-scaling-stroke keeps every line a true 1px hairline at both sizes. */
  .ink,
  .callouts {
    fill: none;
    stroke-width: 1;
    vector-effect: non-scaling-stroke;
  }

  .ink :global(*),
  .callouts line,
  .callouts circle {
    vector-effect: non-scaling-stroke;
  }

  .ink {
    stroke: var(--table-ink);
  }

  .screen {
    fill: var(--table-ink);
  }

  .callouts {
    stroke: var(--flag);
  }

  .callouts text {
    fill: var(--flag);
    stroke: none;
    font-family: var(--font-mono);
    font-size: 22px;
    font-weight: 500;
    text-anchor: middle;
    dominant-baseline: central;
  }

  .zones {
    display: flex;
    flex-direction: column;
    gap: var(--space-3);
    width: 320px;
    margin: 0;
    padding: 0;
    list-style: none;
    font-size: 13px;
    line-height: 18px;
    color: var(--table-soft);
  }

  .zones li {
    display: flex;
    gap: var(--space-2);
    align-items: flex-start;
  }

  /* The same balloon as on the drawing, so the list reads as its key. */
  .num {
    display: flex;
    flex-shrink: 0;
    align-items: center;
    justify-content: center;
    box-sizing: border-box;
    width: 20px;
    height: 20px;
    border: 1px solid var(--flag);
    border-radius: 50%;
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    color: var(--flag);
  }

  strong {
    font-weight: 600;
    color: var(--table-ink);
  }
</style>
