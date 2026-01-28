
# Arduino BLE Remote Controllable Car

A simple Arduino-based RC car that you can control over **Bluetooth Low Energy (BLE)**.

## What this project does

This repository contains the Arduino sketch (`RcCar.ino`) and supporting files to build a small car that can receive commands via BLE and drive accordingly.

## Features

- BLE-based remote control
- Simple command handling (forward / backward / left / right / stop)
- Easy to adapt to different motor drivers and pinouts

## Hardware (example)

- Arduino board (BLE-capable, or with a BLE module)
- Motor driver (e.g. L298N / TB6612FNG)
- DC motors + chassis + battery pack
- Jumper wires

## Getting started

1. Open `RcCar.ino` in the Arduino IDE.
2. Select the correct board and port.
3. Adjust pin definitions in the sketch if your wiring differs although the visual diagram of the circuit should be enough.
4. Upload the sketch to your board.
5. Pair/connect from a BLE controller app and send the expected commands.

## Notes

- If you change the command characters/strings in the sketch, make sure your controller app sends the same values.
- Motor drivers vary: double-check your driver’s datasheet and wiring before powering the motors.

## License

See `LICENSE` for information about redistribution and usage rights.
