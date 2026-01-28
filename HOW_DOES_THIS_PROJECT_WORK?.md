## Overview

This project is an Arduino‑based, Bluetooth‑controlled RC car.  
Your **phone** sends simple control commands (like “forward” or “left”) over Bluetooth, the **Bluetooth module** passes those commands to the **Arduino**, and the **Arduino** drives the **DC motors** through an **L298N motor driver**.

At a high level, the system works as follows:

- **Phone app**: Sends characters over Bluetooth when you press on‑screen buttons or move a joystick.
- **Bluetooth module (HC‑05)**: Converts those wireless signals into serial bytes on its TX/RX pins.
- **Arduino**: Reads incoming characters from the serial port, interprets them as movement commands, and sets output pins accordingly.
- **L298N motor driver**: Uses those Arduino signals to switch higher‑current motor power on and off and to change rotation direction.
- **Motors & chassis**: Physically move the car in response to the driver’s outputs.

The rest of this document explains the **theory behind each part** and provides a **clear control-flow plan** for how the project should work from power‑up to driving the car.

---

## Hardware Theory

### 1. DC Motors and H‑Bridge Concept

- **DC motor basics**
  - A DC motor spins when a voltage is applied across its terminals.
  - If you **reverse the polarity** (swap + and −), the motor spins in the **opposite direction**.
  - Speed is roughly proportional to the **average voltage** it sees, which you can control using **PWM (pulse‑width modulation)**.

- **Why we need an H‑bridge**
  - The Arduino’s I/O pins cannot supply enough current to drive a motor directly.
  - An **H‑bridge** is a circuit made of 4 switches (usually transistors) that allow:
    - Forward rotation
    - Reverse rotation
    - Braking / stopping
  - The **L298N** contains two H‑bridges, so it can drive **two DC motors** independently (typically one on each side of the car).

### 2. How the L298N Motor Driver Works

The L298N sits **between**:

- A **power source** for the motors (e.g. a battery pack).
- The **DC motors** themselves.
- The **Arduino control pins**.

Each motor is controlled using **two input pins** and **one enable pin**:

- **IN1 and IN2**: Control Motor A direction.
- **IN3 and IN4**: Control Motor B direction.
- **ENA and ENB**: Enable pins for Motor A and Motor B, used for speed control via PWM.

Typical logic for one motor (e.g. Motor A):

- **IN1 = LOW,  IN2 = HIGH → Motor forward**
- **IN1 = HIGH, IN2 = LOW  → Motor backward**
- **IN1 = LOW,  IN2 = LOW  → Motor off (coast)**
- **IN1 = HIGH, IN2 = HIGH → Motor off (brake)**

The **ENA/ENB pins**:

- When **HIGH**: Motor is allowed to move (direction determined by IN1/IN2 or IN3/IN4).
- When driven with `analogWrite()` from the Arduino:
  - The duty cycle of the PWM controls the **effective voltage**, and therefore the **speed** of the motor.
  - Example: `analogWrite(ENA, 255)` → full speed, `analogWrite(ENA, 128)` → about half speed.

In this project:

- One side of the car (left motors) is typically wired as **Motor A**.
- The other side (right motors) is wired as **Motor B**.
- By driving the two sides in different directions/speeds, you can go **straight**, **turn**, or even **spin in place**.

---

## Bluetooth Theory

### 1. How Bluetooth Works (Conceptual)

Bluetooth is a short‑range wireless communication technology:

- It operates in the **2.4 GHz ISM band**.
- It is designed for **low‑power**, **low‑bandwidth** data transfer.
- Devices form a small network where one device often acts as **master/central** and others as **slaves/peripherals**.

For this project:

- Your **phone** is the **central** device (you control everything from here).
- The **Bluetooth module on the car** is the **peripheral**, waiting for commands.
- Commands are sent as **simple characters or short strings** that are easy to parse on the Arduino.

