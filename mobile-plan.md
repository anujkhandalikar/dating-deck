# Mobile friendliness plan (modern phones)

## Goals (what “mobile friendly” means here)
- **Modern phones first**: optimize for ~375–430px width, touch input, portrait orientation.
- **Tap-first UX**: all interactions work with visible, thumb-friendly buttons (no required swipes).
- **Single-column**: one “card/slide” per screen; no horizontal overflow; minimal side-by-side layouts.
- **Readable + fast**: large type, safe line-length, stable layout, avoid hover-only affordances.
- **Allowed to redesign**: adjust spacing/typography/layout to improve mobile UX.

## Current state (quick notes)
- **Deck is fixed-size and scaled**: `.slide-inner` is `1024x576` and scaled with `--deck-scale` based on viewport (`resizeDeck()`).
- **Layouts assume desktop**: multi-column grids (`.two-column`, `.tiled-content`, `.timeline-layout`, `.spec-two-column`) and fixed heights for image frames.
- **Hover effects**: `.slide-container:hover .image-wrapper img` won’t translate to mobile.
- **Ticker is fixed**: bottom ticker can overlap content on small screens (body has padding-bottom, but needs verification across devices).

## Strategy
### Responsive architecture
- **Keep desktop “deck scale” for wide screens**: continue using the current fixed 1024×576 presentation on desktop.
- **Switch to “mobile mode” under a breakpoint** (e.g. `max-width: 768px` or based on `pointer: coarse`):
  - Stop scaling a fixed canvas; instead make slides flow naturally with responsive sizing.
  - Treat each `.slide-container` as a vertical section with content sized for the device.

### CSS approach (no framework)
- **Add a mobile media query** that overrides:
  - `.slide-container`: `height: auto; min-height: 100svh; overflow: visible;`
  - `.slide-inner`: `width: 100%; height: auto; position: relative; transform: none; padding: 20px 16px; box-shadow` reduced.
  - Remove/disable `--deck-scale` usage in mobile mode (keep it for desktop).
- **Reflow multi-column → single-column**:
  - `.two-column`, `.spec-two-column`: `grid-template-columns: 1fr; gap: 16px;`
  - `.tiled-content`: `grid-template-columns: 1fr;` (or `1fr 1fr` only on larger phones if it still reads well).
  - `.timeline-layout`: stack items vertically; convert the horizontal line into a left border or remove decorative line.
- **Typography scaling**:
  - Use `clamp()` for headings/body to avoid giant desktop sizes on mobile.
  - Ensure line-height and spacing support one-handed reading.
- **Tap targets + spacing**:
  - `.ask-button`, `#scan-btn`, links: minimum 44px height, generous padding, obvious pressed state.
  - Replace hover-only effects with `:active` / `:focus-visible` equivalents.
- **Images and fixed heights**:
  - Convert fixed heights (`320px`, `180px`, `330px`) to responsive (`min()` / `clamp()`), and allow `height: auto` where possible.
  - Ensure images don’t force overflow; keep `object-fit: cover` but avoid cropping critical faces (tune per image if needed).
- **Safe areas**:
  - Add padding using `env(safe-area-inset-*)` for iPhone notches/home indicator.
  - Ensure the ticker and bottom CTAs don’t collide with the home indicator.

### JS adjustments
- **Resize logic**:
  - In mobile mode, skip applying `--deck-scale` (or set scale to `1` and remove transform).
  - Consider using `matchMedia('(max-width: 768px)')` to gate behavior.
- **Reduce layout thrash**:
  - Debounce `resizeDeck()` on resize/orientationchange if needed.

## Deliverables
- **`index.html` CSS updates**: add a mobile breakpoint section that converts the deck into a responsive, single-column layout.
- **`index.html` JS update**: modify `resizeDeck()` to be desktop-only.
- **Visual polish**: pressed/active states, reduced shadows, consistent spacing.

## Test plan (manual)
- **Viewport sizes**: 390×844 (iPhone 14), 393×873 (Pixel), 430×932 (iPhone Pro Max).
- **Orientation**: portrait + landscape (ensure no horizontal scroll).
- **Tap-only**: scan button works, CTA button clickable, links accessible.
- **Content checks**:
  - Slide titles don’t overflow.
  - Two-column sections stack cleanly.
  - Timeline reads logically when stacked.
  - Ticker doesn’t cover essential content and respects safe area.
- **Accessibility basics**:
  - Focus visible on interactive elements.
  - Reasonable contrast for small text.

## Open questions to resolve while implementing
- **“Scope unknown”**: we’ll prioritize the main deck experience first, then adjust any secondary pages/files if discovered.

