# Dark Canvas controls

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

## October 10, 2026 qualification

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
