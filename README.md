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

## Bugs found and fixed along the way

- **Vertical clipping on hover/press/focus (Prototype C).** The row's height
  exactly equalled the button height with zero vertical slack, so any
  focus-visible outline or the hover/press scale-up got clipped top and bottom.
  Fixed by growing the row 16px taller (8px clearance each side) and vertically
  centering its contents — covers the worst case (hovering an already-focused
  button, ~6px of combined outline + scale growth).
- **Same clipping bug, different element.** Individual chips had the identical
  risk from `chip-track`'s own scroll container — and per the CSS spec, a
  horizontally-scrollable element (`overflow-x: auto`) forces the other axis to
  also clip, even if you never asked for `overflow-y`. Fixed the same way (extra
  height + `align-items: center`), since disabling the clipping outright isn't
  possible while keeping the row scrollable.
- **CSS specificity bug: disabled arrows didn't actually look disabled in B/C.**
  `.nav-btn[data-disabled="true"] { opacity: 0.35 }` had the *exact same*
  specificity as `.variant-b .nav-btn { opacity: 1 }` / `.variant-c .nav-btn
  { opacity: 1 }`, and those variant rules were declared later in the file — so
  they silently won every time. The logic (JS `dataset.disabled`, shake
  feedback, scroll blocking) was always correct; only the visual opacity was
  wrong. Fixed by adding a `button` type-selector to bump specificity so it wins
  regardless of source order.
- **Missing accessible labels.** The search inputs had a placeholder but no
  `aria-label`, and the filter chips never communicated their selected state to
  assistive tech (no `aria-pressed`) — both flagged by automated accessibility
  checks. Fixed.

## Design tokens

Colors, radius, spacing, and typography are pulled from the SWAN tokens used in
the source Figma file (`sem/color/border/action`, `sem/border/radius/action`,
`comp/button/secondary/*`, `sem/focus/color/outer`, etc.) — see the CSS custom
properties at the top of `index.html`.
