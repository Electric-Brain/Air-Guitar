# 🎸 Air Guitar — ESP32 + IMU Gesture-Controlled Multi-Instrument

Turn hand motion into music. **Air Guitar** uses an ESP32 and an MPU6500 IMU to track hand orientation and movement, mapping tilt, pitch, and strum gestures to real strings, chords, and notes — rendered live on a real-time animated web dashboard with actual audio synthesis in the browser.

No physical strings. No frets. Just motion.

---

## ✨ What it does

- Tilt your hand (**Roll**) → selects which string(s) are active, widening into a full strum as you tilt further
- Move your hand up/down (**Pitch**) → switches between chord shapes (Open, C, G, D, Em, Am, Dm)
- A quick motion (**Accel + Gyro spike**) → triggers the actual "strum" and plays the notes
- 5 selectable instrument modes, each with correct string count and tuning:
  - 🎸 Acoustic
  - ⚡ Electric
  - 🎻 Classical
  - 🎸 Bass (4 strings)
  - 🪕 Ukulele (4 strings)
- A custom web dashboard (served directly from the ESP32) shows a live, animated 3D-style guitar — strings actually vibrate, glow, and react in real time as you play
- Real audio is synthesized in the browser via the Web Audio API (with per-instrument ADSR envelopes), while the ESP32's onboard buzzer plays along for physical feedback
- Adjustable sensitivity slider, live FPS/latency stats, and per-string note/frequency indicators

---

## 🧠 How it works

1. The **MPU6500** IMU streams accelerometer + gyroscope data over I2C.
2. The **ESP32** computes Roll and Pitch from the raw sensor data, uses Roll to decide which string(s) are "active," uses Pitch to select a chord shape, and watches acceleration/gyro magnitude to detect strum events.
3. Motion data, active strings, current chord, and (on a strum) the exact frequencies to play are packaged into JSON and pushed over a **WebSocket** to any connected browser.
4. The **web dashboard** (a single self-contained HTML/CSS/JS page served by the ESP32 itself — no external hosting needed) renders the selected instrument, animates the strings based on the incoming data, and plays the notes using the Web Audio API.
5. Simultaneously, the ESP32's **buzzer** plays a tone for physical/audible feedback even without a browser connected.

```
MPU6500 (I2C) → ESP32 (sensor fusion, strum/chord detection)
                    │
                    ├── WebSocket (port 81) → Browser dashboard (canvas animation + Web Audio synth)
                    ├── HTTP Server (port 80) → Serves the web dashboard
                    └── Buzzer (PWM) → Physical audio feedback
```

---

## 🔌 Hardware & Connections

| Component        | ESP32 Pin | Notes                          |
|-------------------|-----------|---------------------------------|
| MPU6500 SDA        | GPIO 21   | I2C data                        |
| MPU6500 SCL        | GPIO 22   | I2C clock                       |
| MPU6500 VCC        | 3.3V      | **Do not use 5V**               |
| MPU6500 GND        | GND       |                                  |
| Buzzer (+)         | GPIO 25   | Driven via `ledcAttach` PWM tone |
| Buzzer (-)         | GND       |                                  |

**I2C address:** `0x68` (default MPU6500 address)
**I2C clock speed:** 400 kHz (Fast Mode)

> Wearable idea: mount the ESP32 + MPU6500 + a small battery on a glove or wristband for hands-free "air strumming."

---

## 🛠️ Requirements

**Hardware**
- ESP32 dev board (any variant with WiFi)
- MPU6500 (or MPU6050-compatible) IMU module
- Passive buzzer
- Jumper wires, breadboard / perfboard, 3.3V power source

**Arduino Libraries**
- `Wire.h` (built-in)
- `WiFi.h` (built-in, ESP32 core)
- `WebServer.h` (built-in, ESP32 core)
- [`WebSocketsServer`](https://github.com/Links2004/arduinoWebSockets) by Markus Sattler
- [`FastIMU`](https://github.com/LiquidCGS/FastIMU) for MPU6500 sensor handling

---

## 🚀 Setup & Usage

1. **Wire up** the MPU6500 and buzzer as per the table above.
2. **Install libraries** listed above via the Arduino Library Manager (or PlatformIO).
3. Open the `.ino` file and update your WiFi credentials:
   ```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   ```
4. **Flash** the sketch to your ESP32.
5. Open the Serial Monitor (115200 baud) — once connected, it will print the ESP32's local IP address:
   ```
   ✓ Connected! Open: http://<esp32-ip-address>
   ```
6. Open that IP address in a browser on the **same WiFi network**.
7. Tap **"TAP TO ENABLE SOUND"** (required by browsers to unlock audio playback).
8. Pick an instrument mode, tilt your hand to select strings, and move quickly to strum. 🎶

---

## 🎛️ Web Dashboard Features

- Live WebSocket connection status, audio status, FPS, and latency indicators
- Instrument selector (Acoustic / Electric / Classical / Bass / Ukulele)
- Animated canvas guitar per instrument, with strings that vibrate and glow when struck
- Real-time roll/pitch/strum-intensity motion bars
- Strum count, note count, active string count, and live acceleration (G) stats
- Sensitivity slider — synced back to the ESP32 over WebSocket to tune strum threshold sensitivity live

---

## 🔧 Tuning & Customization

- **Tunings**: defined per instrument in the `tunings[][]` array (firmware) and `guitarTypes[]` array (webpage) — edit these to try alternate tunings (Drop D, DADGAD, etc.)
- **Chord shapes**: defined in `chordFrets[][]` — add more chords by extending this array and the `detectChord()` logic
- **Strum sensitivity**: `STRUM_THRESHOLD`, `GYRO_THRESHOLD`, and `STRUM_COOLDOWN` constants control how strums are detected
- **String selection range**: `updateStringSelection()` maps roll angle to string index — adjust `rollRange` for a more/less sensitive tilt response

---

## 💡 Ideas for Future Improvements

- Add a second IMU axis (yaw via magnetometer) for richer gesture mapping
- Onboard SD card / flash recording of "performances"
- Bluetooth MIDI output to control DAWs directly
- Battery + enclosure for a fully wearable, wireless build
- Additional instrument voices (violin, synth lead, etc.)

---

## 📜 License

Feel free to fork, build on, and remix this project. Consider adding a license file (MIT is a good default for hardware/firmware projects like this) if you plan to accept contributions.

---

Built as an experimental project exploring embedded sensor fusion, real-time WebSocket communication, and browser-based audio synthesis — all running on a single ESP32. 🎸
