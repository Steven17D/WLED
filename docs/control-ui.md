# Canvas control interface

`/control` is a device-hosted, dependency-free dark interface. It keeps a fixed
outer window, a bounded scene, and separate scrolling areas for controls and
popup sheets. The existing `/` and `/canvas` interfaces remain available.

## Controls and data

- Power, brightness, three RGB colors, supported white/CCT channels, effects,
  palettes, effect parameters, transitions, timer, sync and realtime override.
- Scene and segment targeting, segment settings, independent strip effects and
  returning strips to the canvas scene.
- Preset save/apply/edit/delete, startup selection and editable playlists with
  duration, transition, repeat, shuffle and final-preset options.
- Scene orientation, controller pixel readback, strip sampling, editable path
  endpoints/reversal and canvas resolution. Layout changes stay in a local
  draft until Save; acknowledged geometry is read back from the controller.
- Native settings, custom palette editor, file manager, firmware update and
  advanced JSON commands inside bounded sheets. Native forms retain their
  existing validation, PIN challenges and firmware handlers.

The page reads the device's state, information, effect names/descriptors,
palette names/stops and preset file. Mutations run serially and read back the
fields that were written. A rejected command or an acknowledged no-op does not
become a successful local edit. Offline controls are disabled and stale live
frames are cleared.

The live canvas uses WLED's L1/L2 WebSocket frames, including the extra padding
in Office's 64 × 40 canvas response. The controller determines RGBW readback;
browser preview boost affects only the display. Strips with independent effects
are identified explicitly because the matrix frame cannot report their separate
physical output. The viewer does not synthesize those pixels.

Palette thumbnails use `/json/palx` stop positions, including custom palettes
and current color-derived palettes. Twenty-five effect thumbnails are embedded
reference stills from the pinned MIT-licensed WLED-Utils data used in the design
review. Other effects use a neutral icon. Reference thumbnails do not claim to
show the current palette, speed or effect frame; all controller effects remain
selectable, and the live scene shows controller frames.

## Design and interaction refinement

The October 10 refinement uses one content surface, a restrained system-font
hierarchy and brighter temporary surfaces. Room-wide power and brightness stay
in the toolbar; the inspector targets the selected scene or strip. Advanced
controls move into one reusable sheet. Effects, palettes and targets open
anchored pickers on desktop and larger sheets on phones. Only the inspector,
picker results, sheet body and horizontal strip dock scroll; the outer window
stays fixed.

