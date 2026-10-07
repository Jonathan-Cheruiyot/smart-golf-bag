# MOTION.md

Motion and interaction specification for the Smart Golf Bag.
Read alongside `CLAUDE.md`. This governs every animation in the project.

---

## 1. The governing principle

**Motion expresses material.** Each surface moves the way its physical
display actually moves. This is the same argument as the visual design: the
two devices look different because they are different, and they should move
differently for the same reason.

Do **not** apply one global transition style to everything. A single
house easing across both devices is exactly what makes a project look
templated, and it quietly destroys the design argument the project is built
on.

### Three motion languages

| Surface | Physical reality | How it moves |
|---|---|---|
| **Bag display** | E-ink. Refreshes in a visible step, no sub-frame greys | Snaps. No eased position changes. A brief inverted flash on change, then settle |
| **Phone** | OLED, 120Hz, held close | Smooth and quick. Eased slides, crossfades, spring-settled numbers |
| **Page harness** | A drafting table, not a device | Slow, quiet, deliberate. Reveal on load, nothing after |

If you are about to animate something, ask which of these three surfaces it
lives on, and use that language.

---

## 2. Motion tokens

Add to `src/app.css` alongside the existing tokens.

```css
/* Durations */
--t-instant:  0ms;     /* e-ink state change, no tween */
--t-flash:    90ms;    /* e-ink refresh flash */
--t-quick:    140ms;   /* phone taps, hovers, focus */
--t-base:     240ms;   /* phone panel changes */
--t-slow:     420ms;   /* page reveals, pairing line */
--t-ambient:  1200ms;  /* background, never blocks input */

/* Easings. Named for what they feel like, not the curve. */
--e-snap:     steps(2, end);                        /* e-ink */
--e-exit:     cubic-bezier(0.4, 0.0, 1, 1);         /* leaving, accelerate out */
--e-enter:    cubic-bezier(0.0, 0.0, 0.2, 1);       /* arriving, decelerate in */
--e-settle:   cubic-bezier(0.2, 0.9, 0.3, 1);       /* phone, slight overshoot */
--e-draft:    cubic-bezier(0.65, 0, 0.35, 1);       /* page, symmetrical, calm */
```

**Never use `ease`, `ease-in-out`, or `linear`.** The browser defaults are
the motion equivalent of Arial.

---

## 3. Surface by surface

### 3a. Bag display: the e-ink rule

Everything on the bag screen changes state in **one frame**. No eased
transitions of position, size, or colour.

The one permitted effect is an **e-ink refresh flash**: when a value changes,
briefly invert that element, then return. Real e-ink does this, it is
visually distinctive, and it draws the eye to exactly what changed, which
is good interaction design rather than decoration.

```svelte
<script>
  // Flash the element when `value` changes, mimicking an e-ink refresh.
  // The flash is the ONLY motion permitted on the bag display.
  let flashing = $state(false);
  let previous = value;

  $effect(() => {
    if (value !== previous) {
      previous = value;
      flashing = true;
      setTimeout(() => (flashing = false), 90);
    }
  });
</script>

<span class="count" class:flash={flashing}>{value}</span>

<style>
  .count {
    /* colour swaps instantly at the midpoint, it does not fade through grey */
    transition: background-color var(--t-flash) var(--e-snap),
                color var(--t-flash) var(--e-snap);
  }
  .count.flash {
    background: var(--ink);
    color: var(--paper);
  }
</style>
```

**Club rack slots** change fill with the same step easing. A club leaving its
slot does not fade out, it is simply gone on the next refresh.

**The alert line** is the one place a tiny concession is allowed: it may
appear with a 90ms step reveal so it does not pop in mid-sentence. No slide,
no fade.

### 3b. Phone: smooth and quick

This is where the satisfying motion lives, and it is justified because an
OLED phone genuinely moves like this.

**Tab switching** uses a directional slide. The content moves in the
direction you navigated, which is a real spatial cue rather than decoration.

```svelte
<script>
  import { fly } from "svelte/transition";
  import { cubicOut } from "svelte/easing";

  const tabs = ["round", "setup", "summary"];
  let tab = $state("round");
  let direction = $state(1);

  function go(next) {
    direction = tabs.indexOf(next) > tabs.indexOf(tab) ? 1 : -1;
    tab = next;
  }
</script>

{#key tab}
  <div
    class="tab-body"
    in:fly={{ x: 24 * direction, duration: 240, easing: cubicOut }}
    out:fly={{ x: -24 * direction, duration: 140, easing: cubicOut }}
  >
    <!-- tab content -->
  </div>
{/key}
```

Note the asymmetry: 240ms in, 140ms out. Exits should always be faster than
entrances, because waiting for something to leave feels sluggish while
watching something arrive feels responsive. This single rule does more for
perceived quality than any easing curve.

**The club count on the phone** settles with a spring rather than snapping,
the opposite of the bag. Svelte 5 uses the `Tween` and `Spring` classes:

```svelte
<script>
  import { Tween } from "svelte/motion";
  import { cubicOut } from "svelte/easing";

  let { inBagCount } = $props();

  const shown = new Tween(inBagCount, { duration: 300, easing: cubicOut });
  $effect(() => { shown.target = inBagCount; });
</script>

<span class="phone-count">{Math.round(shown.current)}</span>
```

**Notifications** arrive with a slide from the top plus a fade, and leave by
fading only. Arrival deserves attention, departure does not.

**The club list reorders** with `animate:flip`, which is the single highest
value animation in the whole project. When a club moves between the in-bag
and out-of-bag lists, flip animates it physically travelling between the two
positions rather than disappearing and reappearing. Items need a keyed each
block for this to work:

