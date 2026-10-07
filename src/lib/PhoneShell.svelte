<script>
  // The phone is a screen, not a printed instrument. It is held close, on
  // OLED, with the golfer's full attention, so it is dark, finely set and
  // denser than the bag, and it carries everything that takes more than a
  // glance. It deliberately does not look like the bag display: the contrast
  // between the two is the design argument of the project.
  //
  // The status strip has only a clock and the pairing state. There is no
  // imitation battery or signal icon, because this is our product's screen,
  // not a stock phone template.
  //
  // Motion, all of it the smooth OLED kind:
  //   tabs           slide in the direction you navigated, a spatial cue
  //   notification   slides down and fades in, leaves by fading only,
  //                  because arrival deserves attention and departure does not
  //   exits          are always faster than entrances (140ms against 240ms)
  import { fly, fade } from "svelte/transition";
  import { cubicOut } from "svelte/easing";
  import { MediaQuery } from "svelte/reactivity";
  import PhoneRound from "./PhoneRound.svelte";
  import PhoneSetup from "./PhoneSetup.svelte";

  let {
    paired,
    tab,
    onTab,
    notifications,
    clubs,
    round,
    loadedCount,
    inBagCount,
    clubsOut,
    leftBehind,
    alertsOn,
    onToggleRound,
    onNextHole,
    onToggleAlerts,
    onToggleLoaded
  } = $props();

  const tabs = [
    { id: "round", label: "Round" },
    { id: "setup", label: "Setup" },
    { id: "summary", label: "Summary" }
  ];

  // Svelte transitions run in JavaScript, so the global reduced-motion CSS
  // does not reach them. Each one checks this instead.
  const reducedMotion = new MediaQuery("(prefers-reduced-motion: reduce)");
  let slide = $derived(reducedMotion.current ? 0 : 24);
  let enterMs = $derived(reducedMotion.current ? 0 : 240);
  let exitMs = $derived(reducedMotion.current ? 0 : 140);

  // Which way the content travels. Purely visual, so it is local.
  let direction = $state(1);

  function go(next) {
    if (next === tab) return;
    const index = (id) => tabs.findIndex((t) => t.id === id);
    direction = index(next) > index(tab) ? 1 : -1;
    onTab(next);
  }

  // A real clock, so the strip is not a painted-on fake.
  let now = $state(new Date());
  $effect(() => {
    const timer = setInterval(() => (now = new Date()), 15000);
    return () => clearInterval(timer);
  });
  let clock = $derived(
    now.toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit" })
  );
</script>

