# Hackathon — MiniX ACC with V2V Fallback

Adaptive Cruise Control on **MiniX**, a 1:10-scale self-driving-car demonstrator
(Jetson Orin Nano Super + Arduino UNO), following a mecanum RC **target car**
that streams its speed and acceleration over Wi-Fi (V2V). ACC runs on camera +
ultrasonic by default and falls back to V2V when the camera can't read the
road or the target.

Source code: [SreekumarC/minix](https://github.com/SreekumarC/minix), branch
`feature/acc-v2v`.

## Contents

| File | What it is |
|---|---|
| [`architecture/MiniX_ACC_System_Architecture.pdf`](architecture/MiniX_ACC_System_Architecture.pdf) ([PNG](architecture/MiniX_ACC_System_Architecture.png)) | System architecture — ego and target vehicles, V2V link, ACC modes |
| [`wiring/ACC_Target_Wiring_RevC.pdf`](wiring/ACC_Target_Wiring_RevC.pdf) ([PNG](wiring/ACC_Target_Wiring_RevC.png)) | Target vehicle wiring — ESP32, MPU-6050, OLED, power, Hall sensors |
| [`docs/Hall_Sensor_Connection.md`](docs/Hall_Sensor_Connection.md) | A3144 Hall wheel sensor connection, mounting, firmware settings and test |

![System architecture](architecture/MiniX_ACC_System_Architecture.png)
