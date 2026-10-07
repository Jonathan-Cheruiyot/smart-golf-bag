<script>
  // The Summary tab is the round's history, and history is phone-only: it
  // takes reading and comparing, which is more than a two-second glance, so
  // none of it ever appears on the bag.
  //
  // Everything here is counted from round.log, which gains one entry each
  // time a club is returned to the bag. Nothing is estimated or invented: if
  // no club has been returned yet, the tab says so.
  //
  // The chart is hand-written inline SVG, no chart library. It is a column
  // chart because the question is "how many, on each hole", a count per
  // category. One series, so one neutral colour and no legend; the heading
  // says what the columns are. Red is not used: on this product red means
  // an alert, and a shot count is not one.
  let { round, clubs } = $props();

  let total = $derived(round.log.length);

  // Always at least the front nine, so a short round still reads as a round
  // and not as two stray columns.
  let holeCount = $derived(
    Math.min(18, Math.max(9, round.hole, ...round.log.map((entry) => entry.hole)))
  );

  let perHole = $derived(
    Array.from({ length: holeCount }, (_, i) => ({
      hole: i + 1,
      shots: round.log.filter((entry) => entry.hole === i + 1).length
    }))
  );

  // One row per club that was used, most used first.
  let clubsUsed = $derived(
    clubs
      .map((club) => ({
        id: club.id,
        name: club.name,
        shots: round.log.filter((entry) => entry.clubId === club.id).length,
        holes: [...club.usedOnHoles].sort((a, b) => a - b)
      }))
      .filter((club) => club.shots > 0)
      .sort((a, b) => b.shots - a.shots)
  );

  let mostUsed = $derived.by(() => {
    if (clubsUsed.length === 0) return "";
    const top = clubsUsed.filter((club) => club.shots === clubsUsed[0].shots);
    if (top.length === 1) return top[0].name;
    return `${top[0].name}, tied with ${top.length - 1} other${top.length > 2 ? "s" : ""}`;
  });

  let state = $derived(
    round.active ? `In progress, hole ${round.hole}` : total > 0 ? "Round over" : "No round yet"
  );

  // Chart geometry, in SVG units that are also pixels.
  const W = 252;
  const BASE = 92; // y of the baseline
  const TOP = 14; // room above the tallest column for its label
  let band = $derived(W / holeCount);
  let barWidth = $derived(Math.min(16, band - 4));
  // The scale starts at zero and never stretches a quiet round: one shot is
  // drawn at a quarter height until some hole has more than four.
  let unit = $derived((BASE - TOP) / Math.max(4, ...perHole.map((h) => h.shots)));

  const plural = (n) => `${n} ${n === 1 ? "shot" : "shots"}`;

  let chartLabel = $derived(
    "Shots per hole. " + perHole.map((h) => `Hole ${h.hole}: ${plural(h.shots)}`).join(". ") + "."
  );
</script>

<div class="summary">
  <div class="meta">
    <span>Round summary</span>
    <span>{state}</span>
  </div>

  {#if total === 0}
    <p class="empty">
      No shots recorded yet. A shot is logged each time a club goes back in
      the bag during a round.
    </p>
  {:else}
    <dl class="stats">
      <div>
        <dt>Total shots</dt>
        <dd class="big">{total}</dd>
      </div>
      <div>
        <dt>Most used club</dt>
        <dd>{mostUsed}</dd>
      </div>
    </dl>

    <figure>
      <figcaption>Shots per hole</figcaption>
      <svg width={W} height="112" viewBox="0 0 {W} 112" role="img" aria-label={chartLabel}>
        <line class="baseline" x1="0" y1={BASE + 0.5} x2={W} y2={BASE + 0.5} />
        {#each perHole as h (h.hole)}
          {@const x = (h.hole - 1) * band + band / 2}
          {#if h.shots > 0}
            <rect
              class="column"
              x={x - barWidth / 2}
              y={BASE - h.shots * unit}
              width={barWidth}
              height={h.shots * unit}
            >
              <title>Hole {h.hole}: {plural(h.shots)}</title>
            </rect>
            <text class="value" {x} y={BASE - h.shots * unit - 4}>{h.shots}</text>
          {/if}
          <!-- With 18 holes there is only room to number every other one. -->
          {#if holeCount <= 9 || h.hole % 2 === 1}
            <text class="tick" {x} y={BASE + 14}>{h.hole}</text>
          {/if}
        {/each}
      </svg>
    </figure>

    <section aria-labelledby="used-heading">
      <h3 id="used-heading">Clubs used</h3>
      <ul>
        {#each clubsUsed as club (club.id)}
          <li>
            <span class="name">{club.name}</span>
            <span class="detail">
              {plural(club.shots)}, hole{club.holes.length === 1 ? "" : "s"} {club.holes.join(", ")}
            </span>
          </li>
        {/each}
      </ul>
    </section>
  {/if}
</div>

<style>
  .summary {
    display: flex;
    flex-direction: column;
    gap: var(--space-3);
  }

  .meta,
  dt,
  figcaption,
  h3 {
    margin: 0;
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  .meta {
    display: flex;
    justify-content: space-between;
    font-weight: 400;
  }

  .empty {
    margin: 0;
    font-size: 14px;
    line-height: 20px;
    color: var(--phone-soft);
  }

  .stats {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: var(--space-5);
    margin: 0;
  }

  dd {
    margin: var(--space-1) 0 0;
    font-size: 14px;
    line-height: 20px;
  }

  dd.big {
    font-family: var(--font-mono);
    font-size: 32px;
    font-weight: 600;
    font-variant-numeric: tabular-nums;
    line-height: 36px;
  }

  figure,
  section {
    margin: 0;
    padding-top: var(--space-2);
    border-top: 1px solid var(--phone-rule);
  }

  svg {
    display: block;
    margin-top: var(--space-2);
  }

  /* The axis is quiet so the columns are what you read. */
  .baseline {
    stroke: var(--phone-rule);
    stroke-width: 1;
  }

  .column {
    fill: var(--phone-text);
  }

  .value,
  .tick {
    font-family: var(--font-mono);
    font-size: 10px;
    font-variant-numeric: tabular-nums;
    text-anchor: middle;
  }

  .value {
    fill: var(--phone-text);
  }

  .tick {
    fill: var(--phone-soft);
  }

  ul {
    margin: var(--space-1) 0 0;
    padding: 0;
    list-style: none;
  }

  li {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    gap: var(--space-2);
    padding: var(--space-1) 0;
    font-size: 14px;
    line-height: 20px;
  }

  .detail {
    font-family: var(--font-mono);
    font-size: 11px;
    text-align: right;
    color: var(--phone-soft);
  }
</style>
