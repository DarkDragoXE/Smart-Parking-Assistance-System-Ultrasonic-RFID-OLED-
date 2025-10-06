# Smart Parking Assistance System (Ultrasonic + RFID + OLED)

A microcontroller-based access and parking gate system that uses:
- **Ultrasonic ranging** to auto-open Gate 1 when a vehicle/person approaches.
- **RFID (MFRC522)** authentication to control Gate 2.
- **Dual servos** for two barrier doors.
- **OLED (SSD1306)** for on-device status messages.
- **Tri-color LED + buzzer** for clear user feedback.

All logic is implemented on Arduino-style firmware and is ready for an Aries V2–style board (separate SPI/I²C instances are configured).

---

## Features

- **Proximity-controlled entry (Gate 1):** Opens when distance ≤ 50 cm; closes automatically when clear.
- **RFID-controlled entry (Gate 2):** Valid UID opens the gate; invalid UID triggers buzzer and error on OLED.
- **Visual and audible feedback:** RYG LEDs indicate status; buzzer alerts on unauthorized attempts.
- **Compact UI:** 128×32 OLED shows init/status/denied messages.
- **Safe motion:** Timed servo moves with state tracking to avoid jitter.

---

## Bill of Materials

- Microcontroller board (Arduino-compatible; Aries V2 pinout assumed)
- Ultrasonic sensor (HC-SR04 or equivalent)
- Two servos (e.g., SG90/MG90S)
- MFRC522 RFID reader + 13.56 MHz tags/cards
- 0.96" SSD1306 OLED (I²C, 128×32)
- RGB/RYG LEDs, buzzer, resistors, jumpers
- 5 V power supply capable of driving servos (separate supply recommended)

---

## Pin Mapping (from firmware)

| Signal                | Pin |
|-----------------------|-----|
| Ultrasonic TRIG       | 3   |
| Ultrasonic ECHO       | 4   |
| Servo – Gate 1        | 0   |
| Servo – Gate 2        | 1   |
| LED Red               | 6   |
| LED Yellow            | 7   |
| LED Green             | 8   |
| Buzzer                | 9   |
| MFRC522 SS (SDA)      | 10  |
| MFRC522 RST           | 2   |
| I²C (SSD1306)         | default Wire(0), addr `0x3C` |
| SPI (MFRC522)         | SPI(0) bus |

> Adjust pins as needed for your board; keep hardware PWM pins for servos.

---

## Software Setup

**Libraries used**
- `SPI.h`
- `Wire.h`
- `Adafruit_GFX.h`
- `Adafruit_SSD1306.h`
- `Servo.h`
- `MFRC522.h`

Install via Arduino Library Manager (or from GitHub) before compiling.

---

## How It Works

1. **Initialization**
   - Sets up SPI (RFID) and I²C (OLED).
   - Displays “Access Control System Initialized” on OLED.
   - Arms Gate 1 (proximity) and Gate 2 (RFID).

2. **Gate 1 – Proximity**
   - Measures distance from the ultrasonic sensor.
   - If `distance <= 50 cm` and gate is closed → servo to 90° (open), Green LED on.
   - If `distance > 50 cm` and gate is open → servo to 0° (close), Red LED on.

3. **Gate 2 – RFID**
   - When a card is presented, reads UID.
   - If UID matches `validUID` → open to 90°, show “Access Granted”, close after 3 s.
   - If not matched → show “Entry restricted for Outsiders”, sound buzzer 1 s.

---

## Configuration

- **Authorized UID**
  ```cpp
  // Change this to your card's 4-byte UID
  byte validUID[4] = {0x83, 0x64, 0x4B, 0x97};
