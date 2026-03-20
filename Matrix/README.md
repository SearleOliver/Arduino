# Arduino LED Matrix Clock — Temperature, Humidity & Snake Game

An Arduino project that drives an 8×8 LED matrix to display the current time, temperature, humidity, and a playable Snake game — all controlled via two buttons and a joystick.

---

## Features

- **Clock** — displays hours and minutes in real time using a DS3231 RTC module
- **Temperature** — reads and displays temperature (°C) from a DHT11 sensor
- **Humidity** — reads and displays relative humidity (%) from a DHT11 sensor
- **Snake game** — fully playable Snake on the 8×8 matrix, controlled with an analog joystick
- **On/Off toggle** — instantly turn the display on or off
- **Mode animation** — diagonal wipe animation plays on every mode switch

---

## Hardware

| Component | Description |
|---|---|
| Arduino (Uno/Nano) | Microcontroller |
| 8×8 LED Matrix (MAX7219) | Display |
| DS3231 RTC module | Real-time clock (I²C) |
| DHT11 sensor | Temperature & humidity sensor |
| Analog joystick (KY-023) | Snake game controls (VRX, VRY, SW) |
| Push button × 2 | Mode switch and On/Off toggle |

---

## Wiring

| Component | Arduino Pin |
|---|---|
| Matrix DATA_IN | 12 |
| Matrix CLK | 11 |
| Matrix LOAD (CS) | 10 |
| DHT11 data | 4 |
| Joystick VRX | A0 |
| Joystick VRY | A1 |
| Joystick button (SW) | 7 |
| Mode button | 2 |
| On/Off button | 3 |
| DS3231 SDA | A4 (Uno) |
| DS3231 SCL | A5 (Uno) |

---

## Dependencies

Install the following libraries via the Arduino Library Manager (`Sketch → Include Library → Manage Libraries`):

| Library | Usage |
|---|---|
| `LedControl` | MAX7219 LED matrix driver |
| `DHT11` | Temperature & humidity sensor |
| `DS3231` | Real-time clock module |

---

## Setup & Configuration

Before uploading, set the current date and time in `setup()`:

```cpp
myRTC.setYear(2024);
myRTC.setMonth(11);
myRTC.setDate(25);
myRTC.setHour(21);
myRTC.setMinute(54);
myRTC.setSecond(10);
```

Display brightness (0–15) can be adjusted by changing:

```cpp
#define BRIGHTNESS 8
```

---

## Usage

### Buttons

| Button | Action |
|---|---|
| **Mode** (pin 2) | Cycle through modes: Clock → Temperature → Humidity → Snake |
| **On/Off** (pin 3) | Toggle the display on or off |

A diagonal wipe animation plays each time the mode changes.

### Modes

| Mode | Display |
|---|---|
| 0 | Current time (HH MM) |
| 1 | Temperature (e.g. `23 °C`) |
| 2 | Humidity (e.g. `61 H%`) |
| 3 | Snake game |

### Snake Controls

Move the joystick in any of the four directions to steer the snake. The snake grows by one segment each time it eats the food dot. Food always respawns in a random position that does not overlap with the snake's body.

| Joystick direction | Snake movement |
|---|---|
| Left | Left |
| Right | Right |
| Up | Up |
| Down | Down |

---

## Display Layout

The 8×8 matrix is split into four 4×4 quadrants, each rendering one custom character:

```
┌────────┬────────┐
│ Digit  │ Digit  │  (top-left, top-right)
│   TL   │   TR   │
├────────┼────────┤
│ Digit  │ Digit  │  (bottom-left, bottom-right)
│   BL   │   BR   │
└────────┴────────┘
```

**Clock:** `H1 H2 / M1 M2`
**Temperature:** `T1 T2 / ° C`
**Humidity:** `H1 H2 / H %`

Custom characters defined in the `nombre[]` array:

| Index | Character |
|---|---|
| 0–9 | Digits 0 to 9 |
| 10 | `°` (degree symbol) |
| 11 | `C` |
| 12 | `%` |
| 13 | `H` |

---

## Code Structure

| Function | Description |
|---|---|
| `setup()` | Initializes matrix, RTC, sensor, snake start position |
| `loop()` | Reads buttons, updates time, switches modes |
| `displayTime()` | Extracts hour/minute digits and calls `updateMatrix()` |
| `displayTemperature()` | Extracts temperature digits and calls `updateMatrix()` |
| `displayHumidity()` | Extracts humidity digits and calls `updateMatrix()` |
| `updateMatrix(tl, tr, bl, br)` | Renders four digits into the four quadrants |
| `displayDigitRotated(digit, rowOffset, colOffset)` | Renders one 4×4 digit at a given position |
| `isButtonPressed(...)` | Debounced button read returning true on rising edge |
| `ModeChangeAnimation()` | Diagonal LED wipe animation on mode change |
| `snake()` | Reads joystick, moves snake, checks food collision |
| `generateFood()` | Spawns food at a random position not on the snake |
| `resetMatrix()` | Clears the entire LED matrix |

---

## Notes

- The RTC time is re-read from the DS3231 only when seconds roll over to 0, minimising I²C traffic.
- DHT11 readings are refreshed every loop iteration (when not in snake mode).
- Snake mode calls `return` early in `loop()` to skip sensor reads and avoid I²C interference during gameplay.
- Debounce delay is set to **200 ms** for both buttons.
- Maximum snake length is capped at **64 segments** (`MAX_LENGTH`).
