# MiniX (Ego) — AS5600 Wheel Encoder Connection Steps

Status: **proposed, not fitted.** MiniX currently estimates its own speed from
the ISM330DHCX IMU (zero-velocity reset at every stop). Two AS5600 magnetic
encoders on the front wheels would give true ground speed and distance.

Diagrams in this folder:
- [`MiniX_AS5600_Mounting.pdf`](MiniX_AS5600_Mounting.pdf) — magnet/sensor mounting rules (section views)
- [`MiniX_AS5600_Front_Wheels.pdf`](MiniX_AS5600_Front_Wheels.pdf) — MiniX-specific mounting on the front (Traxxas) wheels

## Parts (per wheel)

| Qty | Part | Note |
|---|---|---|
| 1 | AS5600 module (I2C, fixed address **0x36**) | from the kit |
| 1 | Diametric magnet Ø6 × 2.5 mm | N and S across the face (kit magnet) |
| 1 | Hub cap Ø~20 mm | nylon spacer, bottle cap or 3D print |
| 1 | Arm, 2 mm aluminium strip × ~15 mm | bolted to the steering knuckle |
| — | M3 nylon/brass screws + standoffs | no steel near the magnet |

## Electrical connections (Jetson 40-pin header)

The AS5600 address is fixed (0x36), so the two sensors go on **two different
I2C buses**.

| AS5600 pin | S1 — front LEFT (I2C bus 7) | S2 — front RIGHT (I2C bus 1) |
|---|---|---|
| VCC (3.3 V) | pin **1** (3V3) | pin **17** (3V3) |
| GND | pin **9** | pin **25** |
| SDA | pin **3** | pin **27** |
| SCL | pin **5** | pin **28** |
| DIR | GND (fixes count direction) | GND |
| OUT / GPO | not connected | not connected |

Shared buses — no address clashes:
- **Bus 7** already has the OLED (0x3C) on pins 1/3/5/6 → S1 (0x36) joins in parallel.
- **Bus 1** already has the ISM330DHCX IMU (0x6B) on pins 17/25/27/28 → S2 (0x36)
  joins in parallel (splice or use a small I2C breakout; header pins are taken).
- Power the AS5600 from **3.3 V**, not 5 V (the Jetson I2C lines are 3.3 V).
- Keep leads under ~20 cm; leave a slack loop at each steering knuckle.

## Mechanical steps

1. **Use the front wheels.** They are not driven, so no wheelspin — they measure
   true ground distance. (Rear motor wheels are possible but can slip.)
2. **Find the wheel centre:** spin the wheel with a marker held still against it.
3. **Glue the hub cap** to the wheel centre with epoxy, then glue the magnet
   centred on the cap (within ~0.5 mm). The cap turns with the wheel.
4. **Make the arm** from 2 mm aluminium, bolt it to the **steering knuckle** so it
   steers with the wheel, and route it round the **front** of the tyre at axle height.
5. **Mount the AS5600** on nylon standoffs on the arm, chip facing the magnet,
   centred on the hub axis.
6. **Set clearances:** magnet-to-chip gap **1–2 mm** (0.5–3 mm works), arm
   **≥ 5 mm** from the tyre, motors and their magnets ≥ ~20 mm away.
7. **Check full steering lock** both ways: the arm must clear chassis, bumper
   and tie rod.

Outboard reach of the arm ≈ tyre outer face + cap (~6 mm) + magnet (2.5 mm) +
gap (1–2 mm) + board and standoffs (~6 mm). Measure on the car before cutting.

## Verify

```bash
i2cdetect -y -r 7      # expect 36 (AS5600 left) and 3c (OLED)
i2cdetect -y -r 1      # expect 36 (AS5600 right) and 6b (IMU)
```

1. Read the AS5600 STATUS register (0x0B): magnet detected, not too weak/strong.
2. Turn each wheel by hand: the angle (registers 0x0C/0x0D, 12-bit) must change
   smoothly through 0–4095 with no jumps.
3. Distance per wheel turn = π × wheel diameter (80 mm → ≈ 251 mm). Roll MiniX
   2 m on the floor and correct the diameter to the measured rolling distance.
