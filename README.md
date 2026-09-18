# Font panel — horizontal filter scroll prototype

Working HTML/CSS/JS reference for three micro-interaction options solving the same
problem: users without a mouse capable of horizontal scroll can't navigate the
font-filter chips (`All`, `In this project`, `Traditional`, `Modern`, `Script`,
`Playful`, `Decorative`) in the font panel.

Companion Figma exploration: [Property Font panel → Findability improvements](https://www.figma.com/design/vHfx8g3ULHkHLew48sv7bX/Property-Font-panel-%3E-Findability-improvements?node-id=2292-17878),
page "👉 Iteration for Sliders/horizontal scrolling". This HTML build exists because
Figma's static-frame prototyping can't represent real scroll physics, drag, or
keyboard focus — this can.

## Running it

No build step, no dependencies. Either:

- Double-click `index.html` to open it directly in any browser, or
- Serve it locally: `python3 -m http.server 8934` from this folder, then open
  `http://localhost:8934/index.html`

## The three variants

| | Arrows | Behavior |
|---|---|---|
| **A) Hover to reveal** | Hidden at rest | Reveal on mouse hover (slide + fade, with a short linger before hiding to avoid flicker). Mouse-only by design — contrast case against B/C. |
| **B) Permanently there** | Always visible | Baseline "just make it discoverable" option. |
| **C) Always visible + tactile / keyboard-accessible** | Always visible | Same engine as B, plus a livelier hover/press scale animation and a real focus ring reachable via `Tab` — the actual fix for users with neither a horizontal-scroll mouse nor a trackpad. |

## Interaction engine (shared by all three)

- **Chip-aligned paging** — clicking an arrow computes the exact scroll position
  that brings the next/previous chip fully into view. It never leaves a chip cut
  in half at the edge.
- **Custom eased animation** — a hand-rolled `requestAnimationFrame` + ease-out-cubic
  tween drives the scroll, instead of relying on inconsistent native
  `scroll-behavior: smooth`.
- **Press-and-hold** — holding an arrow past ~360ms turns it into continuous
  scrolling (like a real slider) until release, then snaps to the nearest chip.
- **Drag-to-scroll** — the chip row itself can be dragged, with velocity-based
  momentum on release and a rubber-band bounce past either end.
- **`scroll-snap-type`/`scroll-snap-align`** — native wheel/trackpad/touch
  scrolling also snaps to chip boundaries, so the "never cut off" guarantee holds
  regardless of input method, not just via the buttons.
- **Disabled-state shake** — clicking an arrow with nothing left to scroll gives a
  small shake instead of doing nothing silently.

## Known trade-off

The "disabled" arrows at each scroll boundary use a visual `data-disabled`
attribute rather than the native `disabled` attribute, specifically so a click
attempt can still trigger the shake feedback. That means they remain technically
focusable/clickable via keyboard even when visually disabled. Fine for a
prototype; swap to a proper disabled/`aria-disabled` + `tabindex="-1"` pattern
before this goes near production.

## Design tokens

Colors, radius, spacing, and typography are pulled from the SWAN tokens used in
the source Figma file (`sem/color/border/action`, `sem/border/radius/action`,
`comp/button/secondary/*`, `sem/focus/color/outer`, etc.) — see the CSS custom
properties at the top of `index.html`.
