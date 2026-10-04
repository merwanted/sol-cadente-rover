<p align="center">
  <a href="README.md">🇬🇧 <b>English</b></a> | <a href="README.tr.md">🇹🇷 <b>Türkçe</b></a>
</p>

# 🪐 Sol Cadente (Ahien-14) — Autonomous Planetary Exploration Rover

<div align="center">

[![Award](https://img.shields.io/badge/Award-1st%20Place%20Winner%20%F0%9F%8F%86-ffd700?style=for-the-badge&labelColor=1a1a1a)](https://github.com/merwanted/sol-cadente-rover)
[![Event](https://img.shields.io/badge/Event-TUA%20Astro%20Hackathon%202026-0A66C2?style=for-the-badge&labelColor=1a1a1a)](https://github.com/merwanted/sol-cadente-rover)
[![Organizer](https://img.shields.io/badge/Host-Turkish%20Space%20Agency%20(TUA)-red?style=for-the-badge&labelColor=1a1a1a)](https://github.com/merwanted/sol-cadente-rover)
[![Team](https://img.shields.io/badge/Team-Sol%20Cadente-orange?style=for-the-badge&labelColor=1a1a1a)](https://github.com/merwanted/sol-cadente-rover)
[![AI](https://img.shields.io/badge/VLM-Gemma%203%2012B%20Multimodal-purple?style=for-the-badge&labelColor=1a1a1a)](https://github.com/merwanted/sol-cadente-rover)

<p align="center">
  <b>1st Place Winning Planetary Rover Prototype at the Turkish Space Agency (TUA) Astro Hackathon</b><br/>
  <i>Multimodal Vision-Language Model (VLM) route planning, ground control telemetry station, and autonomous fail-safe backtracking when RF/Wi-Fi connection is lost.</i>
</p>

</div>

---

<div align="center">
  <img src="docs/rover_prototype.jpg" alt="Sol Cadente Rover Hardware Prototype" width="48%" style="border-radius: 8px;"/>
  <img src="docs/tua_1st_award.jpg" alt="TUA 1st Place Trophy and Award" width="48%" style="border-radius: 8px;"/>
</div>

---

> ⚠️ **Project Status:** This repository contains the **functional hardware and software prototype (Competition MVP / Proof of Concept)** developed by team **Sol Cadente** during the 48-hour **Turkish Space Agency (TUA) Astro Hackathon**, tested on physically demanding terrain obstacle tracks, and evaluated live before the jury.

---

## 📌 Problem Statement & Engineering Vision

Planetary exploration rovers operating on demanding lunar and martian terrains face three mission-critical failure modes:
1. **Signal Shadowing & Link Loss:** Sudden severance of radio/Wi-Fi telemetry with the ground station due to craters, boulders, or dust conditions.
2. **Blind Obstacle Navigation:** Ultrasonic sensors alone are insufficient on rough topological surfaces; semantic visual scene understanding via optical cameras is mandatory.
3. **Path Efficiency & Return-to-Base (RTH):** Safely returning to base upon mission completion or emergency while minimizing energy expenditure and heading deviations.

**Ahien-14 (Sol Cadente)** solves all three challenges in an end-to-end integrated system combining embedded hardware, localized VLM intelligence, and autonomous reverse-odometry backtracking.

---

## 🏗️ System Architecture

```
  ┌────────────────────────────────────────────────────────────────────────┐
  │                      SOL CADENTE ROVER ARCHITECTURE                    │
  └───────────────────────────────────┬────────────────────────────────────┘
                                      │
     ┌────────────────────────────────┴────────────────────────────────┐
     ▼                                                                 ▼
【 🌍 GROUND CONTROL STATION (PC) 】                    【 🤖 ONBOARD ROVER HARDWARE 】
• Flask Web Dashboard (`dashboard.html`)                • Raspberry Pi 4 (Telemetry & Network Bridge)
• OpenCV Real-time Video Stream (640x480 MJPEG)         • Autonomous Fail-Safe Backtracking (`pi.py`)
• Multimodal AI (Gemma 3:12B via Ollama)                • Arduino Embedded Hardware Driver (`car.ino`)
• 2D Odometry Tracker (Dead-Reckoning Coordinates)      • Dual H-Bridge L298N (PWM Differential Drive)
• Mission Execution Engine (`mission.json`)             • HC-SR04 Ultrasonic Obstacle Ranging (<20cm)
• One-Click Return-to-Home (RTH)                        • DHT22 Climate Telemetry & I2C 16x2 Display
```

---

## 🚀 Key Engineering Capabilities

### 1. 🛡️ Autonomous Fail-Safe Backtracking (`pi.py`)
While in active communication with ground control, the rover records every dispatched movement vector (direction and duration, e.g., `F 150`, `R 600`) into a local circular command history buffer.
* Upon ground telemetry link loss (`WIFI LOST / Ping Timeout > 1.5s`):
* The onboard Raspberry Pi triggers an emergency state and inverts cached movement vectors (`F` ➔ `B`, `L` ➔ `R`).
* The rover **autonomously retraces its physical trajectory step-by-step in reverse** until wireless connection is re-established.

### 2. 👁️ Multimodal VLM Decision Loop (`server.py`)
During each navigation step, the ground control server feeds the live camera frame, real-time ultrasonic ranging, and the current waypoint from `mission.json` into a local **Gemma 3 (12B)** vision-language model:
* The model analyzes physical obstacles, proposing micro-steps (`F 150`) or angular evasion maneuvers (`R 400`, `L 400`) to safely approach waypoints.
* When within 20cm of an objective, it marks the waypoint complete (`NEXT`); if insurmountable obstacles block the vector, it executes a safe skip (`SKIP`).

### 3. 🗺️ Dead-Reckoning Odometry & Route Analytics
With each motor actuation, the telemetry station computes 2D Cartesian coordinates (`x`, `y`) via trigonometric heading calculations:
* Evasion maneuvers are logged as deviations.
* Total path efficiency (`path_efficiency = forward / total_commands * 100`) is calculated and streamed live to the flight dashboard.

---

## 🔌 Hardware Specifications & Pinout (`car.ino`)

| Hardware Component | Arduino Pin | Function |
| :--- | :---: | :--- |
| **HC-SR04 Ultrasonic** | Pin 2 (Trig), Pin 3 (Echo) | Real-time obstacle ranging & emergency braking |
| **DHT22 Sensor** | Pin 4 | Environmental temperature and humidity telemetry |
| **L298N Driver (Left)** | Pin 8 (IN1), Pin 9 (IN2), Pin 5 (ENA PWM) | Left drive motors forward/reverse & speed PWM |
| **L298N Driver (Right)** | Pin 10 (IN3), Pin 11 (IN4), Pin 6 (ENB PWM) | Right drive motors forward/reverse & speed PWM |
| **I2C LCD (16x2)** | Pin A4 (SDA), Pin A5 (SCL) | Onboard chassis state & telemetry screen |
| **Raspberry Pi 4** | USB Serial (`/dev/ttyUSB0`) | 9600 Baud bidirectional serial telemetry bus |

---

## 📂 Repository File Structure

```
sol-cadente-rover/
├── car.ino             # Arduino embedded firmware: motor PWM, sensors, emergency halt
├── pi.py               # Raspberry Pi telemetry daemon & Wi-Fi fail-safe backtracking engine
├── server.py           # Ground control station: Gemma 3 VLM pipeline, OpenCV stream, dead-reckoning
├── mission.json        # Waypoint & patrol mission specifications (Square Patrol, Waypoints)
├── templates/          # Telemetry web interfaces
│   ├── dashboard.html  # Live camera HUD, telemetry indicators, trajectory canvas, event logs
│   └── optimizer.html  # Mission simulation and route optimization console
├── sim/                # Heightmap terrain simulation scripts
├── docs/               # Hardware prototype and award ceremony photography
├── LICENSE             # MIT Open Source License
├── README.md           # English Documentation (Default)
└── README.tr.md        # Turkish Documentation
```

---

## 💻 Setup & Execution Guide

### 1. Arduino Firmware:
Flash `car.ino` onto the rover's Arduino Uno/Mega using the Arduino IDE.

### 2. Rover Onboard Daemon (Raspberry Pi):
```bash
# Launch serial bridge and fail-safe watchdog
python3 pi.py
```

### 3. Ground Control Station (PC):
```bash
# Install dependencies
pip install flask opencv-python requests

# Serve Gemma 3 multimodal model via Ollama
ollama run gemma3:12b

# Launch ground telemetry server
python3 server.py
```
Navigate to `http://localhost:5000` in your browser to access the flight control dashboard.

---

## 🏆 Competition Achievement & Team Credits

This project was awarded **🥇 1st Place (Winner)** at the **Astro Hackathon 2026** organized by the **Turkish Space Agency (TUA)** following physical terrain field tests and technical jury evaluations.

* **Team:** Sol Cadente
* **Team Members:** Mert Özemir, Utku Öksüz, and teammates.
* **Role of Mert Özemir:** Hardware and circuit architecture, Arduino sensor/motor firmware, telemetry integration, and autonomous fail-safe safety systems.
