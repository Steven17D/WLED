# WLED UI design round 2

Question: does an Apple-style fixed window with contextual controls fit Steven's
requested WLED interaction model?

Round 1's three directions were rejected. They remain captured in commit
`daaa4bbf`; the current prototype replaces them with a single integrated surface.

Run from this worktree:

```sh
python3 -m http.server 8094 --bind 127.0.0.1 --directory tools
```

Open http://localhost:8094/wled-ui-prototype.html.

The app frame stays the same size while controls change. The document cannot
scroll. Effects and palettes use searchable popovers with bounded lists. Presets,
orientation, color, timer and settings use contained sheets. Settings sections
replace the sheet content, with a Back control. Scene and strip controls use an
in-place segmented selector. The control inspector, strip list and sheets can
scroll within their own bounds. On phones, the dock scrolls horizontally.

Office strip geometry and selected effect come from a read-only 2026-10-10
snapshot. Artwork, preset scenes, power and brightness are demo values. Changes
are in memory and no requests go to the controller. Settings fields, discovery,
timer, sync and Peek show their interaction shape without operating hardware.

Round 2 validation: T3 browser checks passed 31 assertions for interactions,
fixed window geometry, popover bounds, selection, search, keyboard focus,
settings screens and request isolation. An additional 13 containment checks
passed at each of 728x700, 390x844 and 320x568. Native browser scroll actions
confirmed the inspector and effect list scroll while document scroll remains
zero. Desktop and compact-phone screenshots were inspected, along with the
settings sheet. The inline rendering has a fixed 700px height and was checked
at 728px width with no console errors.

The 19 existing Node tests passed in round 1; no production firmware assets or
build tooling changed in this round. Firmware qualification is still pending.

Keep this prototype on `wled-ui-facelift`. Implement the accepted interaction
model in production separately, preserve WLED's controls and configuration,
validate browser behavior, embedded size and firmware builds, then obtain
Steven's approval before merging to fork `main`.
