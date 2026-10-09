# Office C3 native WLED canvas

This optional extension layers a virtual scene over the qualified four-output
RGBW driver. Enable `WLED_ENABLE_SPATIAL_CANVAS` using the `office_c3_canvas`
example in `platformio_override.sample.ini`; no GPIO or driver changes are
required. Canvas mode defaults off until an explicit JSON state edit.

## Rendering and compatibility

WLED's existing 2D effects render a normal matrix segment named Scene. Four
ordinary 1D segments follow the matrix and use the registered Canvas Slice
effect. After WLED composes the grid, the renderer bilinearly samples each
strip's normalized straight path, including endpoint reversal and the white
channel. Multiple LEDs can sample the same cell. The four physical outputs
remain GPIO 2/3/5/6, lengths 119/118/31/31, starts 0/119/237/268.

At the default 32x20 resolution, Scene is segment 0 and the physical segments
are IDs 1..4, at logical starts 640/759/877/908. WLED's reported logical pixel
count becomes 939; `/json/info.canvas.physicalCount` remains 299. The canvas
state also advertises physical GPIO/range descriptors and the logical offset.
Pulsar must use those descriptors for names, and keep physical DDP offsets
0/476/948/1072 bytes. Realtime DDP always retains physical addressing, including
when the existing respect-ledmaps setting is enabled. The virtual grid is internal scene storage,
not a realtime LED map. DDP temporarily takes over the physical outputs; the
device scene resumes afterward. The editor pauses its scene preview during
physical realtime control so it does not misrepresent the host stream.

A physical segment can select a different WLED effect, brightness or power
setting while other strips sample Scene. Geometry changes preserve current
scene and strip settings. Disabling canvas restores the ordinary physical
ranges. Applying an existing preset with ordinary bus boundaries exits canvas
before replaying the preset. A legacy startup-off preset keeps canvas geometry
and applies its off state; new presets serialize the complete canvas state.

The `/canvas` page edits layout and native effect parameters. Edits are local
until Save layout. Accepted state changes are bounded and queued to the main
loop, where topology changes cannot race the effect service. A replacement
pixel buffer is allocated before releasing the previous one; rejected geometry
leaves the prior configuration active. Failure to allocate the replacement global
frame buffer is rejected before its previous buffer is released; segment buffers
retain WLED's standard allocation checks. Config saves occur only
for explicit edits. Custom sampling has no per-frame allocation.

Grid sizes have a 16:10 aspect ratio, width 16..64 and height 10..40. The UI
provides 32x20, 48x30 and 64x40 choices. Stateful or audio-driven WLED effects
retain their native behavior and hardware requirements; a dropdown entry does
not establish that a physical audio source exists. Performance must be checked
on the actual C3 for the chosen effect and resolution.

The bounded read-only `/canvas/pixels?from=0&count=64` endpoint reports physical
bus readback and composed pixel values. Add `scene=1` to read scene pixels.
It exists for qualification and rejects ranges larger than 64 per request.
An on-demand snapshot transfers ownership between the main loop and HTTP through
an atomic handoff; a first request can return retryable HTTP 503 while capture is
pending. `pixels` is WLED's lossy bus readback, which can return zero with ABL
disabled after its brightness-restoration factor rounds to zero. Use `composed`
for sampler validation; neither field proves the physical wiring or LED appearance.

Queued commands deep-copy HTTP-owned keys and strings while retaining numeric
values. Coordinates use stable four-decimal state output. Save layout persists
geometry; save a normal WLED preset to retain the complete scene/effect settings
across startup.

## Qualification on the Office controller

- Web UI build and 19 Node build tests passed.
- Custom C3 and standard `esp32dev` firmware compilation passed.
- Actual sampler header passed host AddressSanitizer/UndefinedBehaviorSanitizer
  checks for RGBW interpolation, boundaries, reversal, one-pixel paths and all
  299 physical samples. This is source-level qualification, not a hardware test.
- Offline browser fixtures expose 62 native 2D choices and verify all four strip
  rows, whole-path dragging, reversal, save, resize, rejected allocation and
  mobile overflow with no JavaScript errors. No device writes occur in fixtures.