```svelte
<script>
  import { flip } from "svelte/animate";
  import { fade } from "svelte/transition";
</script>

{#each clubsOut as club (club.id)}
  <li animate:flip={{ duration: 260 }} transition:fade={{ duration: 140 }}>
    {club.name}
  </li>
{/each}
```

### 3c. Page harness: quiet and deliberate

**On load**, the regions reveal in sequence: labels first, then the bag, then
the phone, each offset by about 80ms. Use a fade plus a 6px rise, 420ms,
`--e-draft`. This runs once and never again. It makes the first impression of
the page feel composed, which matters because it is the first thing a grader
sees.

**The pairing line** between the bag and the phone in the placement graphic
draws itself once on load using `stroke-dasharray` and `stroke-dashoffset`.
Afterwards it can pulse very slowly, 1200ms, at low opacity, to signal a live
connection. This is the one ambient animation in the project, and it earns
its place by communicating state.

```css
.pair-line {
  stroke-dasharray: 4 4;
  animation: drift 1200ms linear infinite;
}
@keyframes drift {
  to { stroke-dashoffset: -8; }
}
```

**Nothing else on the page animates.** The harness is furniture.

---

## 4. The cross-device moment

The most impressive single interaction available here, and the one to open
the demo video with.

When the bag detects a left-behind club, the alert should visibly *travel* to
the phone: the bag alert appears, then roughly 200ms later the phone
notification slides in, with the pairing line flashing once in between. The
delay is not decorative. It makes the causal chain legible, showing that the
bag sensed it and told the phone, rather than two unrelated things happening
at once.

Implement it as a short staged sequence in `App.svelte`, driven by the same
state change, not as two independent animations that happen to look
coordinated.

> **PAUSE AND ASK JONATHAN** before building this one. Ask whether he wants
> the 200ms relay delay, or both devices reacting simultaneously. The relay
> tells a clearer story but is slightly artificial, since real pairing is
> faster than that.

---

## 5. Interaction states, which matter more than animation

Polish that users feel but rarely name. These are worth more than any
transition, and they are frequently what separates a project that looks
finished from one that does not.

Every interactive element needs four distinct states:

```css
.control {
  transition: background-color var(--t-quick) var(--e-enter),
              border-color var(--t-quick) var(--e-enter);
}

.control:hover { /* a tone shift, never a size change */ }

.control:active {
  /* press feedback. 1px down, nothing more. Must be instant, 0ms */
  transform: translateY(1px);
  transition: none;
}

.control:focus-visible {
  /* NEVER the browser default blue. Use the project palette. */
  outline: 2px solid var(--flag);
  outline-offset: 2px;
}

.control:disabled { opacity: 0.4; cursor: not-allowed; }
```

Three rules:

- **Press feedback is always instant.** A tweened press feels broken, because
  nothing in the physical world has latency between touch and response.
- **Hover never changes size.** Growing buttons shift neighbouring layout and
  look amateur. Change tone instead.
- **`:focus-visible`, not `:focus`.** `:focus` shows rings on mouse clicks
  too, which is why people disable outlines entirely and break keyboard
  navigation.

---

## 6. Reduced motion, which is graded

This is an accessibility requirement in an HCI course, not an optional
courtesy. Some users get motion sickness from animation.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Svelte transitions need handling separately, since they are JavaScript driven:

```svelte
<script>
  import { MediaQuery } from "svelte/reactivity";
  const reduced = new MediaQuery("(prefers-reduced-motion: reduce)");
</script>

<div transition:fly={{ y: reduced.current ? 0 : 24,
                       duration: reduced.current ? 0 : 240 }}>
```

**Test it.** Windows: Settings, Accessibility, Visual effects, turn off
Animation effects. Then reload and confirm everything still works and nothing
disappears. Mention this test in your write-up, since most student projects
skip it entirely.

---

## 7. Banned

- Anything on the bag display that eases, slides, fades, or bounces
- `ease-in-out`, `ease`, or `linear` as an easing value
- Animations longer than 420ms on anything a user is waiting for
- Hover effects that change an element's size or position
- Looping animations other than the one pairing-line drift
- Parallax, scroll-triggered reveals, typewriter text, confetti, particles,
  3D card tilts
- Transitions on page load for anything other than the one staged reveal
- Animating `width`, `height`, `top` or `left`. Use `transform` and `opacity`,
  which are the only two properties the browser can animate cheaply

---

## 8. Where this fits in the build

Motion is a cross-cutting concern, not a phase of its own. Apply it as each
phase is built:

| Phase | Motion work |
|---|---|
| 0 | Add the motion tokens and the reduced-motion block to `app.css` |
| 1 | The staged page reveal on load |
| 2 | E-ink flash on the count, step transitions on rack slots |
| 3 | Phone tab slide, notification arrival, spring-settled count |
| 4 | `animate:flip` on the club lists, the cross-device relay |
| 5 | Simulation progress indicator, which may tween smoothly |
| 6 | Pairing-line draw-on and drift |
| 7 | Full interaction-state pass, reduced-motion test |

---

## 9. How to tell whether it worked

Two checks, both honest:

**The screenshot test.** Take a screenshot with nothing animating. If the
design falls apart without motion, the motion was propping up a weak layout.
Motion should be the last 10%, never the foundation.

**The second-time test.** Watch the demo twice. Anything that was delightful
the first time and irritating the second is too slow, too large, or should
not be there. Most animation that gets cut deserved to be cut for this
reason.
