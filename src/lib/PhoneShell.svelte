<script>
  // The phone is a screen, not a printed instrument. It is held close, on
  // OLED, with the golfer's full attention, so it is dark, finely set and
  // denser than the bag, and it carries everything that takes more than a
  // glance. It deliberately does not look like the bag display: the contrast
  // between the two is the design argument of the project.
  //
  // It is drawn as a solid manufactured object, at the same fidelity as the
  // bag mount beside it, so the two read as real devices on one table:
  //   - flat tone steps, no gradients and no gloss: a lighter edge band so
  //     the frame reads as metal, a chamfer hairline, the dark channel the
  //     glass sits in, a thin uniform bezel, then the screen
  //   - concentric corners: each layer's radius is the screen's radius plus
  //     the thickness outside it, which is what makes a rounded object look
  //     machined and not just "rounded"
  //   - side keys that break the silhouette, a hatched earpiece slot, a
  //     centred round camera, and a gesture bar
  // It is a generic modern phone, deliberately not any one maker's: no
  // branded cutout, typeface or control styling. The app inside it is ours.
  //
  // The status bar is the kind every phone has (clock, signal, wifi,
  // battery) but drawn in our own palette as stroke icons, plus the one
  // thing that is ours: whether the bag is paired.
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
  import PhoneSummary from "./PhoneSummary.svelte";

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

  // A real clock. The signal, wifi and battery icons are set dressing and
  // are hidden from screen readers; the clock and pairing state are real.
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
  <!-- Hardware keys: volume up and down on the left, a longer side key on
       the right. Drawn only; they are not part of the interface. -->
  <span class="side-key left vol-up" aria-hidden="true"></span>
  <span class="side-key left vol-down" aria-hidden="true"></span>
  <span class="side-key right power" aria-hidden="true"></span>

  <div class="chamfer">
  <div class="glass">
  <!-- Earpiece slot in the top bezel: a row of fine ticks, the same hatch
       idea as the zipper on the bag. -->
  <svg class="earpiece" width="40" height="2" viewBox="0 0 40 2" aria-hidden="true">
    <line x1="0" y1="1" x2="40" y2="1" />
  </svg>

  <div class="screen">
  <div class="status-bar">
    <div class="status-left">
      <time>{clock}</time>
      <span class="pairing" class:paired>
        <span class="pair-mark" aria-hidden="true"></span>
        <span aria-hidden="true">Bag</span>
        <span class="sr">{paired ? "Bag paired" : "Bag not paired"}</span>
      </span>
    </div>
    <span class="camera" aria-hidden="true"></span>
    <div class="status-right">
      <svg class="status-icon" width="16" height="12" viewBox="0 0 16 12" aria-hidden="true">
        <path d="M2 11V8.5M6 11V6M10 11V3.5M14 11V1" />
      </svg>
      <svg class="status-icon" width="16" height="12" viewBox="0 0 16 12" aria-hidden="true">
        <path d="M1.5 4.6a9.6 9.6 0 0 1 13 0M4.2 7.3a5.7 5.7 0 0 1 7.6 0" />
        <circle class="solid" cx="8" cy="10" r="1" />
      </svg>
      <svg class="status-icon" width="24" height="12" viewBox="0 0 24 12" aria-hidden="true">
        <rect x="0.75" y="0.75" width="20" height="10.5" rx="3" />
        <path d="M22.75 4.25v3.5" />
        <rect class="solid" x="3" y="3" width="12" height="6" rx="1" />
      </svg>
    </div>
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
          <PhoneSummary {round} {clubs} />
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

  <div class="gesture" aria-hidden="true"><span></span></div>
  </div>
  </div>
  </div>
</div>

