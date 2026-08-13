# Keyes 4WD Mecanum RXY v2

Widget-driven Web Bluetooth control for the Keyestudio KS4031/KS4032 robot and BBC micro:bit V2.

The MakeCode firmware in `firmware/main.ts` owns the controller layout. After Bluetooth pairing, the web app verifies and downloads the embedded CFG, then renders its 31 widgets. Manual D-pad control supports forward, reverse, Mecanum strafing, and diagonals. The same layout includes motor speed, ultrasonic servo, left/right LEDs, three line sensors, distance telemetry, emergency stop, autonomous Line/Avoid modes, firmware version, and uptime, plus the NeoPixel strip (on/off, colour, brightness, Solid/Rainbow/Chase/Sparkle effects) and the micro:bit's own sensors (sound, temperature, tilt X/Y, buttons A/B and logo touch).

## Flash

Create a micro:bit MakeCode project, add Bluetooth, Neopixel and `github:keyestudio2019/mecanum_robot_v2`, then paste `firmware/main.ts` into JavaScript view and download it to the micro:bit. Set Bluetooth to **No Pairing Required**.

## Pin map

Read from the `mecanum_robot_v2` extension source and the KS4031 tutorial, not inferred. **V1 and V2 differ**, which is the single biggest source of wasted time on this robot:

| Function | V1 (KS4031 tutorial) | V2 (this board) |
|---|---|---|
| Line sensors | P1, P2 (two) | **P3, P4, P10 (three)** |
| Servo | P14 | P14 |
| Ultrasonic trig / echo | P15 / P16 | P15 / P16 |
| Infrared receiver | P9 | P9 |
| Motors / headlights | I2C `0x47` | I2C **`0x30`** |
| WS2812 RGB | P8 (documented) | **P7** (undocumented; found by boot scan) |

The V2 extension exposes no RGB or WS2812 function at all — `setLed(LedCount, LedState)` is on/off only for the two headlights, over I2C. Any addressable strip must be driven with the generic `neopixel` package, and Keyestudio does not say on which pin.

## The 5x5 matrix stays off

P3, P4, P7 and P10 are all LED-matrix row/column drivers. With the display enabled those pins are multiplexed and a digital read returns garbage, so **Line mode would steer on noise**. That is why the firmware calls `led.enable(false)` — the trigger is the line sensors, not the NeoPixel strip.

`USE_MATRIX = false` in `firmware/main.ts` is therefore the correct setting for this chassis.

For the same reason **light level is unavailable on this robot**. The micro:bit senses light with the LED matrix itself, so `input.lightLevel()` re-enables the display and reclaims P3, P4, P7, P9 and P10 — stealing P7 back from the NeoPixel strip and corrupting the line sensors. It was polled here briefly and did exactly that. Every `basic.show*` / `led.*` call is gated on it rather than deleted, so it remains a one-line decision, but turning the matrix back on costs you working line-following.

## NeoPixel strip

The 4-LED strip is on **P7**. Keyestudio does not document this for V2 — the V2 extension has no RGB or WS2812 function at all — so it was found empirically with the boot pin scan in `firmware/main.ts` (`NP_PIN_SCAN`, left in place behind a flag because these pin differences recur).

Do not use P8: that is the **V1** pin, from the KS4031 tutorial, and it does not apply to this board.

P7 is one of the six LED-matrix pins, so the display must stay off for the strip to work. That costs nothing here, because the matrix already has to be off for the line sensors — see below.

Effects are `Solid`, `Rainbow`, `Chase` and `Sparkle`, with nine colours and a brightness slider.

## Run

Serve this folder from `localhost` or HTTPS and open `index.html`. Web Bluetooth does not work reliably from a plain `file://` page.

Test initially with all four wheels raised. The firmware stops the motors on STOP, mode changes, stale drive commands, and detected link loss.