This follows Apple's current [material hierarchy](https://developer.apple.com/design/human-interface-guidelines/materials),
[typography](https://developer.apple.com/design/human-interface-guidelines/typography),
[popover](https://developer.apple.com/design/human-interface-guidelines/popovers),
[sheet](https://developer.apple.com/design/human-interface-guidelines/sheets) and
[accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
guidance. The CSS is an original browser implementation, with original SVG
icons and a system font stack. It does not bundle Apple's fonts or symbols.

The displayed frame retains its aspect ratio. Painting and pointer mapping use
the same letterboxed bounds, and labels/handles retain screen-space size.
Ordinary path clicks select a strip without creating a draft. Explicit layout
editing alone enables dragging, with Save and Discard always available in the
scene and a full scene workspace on phones. Real changes create a draft;
reverting all geometry clears it. Offline editing permits local Discard and
requires reconnection before Save.

Stable strip buttons preserve keyboard focus during polling. Closing a picker
returns focus to its trigger; after a confirmed command, temporarily disabled
controls regain focus if it was lost to the document body. A newly focused
control or another open dialog retains focus. Reduced motion, transparency and
increased-contrast preferences have explicit CSS fallbacks. Phone controls have
44-pixel hit regions, including compact toolbar icons and color selectors.
Frame status remains visible on phones, and preview boost is also reachable
through More controls.

## Build

`tools/cdata.js` inlines the page, shared helpers, styles and reference stills into
`html_control.h`. `wled_server.cpp` serves the compressed page at `/control`.
Generated headers are not committed.

```sh
npm ci
npm run build
npm test
pio run -e esp32dev
```

Office qualification also uses the existing private `office_c3_canvas` override
and installed PlatformIO core/toolchain. This UI change does not alter the
output driver or canvas firmware implementation.

## Second local polish pass

Saved looks uses plain rows with effect or playlist metadata, a reserved Edit
action and fixed creation actions. Create/edit forms focus Name first; More
options and Stored command disclose secondary fields. Playlist steps use one
continuous form with accessible reorder actions. Settings groups the existing
destinations under Lighting, Connections and System. Effects and palettes
reveal the current selection on opening, with target context, a result count,
an explicit Clear search action and a designed empty state.

Closing or changing a sheet invalidates its pending navigation and focus
callbacks. Authorized writes still finish and confirm their controller state,
while delayed results cannot reopen a dismissed editor or place an error in a
different menu. Refresh, Save and Delete prevent duplicate pending actions.
Active ad hoc playlists with native ID 0 retain a reachable Stop action.

Three independent reviews passed after the final fixes. The interaction review
passed 59 checks, covering the three reproduced pre-fix issues (hidden current
picker selection, late library reads replacing Settings, and long preset names
displacing Edit), preset/playlist CRUD, timing and reorder payloads, focus,
delayed success/error guards and all 12 native settings Back paths. Temporary
simulator changes were removed and its initial state and presets matched
exactly. Visual checks at 1080 × 760, 390 × 844 and 320 × 568 found no horizontal
overflow or wrapped action footer, including expanded options and stored JSON.
Final code review found no unresolved P0/P1/P2 findings.

Read-only Office qualification connected to 220 effects, 72 palettes and live
frames, opened its native UI settings form, and restored focus on Back. A fresh
private snapshot of the current device state, configuration and presets matched
after these checks; driver errors remained zero. No device writes or firmware
upload were performed during this pass. The compressed control page is 34,640
bytes. Asset generation, all 19 existing Node tests and ESP32/Office C3 firmware
compilation passed after the final source changes.

## Apple-inspired refinement qualification

Three agents independently researched Apple's primary design guidance, reviewed
rendered layouts, exercised interactions and audited the source. Three visual
review rounds closed picker content overflow, small-phone scene collapse,
redundant scene framing, hidden selector cues and compact sheet action wrapping.
Final source review found no unresolved P0/P1/P2 findings.

Fresh browser checks covered desktop 1080 × 760, phone 390 × 844 and 320 × 568.
Every effect/palette cell contains its thumbnail, name and selection marker;
title/search/footer remain fixed while results scroll. Phone layout editing
uses the full workspace, with a 268 × 167.5-pixel preview even at 320 × 568.
Every checked phone toolbar, range, color and sheet action has a 44-pixel hit
region. Document bounds stay fixed and the strip dock stays inside the stage.

Simulator qualification covered stable polling focus, picker close/apply/error
focus return, advanced navigation cleanup, ordinary strip selection without a
draft, genuine drag/Discard, numeric Save/readback, global-versus-selected scope,
HTTP rejection, acknowledged no-op rejection, command queue recovery, bounded
pending readback, offline disable/frame clearing and reconnection. All fault
flags and modified simulator state were restored. Ninety independent geometry
assertions covered painting/pointer bounds across desktop/phone dimensions,
DPR 1/2 and multiple frame aspect ratios.

The refined page also connected to Office's actual 220 effects, 72 palettes and
live frames. A temporary 3% Aurora review scene was confirmed, then the original
off state, canvas geometry, configuration and presets were restored and compared
against a fresh private backup. Driver errors remained zero. No firmware was
uploaded. The asset build, 19 existing Node tests and ESP32/Office C3 firmware
compilation passed after the final source changes.

## Earlier October 10, 2026 functional qualification

The UI was tested locally against a simulator and against Office through an
explicit loopback SSH tunnel. No new firmware was uploaded.

- Existing Node build tests: 19 passed. Web resource build and ESP32/Office C3
  firmware compilation passed.
- Simulator: controls, segment isolation, preset CRUD, playlist editor, draft
  layout/save/discard, keyboard pickers, native form submission/navigation,
  disconnect/reconnect, HTTP rejection, successful HTTP no-op rejection and
  padded/truncated frame handling.
- Desktop and mobile: bounded scene, contained color ring, fixed header/footer,
  reachable last control and inner scrolling at 1080 × 760, 728 × 700,
  390 × 844 and 320 × 568.
- Actual Office: 220 effects, 72 palettes, RGB/white, dim brightness, effects and
  parameters, scene orientation, independent strip/follow scene, preset and
  playlist save/apply/delete, timer and layout changes.
- Native settings/editor pages loaded from the actual controller. Settings
  writes, network reconfiguration, hardware reconfiguration, file uploads and
  OTA were not performed on Office; form submission was exercised in the local
  fixture. These continue to use the existing native firmware flows.
- Original lighting state, canvas geometry, configuration and preset contents
  were restored and compared with the private pre-test backup. Driver errors
  remained zero.

Implementation stays on `wled-ui-facelift` for Steven's review. Main integration
and firmware installation require his approval after local testing.