<style>
  /* 296 by 640 is close to the 19.5:9 of a current phone. The housing is
     four layers, outside in, and each radius is the screen's 30px plus the
     thickness outside it, so the corners stay concentric:
       .phone    1px outline + 3px metal edge band    radius 40
       .chamfer  1px light chamfer + 1px dark channel radius 36
       .glass    4px uniform bezel                    radius 34
       .screen   the display itself                   radius 30 */
  .phone {
    --r-screen: 30px;
    position: relative;
    box-sizing: border-box;
    width: 296px;
    height: 640px;
    padding: 3px;
    border: 1px solid var(--ink);
    border-radius: calc(var(--r-screen) + 10px);
    background: var(--phone-rule);
  }

  /* The light hairline is the polished chamfer where the metal band turns
     in to meet the glass; the dark line inside it is the channel the glass
     sits in. */
  .chamfer {
    box-sizing: border-box;
    height: 100%;
    padding: 1px;
    border: 1px solid var(--phone-soft);
    border-radius: calc(var(--r-screen) + 6px);
    background: var(--oled);
  }

  .glass {
    position: relative;
    box-sizing: border-box;
    height: 100%;
    padding: 4px;
    border-radius: calc(var(--r-screen) + 4px);
    background: var(--oled-raised);
  }

  .screen {
    display: flex;
    flex-direction: column;
    height: 100%;
    overflow: hidden;
    border-radius: var(--r-screen);
    background: var(--oled);
    color: var(--phone-text);
    font-family: var(--font-sans);
    /* Tells the browser this surface is dark, so anything it draws itself,
       such as a scrollbar, is dark too. */
    color-scheme: dark;
  }

  /* Side keys stand 3px proud of the frame. Each is a cap in the frame's
     metal tone with a light leading edge, so it reads as a separate part
     with thickness and not a bump in the outline. */
  .side-key {
    position: absolute;
    box-sizing: border-box;
    width: 4px;
    border: 1px solid var(--ink);
    background: var(--phone-rule);
  }

  .side-key::after {
    content: "";
    position: absolute;
    top: 2px;
    bottom: 2px;
    width: 1px;
    background: var(--phone-soft);
  }

  .side-key.left {
    left: -4px;
    border-right: 0;
    border-radius: 2px 0 0 2px;
  }

  .side-key.left::after {
    left: 0;
  }

  .side-key.right {
    right: -4px;
    border-left: 0;
    border-radius: 0 2px 2px 0;
  }

  .side-key.right::after {
    right: 0;
  }

  .vol-up {
    top: 100px;
    height: 36px;
  }

  .vol-down {
    top: 144px;
    height: 36px;
  }

  .power {
    top: 132px;
    height: 60px;
  }

  .earpiece {
    position: absolute;
    top: 1px;
    left: 50%;
    margin-left: -20px;
    stroke: var(--phone-soft);
    stroke-width: 2;
    stroke-dasharray: 1 2;
  }

  /* Clock and bag pairing on the left, camera centred, the generic phone
     status on the right. The side padding clears the screen's rounded
     corners. */
  .status-bar {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    align-items: center;
    height: 36px;
    padding: 0 var(--space-5) 0 var(--space-5);
    font-family: var(--font-mono);
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.04em;
    font-variant-numeric: tabular-nums;
  }

  /* A round camera, centred. A lens is the one true circle on the phone. */
  .camera {
    position: relative;
    box-sizing: border-box;
    width: 12px;
    height: 12px;
    border: 1px solid var(--phone-rule);
    border-radius: 50%;
    background: var(--oled-raised);
  }

  .camera::after {
    content: "";
    position: absolute;
    top: 3px;
    left: 3px;
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: var(--phone-rule);
  }

  .status-left {
    display: flex;
    align-items: center;
    gap: var(--space-2);
  }

  .status-right {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 6px;
  }

  .status-icon {
    flex-shrink: 0;
    fill: none;
    stroke: var(--phone-text);
    stroke-width: 1.5;
    stroke-linecap: round;
    stroke-linejoin: round;
  }

  .status-icon .solid {
    fill: var(--phone-text);
    stroke: none;
  }

  .pairing {
    display: flex;
    align-items: center;
    gap: var(--space-1);
    font-size: 10px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--phone-soft);
  }

  /* The full wording, for screen readers only. */
  .sr {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
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
    padding: var(--space-3);
    overflow-y: auto;
    scrollbar-width: thin;
    scrollbar-color: var(--phone-rule) var(--oled);
  }

  /* A tab that scrolls can take keyboard focus so it can be scrolled with
     the arrow keys; it gets the project's ring, not the browser's. */
  .tab-body:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: -2px;
  }

  .tabs {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    border-top: 1px solid var(--phone-rule);
  }

  /* 52px tall and 92px wide: a thumb target, well over the 44px minimum. */
  .tabs button {
    height: 52px;
    padding: 0;
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

  /* Inset, because the screen clips anything outside it. */
  .tabs button:focus-visible {
    outline: 2px solid var(--flag);
    outline-offset: -4px;
  }

  /* The thin light bar every gesture-driven phone shows at the bottom. */
  .gesture {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 16px;
  }

  .gesture span {
    width: 96px;
    height: 4px;
    border-radius: 2px;
    background: var(--phone-text);
  }
</style>
