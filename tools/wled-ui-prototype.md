# WLED UI design round 1

Question: which structure should the everyday WLED interface use?

Run from this worktree:

```sh
python3 -m http.server 8094 --bind 127.0.0.1 --directory tools
```

Open http://localhost:8094/wled-ui-prototype.html?variant=A.

- A / Studio: effect library, central scene, right-side controls.
- B / Canvas: spatial scene first, supporting controls in a side rail.
- C / Quiet: light, spacious room controls with the scene alongside.

The bottom switcher and left/right keyboard arrows choose a direction. Arrow
keys remain native while inputs have focus. Variant URLs survive reloads.

Office strip geometry and the selected effect come from a read-only snapshot
on 2026-10-10. Preview power, brightness, effect artwork and presets are examples. All control changes are
in memory; no requests go to the controller. Settings entries are navigation
placeholders for this round. No firmware assets have been changed.

Keep this prototype on `wled-ui-facelift`. After a direction is chosen, implement
the production UI separately, validate controls, browser behavior, embedded size
and firmware builds, then obtain Steven's approval before merging to fork `main`.

Round 1 validation: all 19 existing Node tests passed after `npm ci`. T3 browser
checks passed 40 interaction/isolation assertions across the three variants,
including power, brightness, rotation, strip focus, path visibility, colors,
sample presets, effect search, empty search, settings navigation and keyboard
switching. All four views in each variant passed overflow checks at 728px and
360px; desktop controls were checked at 1280px. Desktop/mobile screenshots were
visually inspected. This is prototype validation, not firmware qualification;
no firmware code or embedded assets were changed or flashed.
