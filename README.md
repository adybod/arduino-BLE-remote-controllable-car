
# Arduino BLE customizeable Controllable Car

A simple Arduino-based RC car that you can control over **Bluetooth Low Energy (BLE)**.

## What this project does

This repository contains the Arduino sketch (`RcCar.ino`) and supporting STL files for the Chassis(coming soon) to build a small car that can receive commands via BLE and drive accordingly.

## Features

- BLE-based remote control
- Simple command handling (forward / backward / left / right / stop)
- Easy to adapt to different motor drivers and pinouts

## Hardware:

- 1x Smartphone(with BLEJoyStick installed)
- 1x L298n motor driver
- 1x Arduino Uno
- 1x Motor tires
- 4x DC gear torque motors
- 2x 9 volt batteries
- 2x 9 volt battery connectors
- 1x HC-05 bluetooth module
- 1x plastic car chassis (source from amazon or (''))
- Double sided tape (for attachments)
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
- The Project is built for Arduino Uno, However any board with functioning Tx and Rx pins should do the job.

## License

See `LICENSE` for information about redistribution and usage rights.