If you’d like a visual explanation of Bluetooth, this video is a good starting point:  
`[How Bluetooth Works](https://www.youtube.com/watch?v=1I1vxu5qIUM)`

### 2. Classic Bluetooth vs BLE (and what we use)

- **Bluetooth Classic (e.g. HC‑05)**
  - Acts as a **wireless serial cable**.
  - Data is sent over a virtual UART, so on the Arduino side you use `Serial` or `SoftwareSerial`.
  - Very simple to integrate: if you can read/write over serial, you can use it.

- **Bluetooth Low Energy (BLE)**
  - Optimized for **very low power** and **small bursts of data**.
  - Uses **GATT profiles** and **characteristics** instead of a continuous serial stream.
  - Requires different libraries and a more structured protocol.

In this specific build:

- The **HC‑05 module** is a **Bluetooth Classic** device.
- Even though the repository name mentions BLE, the HC‑05 behaves as a **serial‑over‑Bluetooth bridge**, which is exactly what we want for sending simple control characters.

---

## How the HC‑05 Bluetooth Module Works in This Project

### 1. Electrical Connection

The HC‑05 communicates using **UART (TX/RX)**:

- **HC‑05 TX → Arduino RX**
- **HC‑05 RX → Arduino TX**

This cross‑connection is critical:

- The **TX (transmit)** pin of one device must go to the **RX (receive)** pin of the other.
- If these are swapped incorrectly or left disconnected, the Arduino will **never receive any data**, and the car will not respond.

Power connections (typical):

- **VCC**: 5 V (or 3.3 V depending on module version — most HC‑05 breakout boards accept 5 V).
- **GND**: Common ground with Arduino and motor driver.

### 2. Data Flow

1. You open the **Bluetooth control app** on your phone.
2. You pair and connect to the **HC‑05** device.
3. When you tap a button or move the joystick in the app:
   - The app sends one or more **characters** (e.g. `'a'`, `'A'`, `'b'`, `'B'`).
4. These characters are transmitted wirelessly and received by the **HC‑05**.
5. The HC‑05 outputs these characters as serial data on its **TX** pin.
6. The Arduino reads them using its serial interface (`Serial.read()` or similar).

---

## Software Architecture on the Arduino

### 1. High‑Level Loop

The Arduino sketch (`RcCar.ino`) should conceptually do the following:

1. **`setup()`**
   - Initialize serial communication with the HC‑05 (e.g. `Serial.begin(9600)`).
   - Configure all motor driver pins (`IN1–IN4`, `ENA`, `ENB`) as `OUTPUT`.
   - Optionally stop all motors initially for safety.

2. **`loop()`**
   - Continuously check if new Bluetooth data is available:
     - `if (Serial.available() > 0) { char c = Serial.read(); }`
   - Interpret the received character using a **`switch`/`case`**:
     - `'a'` / `'A'`: drive forward.
     - `'b'` / `'B'`: drive backward.
     - `'l'` / `'L'`: turn left.
     - `'r'` / `'R'`: turn right.
     - `'s'` / `'S'`: stop.
     - (The exact mapping depends on how your app is configured.)
   - For each case, call small helper functions like:
     - `driveForward()`
     - `driveBackward()`
     - `turnLeft()`
     - `turnRight()`
     - `stopMotors()`

### 2. Motor Control Functions (Conceptual)

Each helper function sets the **L298N pins** to the correct pattern:

- **`driveForward()`**
  - Left side: IN1 = LOW,  IN2 = HIGH
  - Right side: IN3 = LOW, IN4 = HIGH
  - `analogWrite(ENA, speed);`
  - `analogWrite(ENB, speed);`

- **`driveBackward()`**
  - Left side: IN1 = HIGH, IN2 = LOW
  - Right side: IN3 = HIGH, IN4 = LOW
  - Same speed control via ENA/ENB.

- **`turnLeft()`**
  - Right side forward, left side slower or backward (depending on desired turning behavior).

