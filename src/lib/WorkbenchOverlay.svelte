<script>
  // A sheet laid over the drafting table, used for anything that takes more
  // room than the header strip can give: the placement drawing and the info
  // text. It is a native <dialog> opened with showModal() because the browser
  // then handles the hard accessibility parts for free: focus moves in and is
  // trapped, Escape closes it, and focus returns to the button that opened it.
  //
  // It opens and closes instantly. MOTION.md allows the page harness one
  // reveal on load and nothing after.
  let { title, open, onClose, children } = $props();

  const uid = $props.id();
  let dialog = $state();

  // `open` is owned by App. This keeps the dialog element in step with it.
  $effect(() => {
    if (!dialog) return;
    if (open && !dialog.open) dialog.showModal();
    if (!open && dialog.open) dialog.close();
  });
</script>

<!-- onclose fires for Escape and backdrop clicks too, so App's state never
     disagrees with what is on screen. -->
<dialog
  bind:this={dialog}
  onclose={onClose}
  closedby="any"
  aria-labelledby="{uid}-title"
>
  <div class="head">
    <h2 id="{uid}-title">{title}</h2>
    <button class="harness-btn" onclick={onClose}>Close</button>
  </div>
  {@render children()}
</dialog>

<style>
  /* A hairline and a tone change separate the sheet from the table. No
     shadow, no rounded corners. */
  dialog {
    padding: var(--space-5);
    border: 1px solid var(--table-ink);
    background: var(--table);
    color: var(--table-ink);
  }

  dialog::backdrop {
    background: color-mix(in srgb, var(--table-ink) 45%, transparent);
  }

  .head {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--space-6);
    margin-bottom: var(--space-5);
    padding-bottom: var(--space-3);
    border-bottom: 1px solid var(--table-soft);
  }

  h2 {
    margin: 0;
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }
</style>
