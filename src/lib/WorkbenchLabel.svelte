<script>
  // The small annotation that names each region of the page, like the labels
  // on a technical drawing: a mono uppercase name, an optional note saying
  // where the part lives, and a hairline leader rule running to the edge.
  // This is what makes the page read as a design document rather than a web
  // app. It is an <h2> so screen-reader users can jump between regions.
  let { label, note = "" } = $props();
</script>

<h2 class="reveal">
  <span class="name">{label}{note ? "," : ""}</span>
  {#if note}<span class="note">{note}</span>{/if}
  <span class="leader" aria-hidden="true"></span>
</h2>

<style>
  h2 {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    /* 16px line plus 8px margin is one 24px grid square, so whatever sits
       under a label starts on a grid line. */
    margin: 0 0 var(--space-2);
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    line-height: 16px;
    letter-spacing: 0.08em;
    white-space: nowrap;
    color: var(--table-ink);
  }

  .name {
    text-transform: uppercase;
  }

  .note {
    font-weight: 400;
    color: var(--table-soft);
  }

  .leader {
    flex: 1;
    height: 1px;
    background: var(--table-soft);
  }
</style>
