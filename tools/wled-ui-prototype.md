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

## Round 3: five additional options

Steven accepted the round 2 design and requested five more options. The original
file stays unchanged. Open the new self-contained comparison at
http://localhost:8094/wled-ui-options-prototype.html?variant=1 using the same server.
The selector and arrows belong to the prototype comparison, not the product UI.

| Key | Name | Layout question |
| --- | --- | --- |
| 0 | Original | Accepted scene, inspector and bottom strip dock |
| 1 | Canvas | Should controls float over a larger scene? |
| 2 | Control Center | Should brightness and color lead, with a smaller scene? |
| 3 | Workbench | Should strip selection use a left rail, with tuning below the scene? |
| 4 | Studio | Should the scene use a dark surface and a bottom control tray? |
| 5 | Scenes | Should a preset library lead, with fine tuning in a sheet? |

Each key is shareable through `?variant=`. Demo light settings survive switches
in memory; reloading resets them. Left/right arrows cycle designs except while
editing inputs or using a dialog. No hardware requests or persistent writes occur.
Each variant retains power, brightness, RGBW color, effect/palette menus,
orientation, strip focus/direction, presets and contained settings screens.
Studio and Scenes also provide a Fine tune sheet for motion and white controls.

Validation: 132 browser assertions covered the six designs at 728x700; 36 checks
covered 390x844; 48 covered 320x568. After spacing and resize corrections, another
40 assertions covered fixed bounds, menus, nested settings, Fine tune focus,
state retention and URL selection. Native clicks and keyboard events confirmed
comparison arrows and that an arrow key adjusts a focused slider without
switching designs. Live resizing moved Canvas's strip dock between its desktop
and phone positions while root and window scroll stayed zero. Control Center's
phone strip selectors also appeared after resizing without a reload. Every new
layout was visually inspected on desktop and phone; inline variants were checked
at 728px with a fixed 700px frame and no console errors. `npm test` passed all 19
existing tests. Production assets and firmware are untouched by this round.

Requested omp Opus 5.5 help was attempted with
`omp --model claude-opus-5-5 --no-tools --no-session -p ...`. It failed before
generating a response: `No API key found for anthropic.` These designs therefore
have no Opus contribution. Authentication remains the blocker for that request.
Main integration still requires Steven's approval after production testing.
