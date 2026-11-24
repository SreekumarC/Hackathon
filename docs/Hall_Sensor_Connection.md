# A3144 Hall Wheel Sensors — Connection Details (ACC Target Vehicle)

Status: **planned, not yet fitted.** The target firmware currently runs with
`HAS_HALL_SENSORS = false` (speed from the MPU-6050 IMU only). Fitting the two
Hall sensors gives the V2V link an accurate wheel speed and distance.

Wiring diagram: [`../wiring/ACC_Target_Wiring_RevC.pdf`](../wiring/ACC_Target_Wiring_RevC.pdf)

## Parts

| Qty | Part | Note |
|---|---|---|
| 2 | A3144 Hall switch (TO-92) | one per wheel, left and right |
| 2 | 10 kΩ resistor | pull-ups (brown-black-black-red) |
| 4–8 | Small neodymium magnets | `MAGNETS_PER_REV` per wheel hub |

## A3144 pinout

Hold it with the **flat, marked face towards you** and the legs pointing down:

| Pin | Name | Connect to |
|---|---|---|
| 1 (left) | VCC | **5 V** (LM2596 buck output) |
| 2 (middle) | GND | common GND |
| 3 (right) | OUT | ESP32 input + 10 kΩ pull-up to **3V3** |

## ESP32 connections (DevKit V1, 30-pin)

| Sensor | OUT → ESP32 | Pull-up | VCC | GND |
|---|---|---|---|---|
| Hall LEFT | **D34** (GPIO34) | 10 kΩ D34 → 3V3 | 5 V | GND |
| Hall RIGHT | **D35** (GPIO35) | 10 kΩ D35 → 3V3 | 5 V | GND |

Why it's wired this way:
- **A3144 needs ≥ 4.5 V**, so it's powered from the 5 V buck output, not 3V3.
- Its output is **open-collector** (it only pulls low), so the pull-up to 3V3
  keeps the ESP32 pin at a safe 3.3 V even though the sensor runs on 5 V.
- GPIO34/35 are input-only and have **no internal pull-ups**; the external
  10 kΩ resistors are required, or the pins float and give false pulses.
- One common GND for battery −, buck, ESP32 and both sensors.

## Mechanical

- Glue the magnets evenly spaced on each wheel hub, **same pole** facing out
  (the A3144 switches on a **south** pole facing its marked face).
- Mount each sensor so the marked face passes the magnets at **2–4 mm**.
- Keep the sensor leads away from the motor wires; twist OUT with GND if long.

## Firmware settings (`ACC_Target_Telemetry.ino`)

```cpp
const bool    HAS_HALL_SENSORS = true;     // enable once wired
const float   WHEEL_DIAMETER_M = 0.065f;   // measure over the mecanum rollers
const uint8_t MAGNETS_PER_REV  = 4;        // magnets on each hub
```

The sketch counts falling edges (magnet arrives) on D34/D35 with a 1.5 ms
debounce; a wheel with no pulse for 0.4 s reads 0 m/s. Distance per pulse =
π × wheel diameter / magnets per rev (0.051 m with the values above). Wheel
speed is fused with the IMU: the IMU gives fast response, the wheels remove drift.

## Test

1. Flash with `HAS_HALL_SENSORS = true`, open Serial Monitor (115200).
2. Turn each wheel slowly by hand and watch the 1 Hz Serial line: `v` rises
   while it turns and returns to 0 within 0.4 s of stopping.
3. Push the car 1 m in a straight line: the OLED `d` (distance) should read
   ~1.0 m. If not, correct `WHEEL_DIAMETER_M`.
4. No pulses at all → check the 10 kΩ pull-up, 5 V on VCC, magnet pole (flip it)
   and the air gap.
