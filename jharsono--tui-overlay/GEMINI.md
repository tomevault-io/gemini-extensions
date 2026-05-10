## tui-overlay

> Composable overlay widget for Ratatui: drawers, modals, popovers, and toasts from a single `Overlay` primitive.

# tui-overlay

Composable overlay widget for Ratatui: drawers, modals, popovers, and toasts from a single `Overlay` primitive.

## Architecture

- `Overlay` (widget, builder) owns presentation config: anchor, size, slide, backdrop, block chrome, bg fill
- `OverlayState` owns lifecycle and timing: phase, progress, duration, easing
- Overlay does not own body content — dims, draws chrome, exposes `inner_area()` for caller
- `overlay_rect()` exposes the full bounding rect (including chrome) for hit testing
- `Phase` is `pub(crate)` — widget reads it, callers query via `is_open`/`is_closed`/`is_animating`
- `layout::resolve_rect` is the single layout entry point: constraints + anchor + offset → clamped `Rect`
- `clip_by_slide` clips overlay rect along slide axis by eased visibility fraction
- Backdrop: bg = base, fg = explicit override or derived from base via `|c| c / 2 + 64`
- `color_to_rgb` routes: Rgb direct, Indexed decoded (cube/grayscale/ANSI-16), Reset → black
- Duration/easing live on `OverlayState` (not `Overlay`) because `tick()` is called outside render
- Default duration = zero (instant transitions); non-zero enables Opening/Closing animation phases
- Interruption: phase flip preserves progress for smooth reversal
- Block render skipped when clipped rect is too small for borders (prevents panic during closing animation)
- `.bg(Color)` fills every cell in `overlay_rect` after `clear_region`, before block render — covers border cells that `Block` leaves transparent
- Demo uses `ratatui::crossterm` re-export (not direct crossterm dep)

## Crate Structure

```
src/
  lib.rs          overlay.rs      state.rs
  anchor.rs       layout.rs       backdrop.rs
  slide.rs        easing.rs
tests/
  integration.rs  ← public API tests (51), maps to docs/features/overlay/*.feature
examples/
  demo.rs         ← villain lair control panel with 7 presets and 5 speed settings
docs/features/overlay/
  01-05.feature   ← BDD specs: positioning, backdrop, animation, rendering, recipes
```

## Current Focus

`.bg(Color)` builder landed — consumers no longer need a manual `buf.set_style` workaround before rendering block content into an overlay without a backdrop. The taho `theme_drawer.rs` workaround can be removed.

No open work items.

---
> Source: [jharsono/tui-overlay](https://github.com/jharsono/tui-overlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-27 -->
