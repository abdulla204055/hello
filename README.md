# TurnstileGateESP32

Dual-leaf swing turnstile gate controller for **ESP32** (Arduino IDE).

This library controls **two BLDC Hall motors** (3 Hall channels each) via an external BLDC driver board that accepts:
- **PWM (0–5V recommended)**
- **DIR**
- **BRAKE / EL**

It performs:
- **Simultaneous calibration** to find 0-wall and 180-wall for both leaves
- **Synchronous move** with overshoot stop (sign flip) + stall/no-progress detection
- Easy high-level commands:
  - `open()`  → move to **180°**
  - `close()` → move to **90°**
  - `park()`  → move to **0°**
  - `moveTo(deg)` → move to any degree

---

## Hardware Notes (Important)

ESP32 GPIO is **3.3V**. Many BLDC driver boards (including common TZT 6–60V 400W boards) expect strong **5V logic** and may load the signal (often via an optocoupler input).

If you see:
- PWM high droops (e.g., 5V open-circuit but ~2.9V when connected),
- weak braking on BRAKE/EL,
- unreliable DIR,

then add a **3.3V → 5V buffer**:
- Best: **74AHCT** family (e.g., `74AHCT125`, `74AHCT14`, `74AHCT245`)
- Or a proper **push-pull transistor/MOSFET** stage.

Use the same buffer stage for **PWM, DIR, BRAKE**.

---

## Install

1. Download the ZIP of this library.
2. Arduino IDE → **Sketch → Include Library → Add .ZIP Library...**
3. Select the zip file.

---

## Quick Start

```cpp
#include <TurnstileGateESP32.h>

TurnstileGateESP32 gate;

void setup() {
  Serial.begin(115200);

  TurnstileGateESP32::Pins pins;      // defaults match common ESP32 wiring
  TurnstileGateESP32::Settings cfg;   // defaults match tested values

  // Example customization
  cfg.DIR_INV_L = true;
  cfg.brakeActiveLow = true;
  cfg.PWM_RUN = 200;

  gate.begin(pins, cfg);

  if (!gate.calibrate()) {
    Serial.println("Calibration failed!");
    while (true) delay(1000);
  }

  gate.close(); // go to 90
}

void loop() {
  gate.open();
  delay(1500);
  gate.close();
  delay(1500);
  gate.park();
  delay(1500);
  gate.moveTo(120);
  delay(1500);
}
```

---

## API

### `bool begin(const Pins& pins, const Settings& settings)`
Initializes GPIO, LEDC PWM, and interrupts.

### `bool calibrate()`
Runs simultaneous calibration (0-wall then 180-wall). Must be done before moves.

### `bool open()`
Moves to **180°**. If already near 180°, does nothing and returns `true`.

### `bool close()`
Moves to **90°**. If already near 90°, does nothing and returns `true`.

### `bool park()`
Moves to **0°**. If already near 0°, does nothing and returns `true`.

### `bool moveTo(long degree)`
Moves to any degree 0..180.

### `Settings`
All tunables can be changed from your sketch before calling `begin()` (or updated later using `setSettings()`):
- `PWM_RUN`, `PWM_SLOW`, `PWM_CAL`
- `START_KICK_PWM`, `START_KICK_MS`
- `STALL_MS`, `MOVE_TIMEOUT`, `NO_PROGRESS_MS`
- `WALL_MARGIN_STEPS`, `LEFT_90_MARGIN_STEPS`, `RIGHT_90_MARGIN_STEPS`
- `DIR_INV_L/R`, `POS_INV_L/R`, `PASS_DIR`, `brakeActiveLow`
- `NET_WIN_MS`, `NET_MIN_STEPS`, `NET_BAD_LIMIT`
- `pwmFreqHz`, `pwmResBits`

---

## Examples Included

- **BasicControl**: minimal setup + open/close/park/moveTo
- **CustomTuning**: shows how to change all tunables
- **OpenClosePark**: repeated open/close/park cycle
- **MoveToDegrees**: moves to a list of angles

---

## License: N/A
