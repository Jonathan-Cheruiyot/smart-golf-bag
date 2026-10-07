<script>
  // The Setup tab is where the phone acts as a remote control. Choosing
  // today's clubs takes reading, comparing and deciding, which is far more
  // than a two-second glance, so it lives here and never on the bag. The bag
  // simply shows the result: its rack redraws the moment a club is ticked.
  //
  // The rules of golf allow 14 clubs. The limit is enforced here with a
  // stated reason, never a silent failure, because a tick box that refuses
  // to tick without explanation looks broken.
  let { clubs, loadedCount, onToggleLoaded } = $props();

  const LIMIT = 14;

  // Why the last tap was refused. Feedback only, so it is local.
  let reason = $state("");

  function toggle(event, club) {
    if (!club.loaded && loadedCount >= LIMIT) {
      // preventDefault stops the box from ticking, so the screen never shows
      // a state the bag does not have.
      event.preventDefault();
      reason = `The bag is full: ${LIMIT} clubs is the limit. Take one out before adding the ${club.name}.`;
      return;
    }
    if (club.loaded && !club.inBag) {
      event.preventDefault();
      reason = `The ${club.name} is out of the bag. Put it back before removing it from today's set.`;
      return;
    }
    reason = "";
    onToggleLoaded(club.id);
  }
</script>

<div class="setup">
  <div class="head">
    <h3>Today's bag</h3>
    <p class="tally" class:full={loadedCount === LIMIT}>
      {loadedCount} / {LIMIT} selected
    </p>
  </div>

  {#if reason}
    <p class="reason" role="alert">{reason}</p>
  {/if}

  <ul>
    {#each clubs as club (club.id)}
      <li>
        <label>
          <input
            type="checkbox"
            checked={club.loaded}
            onclick={(event) => toggle(event, club)}
          />
          <span class="name">{club.name}</span>
        </label>
      </li>
    {/each}
  </ul>
</div>

<style>
  .head {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    padding-bottom: var(--space-2);
    border-bottom: 1px solid var(--phone-rule);
  }

  h3,
  .tally {
    margin: 0;
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  .tally {
    font-variant-numeric: tabular-nums;
  }

  /* A full bag reads brighter and bolder, not just a different colour. */
  .tally.full {
    font-weight: 600;
    color: var(--phone-text);
  }

  .reason {
    margin: var(--space-2) 0 0;
    padding: var(--space-2) var(--space-3);
    border-left: 2px solid var(--flag);
    background: var(--oled-raised);
    font-size: 13px;
    line-height: 18px;
  }

  /* Two columns so all 18 clubs fit without scrolling. */
  ul {
    display: grid;
    grid-template-columns: 1fr 1fr;
    column-gap: var(--space-2);
    margin: var(--space-2) 0 0;
    padding: 0;
    list-style: none;
  }

  /* The whole row is the label, so the touch target is the full 44px row
     and not just the small box. */
  label {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    min-height: 44px;
    font-size: 13px;
    line-height: 16px;
    cursor: pointer;
  }

  input {
    position: relative;
    flex-shrink: 0;
    box-sizing: border-box;
    width: 18px;
    height: 18px;
    margin: 0;
    appearance: none;
    border: 1px solid var(--phone-soft);
    border-radius: 2px;
    background: var(--oled);
    cursor: pointer;
    transition: background-color var(--t-quick) var(--e-enter);
  }

  input:checked {
    border-color: var(--phone-text);
    background: var(--phone-text);
  }

  /* The tick is drawn from two borders, so it takes its colour from a token. */
  input:checked::after {
    content: "";
    position: absolute;
    top: 2px;
    left: 5px;
    width: 4px;
    height: 8px;
    border: solid var(--oled);
    border-width: 0 2px 2px 0;
    transform: rotate(45deg);
  }

  input:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: 2px;
  }

  /* Clubs left at home recede, so the eye lands on what is in the bag. */
  input:not(:checked) + .name {
    color: var(--phone-soft);
  }
</style>