- **`turnRight()`**
  - Left side forward, right side slower or backward.

- **`stopMotors()`**
  - All IN pins LOW (or braking pattern), ENA/ENB PWM = 0.

This separation of concerns keeps the code clear: **Bluetooth parsing** is independent from **motor control logic**.

---

## Phone App Behaviour and Command Mapping

Most Bluetooth RC‑car apps (both Android and iOS) work similarly:

- Each button or joystick direction **sends a character or small string**.
- There is usually an **info (`ℹ️`) screen** that documents which character corresponds to which button.

In the context of this project:

- **Example mapping** (you can adjust as needed):
  - `'a'` / `'A'`: left wheel or whole car forward.
  - `'b'` / `'B'`: left wheel or whole car backward.
  - Other letters may correspond to right wheel forward/backward, stop, etc.
- You then implement a `switch (incomingChar)` in your Arduino code and **assign each character to a movement function**.

Connection flow inside the app:

- Tap the **Bluetooth icon** (often at the top right).
- A list of nearby devices appears.
- Select the device named **“HC-05”** (or similar).
- Once connected, the app will show it is linked, and button presses will now send data to your car.

---

## Detailed Plan: How the Project Should Work End‑to‑End

### Step 1: Assemble the Hardware

- **Arduino**
  - Mount it securely on the chassis.
  - Ensure VIN/5 V and GND are accessible.

- **Motor Driver (L298N)**
  - Connect motor outputs:
    - OUT1/OUT2 → left motor(s).
    - OUT3/OUT4 → right motor(s).
  - Connect ENA and ENB to Arduino PWM pins (e.g. D5 and D6).
  - Connect IN1–IN4 to Arduino digital pins (e.g. D8–D11).
  - Provide motor supply voltage (e.g. 7–12 V) to the L298N `+12V` input.
  - Connect **GND** of the L298N to both:
    - Battery negative.
    - Arduino GND (common ground is required).

- **Bluetooth Module (HC‑05)**
  - VCC → Arduino 5 V (or as specified by your module).
  - GND → Arduino GND.
  - TX → Arduino RX.
  - RX → Arduino TX (use a voltage divider or level shifter if your HC‑05 RX is 3.3 V tolerant only).

### Step 2: Upload the Arduino Sketch

- Open `RcCar.ino` in the Arduino IDE.
- Make sure:
  - Serial baud rate matches the HC‑05 default (commonly `9600`).
  - Motor driver pin assignments in the code match your actual wiring.
  - The `switch`/`case` structure matches the characters the app sends.
- Upload the code to the Arduino.

### Step 3: Pair and Connect From the Phone

- Turn on the RC car (power the Arduino + L298N + HC‑05).
- On your phone:
  - Enable Bluetooth.
  - Pair with the device named **“HC-05”** (PIN is often `1234` or `0000`).
  - Open your Bluetooth RC app.
  - Select the HC‑05 from the device list and connect.

### Step 4: Drive and Test

- Press forward/back/left/right buttons in the app.
- Observe:
  - The Arduino receives characters over serial.
  - The code selects the appropriate movement function.
  - The L298N changes motor direction and speed.
  - The car moves as expected.

If the car does not respond:

- Double‑check **TX/RX wiring** (HC‑05 ↔ Arduino).
- Confirm **common ground** between all modules.
- Verify the **pin numbers** in the code match your actual connections.

---

## Summary

Conceptually, this project combines:

- **DC motor theory** (direction via polarity, speed via PWM).
- **H‑bridge motor driving** with the **L298N**.
- **Wireless serial communication** using the **HC‑05 Bluetooth module**.
- A simple **command protocol** (characters from the phone app) interpreted by the **Arduino**.

With the hardware wired correctly and the code mapping each incoming character to a clear motor action, you get a responsive, remote‑controllable RC car driven entirely over Bluetooth.

After all of this background knowledge for a simple rc car, I hope you will have taken something away from this project.
HAPPY BUILDING! :>