<div class="phone">
  <div class="status-strip">
    <time>{clock}</time>
    <span class="pairing" class:paired>
      <span class="pair-mark" aria-hidden="true"></span>
      {paired ? "Bag paired" : "Bag not paired"}
    </span>
  </div>

  <!-- Two kinds of notification, told apart by shape, icon and words as well
       as colour. A club left behind is a filled band with a flag. The bag
       itself left behind is an outlined band with a location pin, because it
       is a different problem at a different scale: walk back much further.
       The text is written in App, where the alert is raised. -->
  {#each notifications as notice (notice.id)}
    <div
      class="notice"
      class:away={notice.kind === "away"}
      role="alert"
      in:fly={{ y: -slide, duration: enterMs, easing: cubicOut }}
      out:fade={{ duration: exitMs }}
    >
      <svg class="notice-icon" width="20" height="20" viewBox="0 0 20 20" aria-hidden="true">
        {#if notice.kind === "away"}
          <path d="M10 18s-5.5-5.6-5.5-9.8a5.5 5.5 0 0 1 11 0C15.5 12.4 10 18 10 18z" />
          <circle cx="10" cy="8" r="2" />
        {:else}
          <path d="M5 2v16M5 3h10l-3 3.5 3 3.5H5" />
        {/if}
      </svg>
      <div>
        <p class="notice-title">{notice.title}</p>
        <p class="notice-text">{notice.text}</p>
      </div>
    </div>
  {/each}

  <!-- Both the leaving and the arriving tab exist for a moment, so they
       share one grid cell instead of stacking. -->
  <div class="body">
    {#key tab}
      <div
        class="tab-body"
        in:fly={{ x: slide * direction, duration: enterMs, easing: cubicOut }}
        out:fly={{ x: -slide * direction, duration: exitMs, easing: cubicOut }}
      >
        {#if tab === "round"}
          <PhoneRound
            {round}
            {loadedCount}
            {inBagCount}
            {clubsOut}
            {leftBehind}
            {alertsOn}
            {onToggleRound}
            {onNextHole}
            {onToggleAlerts}
          />
        {:else if tab === "setup"}
          <PhoneSetup {clubs} {loadedCount} {onToggleLoaded} />
        {:else}
          <p class="pending">Summary: shots per hole and most-used club after the round. Built in Phase 7.</p>
        {/if}
      </div>
    {/key}
  </div>

  <nav class="tabs" aria-label="Phone sections">
    {#each tabs as t (t.id)}
      <button aria-current={tab === t.id ? "page" : undefined} onclick={() => go(t.id)}>
        <span>{t.label}</span>
      </button>
    {/each}
  </nav>
</div>

<style>
  /* 300 by 620 with a 36px corner radius: the phone body is rounded because
     it is a physical object. Everything inside gets at most 4px. */
  .phone {
    display: flex;
    flex-direction: column;
    box-sizing: border-box;
    width: 300px;
    height: 620px;
    overflow: hidden;
    border: 1px solid var(--phone-rule);
    border-radius: 36px;
    background: var(--oled);
    color: var(--phone-text);
    font-family: var(--font-sans);
    /* Tells the browser this surface is dark, so anything it draws itself,
       such as a scrollbar, is dark too. */
    color-scheme: dark;
  }

  .status-strip {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--space-4) var(--space-5) var(--space-2);
    font-family: var(--font-mono);
    font-size: 11px;
    letter-spacing: 0.08em;
    font-variant-numeric: tabular-nums;
  }

  .pairing {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  /* Filled when paired, hollow when not, so the state is not colour alone. */
  .pair-mark {
    box-sizing: border-box;
    width: 8px;
    height: 8px;
    border: 1px solid var(--phone-soft);
    border-radius: 50%;
  }

  .pairing.paired {
    color: var(--fairway);
  }

  .pairing.paired .pair-mark {
    border-color: var(--fairway);
    background: var(--fairway);
  }

  /* A filled red band with light text. Red text on black would only be
     3.3:1, so the red is the ground and the text sits on it at 5.2:1. */
  .notice {
    display: flex;
    gap: var(--space-2);
    margin: 0 var(--space-3) var(--space-1);
    padding: var(--space-1) var(--space-3);
    border: 2px solid var(--flag);
    border-radius: 4px;
    background: var(--flag);
  }

  .notice.away {
    background: var(--oled-raised);
  }

  .notice-icon {
    flex-shrink: 0;
    fill: none;
    stroke: var(--phone-text);
    stroke-width: 1.5;
  }

  .notice p {
    margin: 0;
  }

  .notice-title {
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 600;
    line-height: 16px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }

  .notice-text {
    font-size: 13px;
    line-height: 18px;
  }

  .body {
    display: grid;
    flex: 1;
    min-height: 0;
    overflow: hidden;
  }

  .tab-body {
    grid-area: 1 / 1;
    min-height: 0;
    padding: var(--space-3) var(--space-4);
    overflow-y: auto;
    scrollbar-width: thin;
    scrollbar-color: var(--phone-rule) var(--oled);
  }

  .pending {
    margin: 0;
    font-size: 14px;
    line-height: 20px;
    color: var(--phone-soft);
  }

  .tabs {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    border-top: 1px solid var(--phone-rule);
  }

  /* 56px tall and 100px wide: a thumb target, well over the 44px minimum. */
  .tabs button {
    height: 56px;
    padding: 0 0 var(--space-2);
    border: none;
    border-radius: 0;
    background: var(--oled);
    color: var(--phone-soft);
    font-family: var(--font-mono);
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    cursor: pointer;
    transition: background-color var(--t-quick) var(--e-enter);
  }

  /* The rule sits under the label itself, clear of the phone's rounded
     corners, which would clip a rule along the bottom edge. */
  .tabs span {
    padding-bottom: var(--space-1);
    border-bottom: 2px solid transparent;
  }

  .tabs button:hover {
    background: var(--oled-raised);
  }

  .tabs button:active {
    transform: translateY(1px);
    transition: none;
  }

  /* The active tab is brighter and bolder as well as ruled in red, so it
     does not depend on the colour. */
  .tabs button[aria-current="page"] {
    color: var(--phone-text);
    font-weight: 600;
  }

  .tabs button[aria-current="page"] span {
    border-bottom-color: var(--flag);
  }

  /* Inset, because the phone body clips anything outside it. */
  .tabs button:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: -4px;
  }
</style>
