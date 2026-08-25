````markdown
<div align="center">

# 🔐 ESP32-CAM Smart Door Lock System

### AI Thinker ESP32-CAM | Live Video Streaming | Smart Web Dashboard | Servo Lock Control

<p align="center">

![ESP32](https://img.shields.io/badge/ESP32-CAM-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=Arduino&logoColor=white)
![FreeRTOS](https://img.shields.io/badge/FreeRTOS-Real_Time_OS-success?style=for-the-badge)
![WiFi](https://img.shields.io/badge/WiFi-Access_Point-blue?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-orange?style=for-the-badge)

</p>

### 🚪 A Professional IoT Smart Door Lock with Live Camera Streaming and Web-Based Remote Control

</div>

---

# 📸 Project Preview

> **Replace these with your own screenshots**

<p align="center">

| Web Dashboard | Live Camera |
|:-------------:|:-----------:|
| ![](images/dashboard.png) | ![](images/live_stream.png) |

</p>

---

# 📖 Overview

This project is a **Professional Smart Door Lock System** built using the **AI Thinker ESP32-CAM**.

Unlike traditional smart lock projects, this system creates its **own Wi-Fi Access Point**, allowing any nearby phone or laptop to connect **without requiring an internet connection or router**. Once connected, users can securely access a **modern responsive web dashboard** to:

- 🔓 Unlock the Door
- 🔒 Lock the Door
- 📹 Watch Live Camera Feed
- 📸 Capture Snapshots
- 💡 Toggle Flash Light
- 📋 Monitor Activity Log
- ⏱ Auto Re-Lock the Door

The project combines **Embedded Systems**, **IoT**, **ESP32**, **FreeRTOS**, **Web Development**, and **Servo Control** into a single real-world application.

---

# ✨ Features

## 🔐 Smart Door Lock

- Servo Controlled Lock
- Remote Lock / Unlock
- Automatic Re-Lock Timer
- Door Status Indicator

---

## 📹 Live Video Streaming

- MJPEG Video Streaming
- VGA Resolution
- Mobile Friendly
- Low Latency
- Separate Streaming Server

---

## 🌐 Professional Web Dashboard

- Beautiful Responsive UI
- Works on Mobile & Desktop
- Real-Time Status Updates
- Modern Design
- Activity Log
- Snapshot Button

---

## 📡 Networking

- Works without Internet
- ESP32-CAM creates its own Wi-Fi Hotspot
- No Router Required
- WPA2 Protected Access Point

---

## 💡 Additional Features

- Flash LED Control
- Camera Snapshot
- Event Logging
- Status LEDs
- FreeRTOS Background Streaming Task
- Auto Camera Initialization

---

# 🏗 System Architecture

```
              Smartphone / Laptop
                      │
             Connect via Wi-Fi
                      │
        ESP32-CAM Access Point (AP)
                      │
      ┌───────────────┼────────────────┐
      │               │                │
      │               │                │
 Live Camera      Servo Lock      Flash LED
      │               │                │
      └───────────────┼────────────────┘
                      │
          Professional Web Dashboard
```

---

# 🛠 Hardware Required

| Component | Quantity |
|-----------|:--------:|
| ESP32-CAM (AI Thinker) | 1 |
| SG90 Servo Motor | 1 |
| FTDI Programmer | 1 |
| Jumper Wires | As Required |
| 5V Power Supply | 1 |

---

# 🔌 Wiring

| ESP32-CAM | Component |
|------------|-----------|
| GPIO2 | Servo Signal |
| GPIO4 | Flash LED |
| GPIO33 | Status LED |
| 5V | Servo VCC |
| GND | Servo GND |

---

# 🌐 Default Wi-Fi Configuration

| Parameter | Value |
|-----------|-------|
| SSID | SmartDoorLock |
| Password | 12345678 |
| Dashboard | http://192.168.4.1 |
| Stream Port | 81 |

These are defined directly in the firmware configuration.

---

# 📂 Project Structure

```
ESP32_CAM_Smart_Door_Lock
│
├── SmartDoorLock.ino
├── images/
│   ├── dashboard.png
│   ├── live_stream.png
│   └── wiring.png
├── README.md
└── LICENSE
```

---

# 🚀 Getting Started

## 1 Clone Repository

```bash
git clone https://github.com/YourUsername/ESP32-CAM-Smart-Door-Lock.git
```

---

## 2 Open Arduino IDE

Install the latest **ESP32 Board Package**

---

## 3 Select Board

```
AI Thinker ESP32-CAM
```

---

## 4 Upload

- Connect GPIO0 to GND
- Press RESET
- Upload Sketch
- Disconnect GPIO0
- Press RESET Again

---

## 5 Connect

Phone →

```
Wi-Fi

SSID :
SmartDoorLock

Password :
12345678
```

---

## 6 Open Browser

```
http://192.168.4.1
```

You'll see the Smart Door Lock Dashboard.

---

# 📱 Dashboard Features

✅ Live Camera Feed

✅ Unlock Door

✅ Lock Door

✅ Flash Light Control

✅ Snapshot Capture

✅ Activity Log

✅ Connected Clients

✅ Uptime Counter

✅ Auto Re-Lock Timer

The dashboard includes responsive controls, status badges, activity logging, and a live MJPEG stream.

---

# 📡 HTTP API

| Endpoint | Description |
|-----------|-------------|
| / | Dashboard |
| /status | Device Status |
| /open | Unlock Door |
| /lock | Lock Door |
| /flash | Toggle Flash |
| /snapshot | Capture Image |
| :81/stream | Live Camera Feed |

These endpoints are registered by the embedded web server.

---

# 📚 Concepts Covered

- ESP32-CAM
- FreeRTOS
- HTTP Server
- MJPEG Streaming
- Servo Motor Control
- Wi-Fi Access Point
- JSON APIs
- HTML
- CSS
- JavaScript
- Embedded C++
- Camera Driver
- Event Logging
- PWM
- LEDC Timer
- Non-Blocking Programming

---

# 🎯 Learning Outcomes

After completing this project you will learn:

- ESP32-CAM Programming
- Camera Configuration
- Live Video Streaming
- Embedded Web Servers
- Building REST APIs
- Servo Motor Control
- FreeRTOS Task Creation
- Embedded UI Design
- Wi-Fi AP Mode
- IoT Dashboard Development

---

# 🚀 Future Improvements

- Face Recognition
- RFID Authentication
- Fingerprint Sensor
- Firebase Integration
- Telegram Alerts
- Cloud Storage
- Mobile Application
- Voice Commands
- MQTT Support
- Motion Detection
- AI Person Detection

---

# ⭐ Support

If you found this project helpful,

please consider giving this repository a ⭐ **Star**.

It motivates me to create more professional **Embedded Systems**, **IoT**, **ESP32**, **STM32**, and **FreeRTOS** projects.

---

### GitHub

https://github.com/Surya-8948

---

# 📜 License

This project is released under the **MIT License**.

Feel free to use, modify, and distribute it for educational and personal projects.

---

<div align="center">

# 🌟 Star this Repository if you Like It 🌟

### Made with ❤️ by Surya Bajpai

**Happy Coding 🚀**

</div>
````
