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
0/476/948/1072 bytes. DDP with normal unmapped realtime addressing temporarily
takes over the physical outputs; the device scene resumes afterward.

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
or allocation leaves the prior configuration active. Config saves occur only
for explicit edits. Custom sampling has no per-frame allocation.

Grid sizes have a 16:10 aspect ratio, width 16..64 and height 10..40. The UI
provides 32x20, 48x30 and 64x40 choices. Stateful or audio-driven WLED effects
retain their native behavior and hardware requirements; a dropdown entry does
not establish that a physical audio source exists. Performance must be checked
on the actual C3 for the chosen effect and resolution.

The bounded read-only `/canvas/pixels?from=0&count=64` endpoint reports physical
bus readback and composed pixel values. Add `scene=1` to read scene pixels.
It exists for qualification and rejects ranges larger than 64 per request.

## Validation before deployment

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

Hardware canvas qualification, reboot/preset/DDP compatibility, sustained
operation and rollback remain deployment gates until their results are recorded.
Keep the previous qualified image and fresh config/preset backups outside Git.
Before reverting to firmware without this extension, disable canvas and verify
the ordinary four segment ranges; the old firmware cannot interpret virtual
canvas offsets. Preserve protected `wsec.json`, bootloader and partition table.

Reference: [WLED mapping](https://kno.wled.ge/advanced/mapping/),
[WLED segments](https://kno.wled.ge/features/segments/), and the pinned
`FX_fcn.cpp`, `FX_2Dfcn.cpp` and `usermods/user_fx` sources at WLED 16.0.1.
