# Bring-up tests

One file, one subsystem. Each is a **complete, standalone MakeCode program** — paste it into a fresh project, flash, watch the robot. No Bluetooth, no web app, no other files.

They exist because a fault in the full firmware is hard to localise: the app connects, commands are acknowledged, and nothing moves. These narrow it to one subsystem in about a minute.

## Order to run them

Work outward from what needs least. If an early test fails, later ones will too and their results mean nothing.

| # | File | Extensions | Answers |
|---|---|---|---|
| 1 | `test-leds.ts` | mecanum_robot_v2 | Is the I2C driver alive and powered? |
| 2 | `test-motors.ts` | mecanum_robot_v2 | Do all four wheels turn, in the right directions? |
| 3 | `test-servo.ts` | mecanum_robot_v2 | Does the head sweep? What is the right trim? |
| 4 | `test-ultrasonic.ts` | mecanum_robot_v2 | Does the sensor echo, and which read routine suits it? |
| 5 | `test-line.ts` | mecanum_robot_v2 | Do the three line sensors read cleanly? |
| 6 | `test-neopixel.ts` | mecanum_robot_v2, Neopixel | Does the strip light, and on which pin? |
| 7 | `test-microbit.ts` | none | Do the on-board sensors report? |
| 8 | `test-ble.ts` | Bluetooth | Does Bluetooth work at all, with no robot involved? |

`test-leds.ts` first is deliberate: the headlights are the cheapest thing on the I2C bus. If they work, the driver has power and you are on the right extension — which removes the two most common causes of "nothing moves" before you look at anything else.

## Things worth knowing before you start

**The motor rail is not the micro:bit's rail.** USB alone powers the micro:bit, the display and the on-board sensors. It does **not** power the motors, the servo, the headlights or the ultrasonic. A robot can look completely alive and still be unable to move: check the switch on the expansion board.

**V1 and V2 are different extensions.** These tests all use `mecanumRobotV2`. Code written for the V1 extension compiles cleanly and then does nothing, because its driver sits at a different I2C address (`0x47` vs `0x30`) and never acknowledges. That failure looks exactly like a flat battery.

**Three subsystems fight the LED display.** P3, P4 and P10 (line sensors) and P7 (NeoPixel strip) are all matrix pins. `test-line.ts` and `test-neopixel.ts` therefore call `led.enable(false)` and report through the headlights, the buzzer or serial instead of the screen. That is not a limitation of the tests — it is the constraint the real firmware lives with.

## Pin map

| Function | Pin |
|---|---|
| Line sensors left / center / right | P3 / P4 / P10 |
| NeoPixel strip (4 LEDs) | P7 &nbsp;*(P8 on a V1 chassis)* |
| Ultrasonic trigger / echo | P15 / P16 |
| Head servo | P14 |
| Motors, headlights | I2C `0x30` |

## Reading a failure

Each file's header comment lists what **PASS** looks like and what the common **FAIL** modes mean. Start there rather than from the symptom — several distinct faults present identically from across the room, which is the whole reason these files exist.
