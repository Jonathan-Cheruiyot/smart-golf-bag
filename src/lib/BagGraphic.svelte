<script>
  // Level 0 requirement: a graphic showing where this UI would sit on the
  // real object. The four numbered zones also prove the interface is not
  // confined to a single flat surface.
  const zones = [
    { n: 1, title: "Side pocket display", text: "The main screen, shown to the left. Angled up so it reads while the bag stands on its legs." },
    { n: 2, title: "Strap buttons", text: "Next Hole and Alerts. Raised so they work by feel with the bag on your shoulder." },
    { n: 3, title: "Top cuff sensors", text: "Each divider senses its club. A light under an empty slot turns red." },
    { n: 4, title: "Base stand sensor", text: "Detects when the bag is set down, so the screen wakes when it can be seen." }
  ];

  // `compact` draws only the bag, small, for the figure in the header strip.
  // The button around it carries the label, so the drawing is hidden from
  // screen readers there to avoid reading it twice.
  let { compact = false } = $props();
</script>

<div class="wrap">
  <svg
    width={compact ? 27 : 180}
    height={compact ? 48 : 320}
    viewBox="0 0 360 640"
    aria-hidden={compact}
    aria-label="Golf bag with four numbered interface zones"
  >
    <!-- club shafts poking out of the top -->
    <line x1="140" y1="30" x2="140" y2="112" stroke="#1d2721" stroke-width="3" />
    <line x1="164" y1="14" x2="164" y2="112" stroke="#1d2721" stroke-width="3" />
    <line x1="190" y1="40" x2="190" y2="112" stroke="#1d2721" stroke-width="3" />
    <line x1="214" y1="22" x2="214" y2="112" stroke="#1d2721" stroke-width="3" />
    <rect x="130" y="22" width="20" height="12" rx="3" fill="#1d2721" />
    <rect x="156" y="6" width="18" height="12" rx="3" fill="#1d2721" />
    <rect x="206" y="14" width="18" height="12" rx="3" fill="#1d2721" />

    <!-- top cuff with one red slot light -->
    <rect x="100" y="108" width="160" height="40" rx="12" fill="#dce8df" stroke="#1d2721" stroke-width="2" />
    <circle cx="122" cy="128" r="5" fill="#2f6b45" />
    <circle cx="146" cy="128" r="5" fill="#2f6b45" />
    <circle cx="170" cy="128" r="5" fill="#2f6b45" />
    <circle cx="194" cy="128" r="5" fill="#a8432d" />
    <circle cx="218" cy="128" r="5" fill="#2f6b45" />
    <circle cx="242" cy="128" r="5" fill="#2f6b45" />

    <!-- legs -->
    <line x1="120" y1="300" x2="56" y2="600" stroke="#1d2721" stroke-width="4" />
    <line x1="130" y1="300" x2="90" y2="612" stroke="#1d2721" stroke-width="4" />

    <!-- bag body -->
    <rect x="110" y="148" width="140" height="420" rx="30" fill="#fbfaf6" stroke="#1d2721" stroke-width="2" />

    <!-- side pocket screen -->
    <rect x="128" y="240" width="104" height="170" rx="16" fill="#ffffff" stroke="#1d2721" stroke-width="2" />
    <rect x="140" y="256" width="80" height="120" rx="8" fill="#14301f" />
    <text x="180" y="300" text-anchor="middle" font-size="26" font-weight="700" fill="#ffd37a">13</text>
    <text x="180" y="320" text-anchor="middle" font-size="11" fill="#a9c7b3">of 14 clubs</text>
    <rect x="150" y="334" width="60" height="16" rx="3" fill="#6b2620" />

    <!-- strap with two buttons -->
    <path d="M250 176 C 334 236, 334 420, 250 478" fill="none" stroke="#1d2721" stroke-width="12" stroke-linecap="round" />
    <circle cx="318" cy="296" r="9" fill="#2f6b45" stroke="#1d2721" stroke-width="2" />
    <circle cx="321" cy="336" r="9" fill="#2f6b45" stroke="#1d2721" stroke-width="2" />

    <!-- base -->
    <rect x="110" y="552" width="140" height="28" rx="12" fill="#dce8df" stroke="#1d2721" stroke-width="2" />

    <!-- numbered markers -->
    <circle cx="84" cy="318" r="16" fill="#a8432d" />
    <text x="84" y="325" text-anchor="middle" font-size="17" font-weight="700" fill="#ffffff">1</text>
    <circle cx="340" cy="258" r="16" fill="#a8432d" />
    <text x="340" y="265" text-anchor="middle" font-size="17" font-weight="700" fill="#ffffff">2</text>
    <circle cx="292" cy="128" r="16" fill="#a8432d" />
    <text x="292" y="135" text-anchor="middle" font-size="17" font-weight="700" fill="#ffffff">3</text>
    <circle cx="292" cy="566" r="16" fill="#a8432d" />
    <text x="292" y="573" text-anchor="middle" font-size="17" font-weight="700" fill="#ffffff">4</text>
  </svg>

  {#if !compact}
    <ul class="zones">
      {#each zones as zone}
        <li>
          <span class="num">{zone.n}</span>
          <span>
            <strong>{zone.title}</strong><br />
            {zone.text}
          </span>
        </li>
      {/each}
    </ul>
  {/if}
</div>

<style>
  .wrap {
    display: flex;
    gap: 18px;
    align-items: flex-start;
  }

  .zones {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    flex-direction: column;
    gap: 12px;
    font-size: 13px;
    line-height: 1.4;
    color: #3f4a43;
  }

  .zones li {
    display: flex;
    gap: 8px;
    align-items: flex-start;
  }

  .num {
    flex-shrink: 0;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #a8432d;
    color: #ffffff;
    font-size: 12px;
    font-weight: 700;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  strong {
    color: #1d2721;
  }
</style>
