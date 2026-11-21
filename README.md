```markdown
# Drone-Calibration

A PlatformIO/Arduino C++ project for an ESP32-S3 quadcopter flight controller. It includes RC receiver input handling, MPU6050 sensor reads, accelerometer and gyroscope calibration, complementary angle estimation, cascaded angle/rate PID control, and four-motor ESC output.

> **Safety warning:** This project can drive motor and ESC outputs. Remove propellers and use a bench supply or limited-power test setup while developing or calibrating the controller.

## Features

- Six-channel PWM receiver input
- MPU6050 accelerometer and gyroscope integration over I2C
- Gyroscope and accelerometer calibration utility
- Complementary roll and pitch angle estimation
- Cascaded PID control for roll, pitch, and yaw
- Four-channel motor output through ESP32Servo
- Automatic motor cutoff below the receiver throttle threshold
- Optional serial debugging output for sensors and PID values

## Hardware

The default PlatformIO environment targets:

- **Board:** Adafruit Feather ESP32-S3
- **Framework:** Arduino
- **Servo library:** madhephaestus/ESP32Servo
- **IMU:** MPU6050 at I2C address `0x68`
- **ESC/motor outputs:** Four PWM channels
- **Receiver:** Six PWM channels

| Component | ESP32-S3 pin |
|---|---|
| Motor 1 = 13 |
| Motor 2 = 12 |
| Motor 3 = 14 |
| Motor 4 = 27 |
| Receiver channel 1 = 34 |
| Receiver channel 2 = 35 |
| Receiver channel 3 = 32 |
| Receiver channel 4 = 33 |
| Receiver channel 5 = 25 |
| Receiver channel 6 = 26 |



## Calibration

`src/MPUCalibration.cpp` contains an IMU calibration utility.

Before calibration:

1. Remove all propellers.
2. Place the quadcopter flat and completely stationary.
3. Upload the calibration program.
4. Open the serial monitor at `115200` baud.
5. Follow the displayed instructions.
6. Copy the printed calibration values into `src/AngleModeFlightcontroller.cpp`.

The calibration output includes:

- `RateCalibrationRoll`
- `RateCalibrationPitch`
- `RateCalibrationYaw`
- `AccXCalibration`
- `AccYCalibration`
- `AccZCalibration`

Verify every offset on the bench before flight.

## Project Structure

```text
.
├── platformio.ini
├── src/
│   ├── main.cpp
│   ├── MPUCalibration.cpp
│   ├── AngleModeFlightcontroller.cpp
│   └── Drone Calibration.code-workspace
└── test/
    └── README
```

- `main.cpp` — Minimal PlatformIO Arduino entry point.
- `MPUCalibration.cpp` — Gyroscope and accelerometer calibration utility.
- `AngleModeFlightcontroller.cpp` — Receiver handling, IMU processing, PID control, and motor mixing.
- `test/README` — Placeholder information for future PlatformIO tests.

## Control Overview

The flight controller runs at approximately 250 Hz using a 4 ms control interval:

1. Read accelerometer and gyroscope data from the MPU6050.
2. Apply calibration offsets.
3. Calculate roll and pitch angles.
4. Estimate attitude with a complementary filter.
5. Convert receiver roll and pitch inputs into desired angles.
6. Run outer angle PID loops.
7. Run inner roll, pitch, and yaw rate PID loops.
8. Mix controller outputs into four motor commands.
9. Apply minimum, maximum, and throttle cutoff limits.

Motor positions:

- Motor 1: front right
- Motor 2: rear right
- Motor 3: rear left
- Motor 4: front left