- `git diff --check` passed. Generated HTML headers and firmware are excluded
  from Git.

- All 299 composed RGBW samples matched an independent bilinear sampler, including
  a diagonal path and reversal; white-channel preservation passed.
- All three editor resolutions preserved independent strip effects, brightness,
  effect checkboxes and blend settings.
- Existing ordinary presets exit canvas; new canvas presets restore it. A reboot
  preserves geometry, four GPIO outputs and startup off. Temporary qualification
  presets were removed and all original presets retained.
- Physical RGBW DDP matched all 299 intended composed pixels with the existing
  respect-ledmaps setting enabled. This regression failed on the preceding canvas
  build (0 of 299 matches) and passed with the correction.
- The actual device editor exposed all four names/counts and 62 native 2D choices
  without JavaScript errors. All 62 effects passed the live API/driver sweep at
  32x20; this does not qualify every effect at the maximum resolution or establish
  an audio input. Lowest sampled free heap was 106760 bytes.
- A full Wi-Fi downgrade to the previous qualified four-output image and return
  to this image passed, retaining the physical configuration and restoring canvas.
- Ten minutes alternating native scenes and physical RGBW DDP passed: 7242
  streamed frames, no reset or driver errors, minimum sampled free heap 107852
  bytes. Configuration and presets were unchanged during the test.
- The actual editor paused during realtime ownership and resumed after streaming
  with no JavaScript errors.
- Pulsar debug/release builds and 41 direct production-code checks passed against
  this device, including 90 dim frames and exact state/geometry restoration. Its
  full XCTest remains unavailable in the CLT-only toolchain. Steven explicitly
  waived that gate for this update; the stable-signed companion is installed.
  The live installed-app UI check awaits unlocking the Mac.

These live checks qualify composed software output and driver telemetry. Earlier
four-output driver qualification included physical observation; the new canvas
has not received a new visual observation. Private firmware/config/preset backups
and machine-readable qualification results are retained outside Git. The companion
update is installed under an explicit one-change XCTest waiver. Canvas is enabled
at dim brightness with Scene Rotozoomer and all four physical strips sampling
their paths; Pulsar Output remains off. The editor is available at
`http://office.local/canvas`.
Keep the previous qualified image and fresh config/preset backups outside Git.
Before reverting to firmware without this extension, disable canvas and verify
the ordinary four segment ranges; the old firmware cannot interpret virtual
canvas offsets. Preserve protected `wsec.json`, bootloader and partition table.

Reference: [WLED mapping](https://kno.wled.ge/advanced/mapping/),
[WLED segments](https://kno.wled.ge/features/segments/), and the pinned
`FX_fcn.cpp`, `FX_2Dfcn.cpp` and `usermods/user_fx` sources at WLED 16.0.1.

## Blank canvas on effect-metadata parse failure

The editor could stop initialization with `Expected ',' or ']' after array
element in JSON` because the existing `/json/fxdata` chunk callback required a
whole effect record to fit. When an outbound buffer was smaller than the next
record, it returned zero, which AsyncWebServer interpreted as the end of the
response. The resulting JSON array was truncated. Live browser reloads reproduced
this at different offsets, including 5519; Steven reported offset 8308.

The callback now keeps a per-response record offset and copies partial escaped
records into arbitrary buffer capacities. A bounded stack buffer holds one
record; no whole-response allocation or per-chunk heap allocation is added.
Returning zero is reserved for completion of the array.

An extracted-source host regression failed before the fix and passed afterward
under AddressSanitizer/UndefinedBehaviorSanitizer at callback capacities
1/2/3/7/31/64/128/1460 bytes, including escaped text and buffer guards. Both
firmware targets and all 19 Node tests passed. The fix was uploaded over Wi-Fi;
20 consecutive live browser loads populated all four strips with no malformed
JSON or JavaScript errors. Scene, layout, configuration and presets were restored
and checked against fresh pre-update snapshots. These fix-specific checks are
separate from the earlier ten-minute canvas/driver qualification above.
