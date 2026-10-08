# Experimental C3 shared RGBW backend

Software candidate prepared on 2026-10-08 against WLED 16.0.1 commit
`29b389df1c1aaec6ff53aea742d17063b985906c`. It has not been uploaded or tested
on physical hardware. GPIO assignments and strip counts remain configuration;
none are hard-coded by these targets.

Copy the relevant environment sections from `platformio_override.sample.ini`
into ignored `platformio_override.ini`, then build:

```sh
npm ci
npm run build
npm test
pio run -e esp32dev -e esp32c3dev -e office_c3_inventory -e office_c3_rgbw4
```

`office_c3_inventory` retains the stock C3 LED driver and its two-output limit.
It adds read-only `c3Ota` fields to `/json/info`: reset reason, running app
address/size and next OTA partition size. This provides actual slot facts before
installing the experimental output backend. It is a locally built stock-driver
baseline with telemetry, not the official release binary.

`office_c3_rgbw4` enables `WLED_ENABLE_C3_SHARED_RGBW`. It accepts at most four
SK6812/WS2814 type-30 RGBW buses with driver preference 0. Other one-wire protocols
are rejected; SPI/PWM use the existing paths. The C3 still has two hardware TX
channels. This implementation uses one SDK RMT worker sequentially, with the
pinned NeoPixelBus SK6812 translator. WLED keeps its existing color, white,
brightness, gamma, current-limit, effects, presets and pixel readback processing.

Each logical bus owns editing/snapshot buffers. A frame barrier copies all
outputs after ABL processing and before any sends. The driver detaches the
previous GPIO, routes channel 0 to the next output, waits with a bounded timeout,
and holds the line LOW after a minimum reset interval. It does not save
configuration or reallocate buffers per frame. Driver faults latch until reboot
and return readiness so WLED teardown does not wait forever. If SDK shutdown
itself fails, ISR-visible buffers are retained instead of freed.

The shared target also exposes `c3Rgbw` diagnostics: bus count, physical TX and
worker counts, readiness/fault state, errors, completed transfers, frame count,
last/max frame duration and last SDK error. These counters require working
networking; they cannot capture an early boot failure. Start hardware validation
at 30 fps and low brightness. Actual timing and Wi-Fi responsiveness are unproven.

## Validation completed

- Untouched pinned stock C3 build passed separately.
- Regular ESP32, stock C3, inventory C3 and shared C3 firmware builds passed.
- `npm test`: 19 tests passed. Two new native settings regressions failed before
  the UI fix; all three shared-output settings tests pass after it.
- Actual settings page in an offline browser fixture: four-output validation and
  form submission, stock two-output rejection, pin conflict rejection, unsupported
  one-wire option rejection, desktop/phone rendering; no JS errors.
- Main browser UI using offline API fixtures: load, brightness, RGBW color,
  effect selection and settings navigation passed with no JS errors. The live
  0.15.1 GET snapshot was adapted to the pinned 16.0.1 segment-capability schema.
- Actual production driver compiled unchanged with a mocked SDK under host address
  and undefined-behavior sanitizers: frame snapshot/readback, no per-frame
  allocation, four-output handoff, excess/duplicate/protocol rejection,
  allocation failure, config/install/translator/route/write/wait failures,
  teardown failure and normal reinitialization passed in nine scenarios.
- Custom application image checksum/hash valid; ESP32-C3, DIO, 4 MB header,
  IDF `4.4.8.240628`. Both local images fit the standard 1,572,864-byte app slot.
  Actual installed slot sizes must still be obtained from the device.

Build/test logs, mock harness, offline browser fixtures, images and SHA-256
manifest are retained privately outside Git. Generated HTML headers and firmware
binaries are not committed. Mock results do not prove real RMT interrupts,
translator waveform, GPIO wiring, strip protocol or supply capacity.

## Deployment gates

Before a custom upload, validate the stock-driver baseline on the existing two
configured outputs and check actual OTA capacity. Preserve settings, presets,
Wi-Fi/AP credentials and known-good firmware separately. An app-only update does
not replace the installed bootloader/partition table or establish automatic
rollback. USB recovery is user-confirmed possible, but has not been rehearsed.

Keep the shared target on the original two-output configuration initially.
Enable additional outputs only after physical wiring/count/protocol verification.
Then verify independent RGB/white/effects, startup-off, settings persistence,
30-minute network/effects/DDP operation and a controlled reachable rollback.
None of these hardware gates has run yet.

Reference implementations: [pinned NeoPixelBus translator](https://github.com/Makuna/NeoPixelBus/blob/1d7ff38f14d04a976f9c6e365c83a232bcba04fc/src/internal/methods/NeoEsp32RmtMethod.h),
[IDF 4.4.8 C3 RMT API](https://docs.espressif.com/projects/esp-idf/en/v4.4.8/esp32c3/api-reference/peripherals/rmt.html),
[GPIO routing implementation](https://github.com/espressif/esp-idf/blob/v4.4.8/components/driver/rmt.c).
