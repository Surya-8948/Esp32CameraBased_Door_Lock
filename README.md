
<div align="center">

# 🔐 ESP32-CAM Smart Door Lock System

### Professional IoT Smart Door Lock with Live Camera Streaming & Responsive Web Dashboard

Remote Door Control • Live Video Streaming • ESP32-CAM • FreeRTOS • Servo Lock

<p align="center">

![ESP32](https://img.shields.io/badge/ESP32-CAM-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=Arduino&logoColor=white)
![FreeRTOS](https://img.shields.io/badge/FreeRTOS-Real--Time_OS-success?style=for-the-badge)
![WiFi](https://img.shields.io/badge/WiFi-Access_Point-blue?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-orange?style=for-the-badge)

</p>

*A professional Embedded IoT project demonstrating how to build a **Wi-Fi based Smart Door Lock System** using **ESP32-CAM**, featuring **Live Video Streaming**, **Remote Servo Control**, and a **Modern Responsive Web Dashboard**.*

</div>

---

# 📸 Project Demonstration


<p align="center">



<img src="https://github.com/Surya-8948/Esp32CameraBased_Door_Lock/blob/main/Esp32DoorlockDashboard.png?raw=true" width="850">

</p>

---

# 📖 Overview

This project demonstrates a **Professional Smart Door Lock System** using the **AI Thinker ESP32-CAM**.

Unlike conventional smart locks that depend on cloud services or external routers, this system creates its **own Wi-Fi Access Point**, allowing smartphones or laptops to connect directly without an internet connection.

Once connected, users can access a **beautiful responsive web dashboard** to monitor the live camera stream, unlock or lock the door remotely, control the flash LED, capture snapshots, and monitor real-time activity logs.

The project combines multiple Embedded Systems concepts including:

- ESP32-CAM
- Servo Motor Control
- Embedded Web Server
- FreeRTOS
- MJPEG Streaming
- Responsive Web UI
- Wi-Fi Access Point Mode
- JSON APIs

---

# ✨ Features

✔ ESP32-CAM Live MJPEG Video Streaming

✔ Modern Responsive Web Dashboard

✔ Remote Door Lock / Unlock

✔ Servo Controlled Smart Lock

✔ Built-in Wi-Fi Hotspot

✔ No Internet Required

✔ Flash Light Control

✔ Snapshot Capture

✔ Live Activity Log

✔ Connected Client Counter

✔ Device Uptime

✔ Auto Re-Lock Timer

✔ FreeRTOS Background Streaming Task

✔ Mobile Friendly Interface

✔ Clean Modular Source Code

---

# 🌐 Web Dashboard

The project includes a professional **HTML + CSS + JavaScript** dashboard hosted directly on the ESP32-CAM.

### Dashboard Features

- 📹 Live Camera Feed
- 🔓 Open Gate
- 🔒 Keep Locked
- 💡 Flash Light ON/OFF
- 📸 Capture Snapshot
- 📊 Live Door Status
- 📋 Activity Log
- 👥 Connected Clients
- ⏱ Device Uptime
- ⌛ Auto Re-Lock Countdown

---

# 🛠 Hardware Required

| Component | Quantity |
|-----------|:--------:|
| ESP32-CAM (AI Thinker) | 1 |
| SG90 Servo Motor | 1 |
| FTDI USB Programmer | 1 |
| Jumper Wires | As Required |
| 5V Power Supply | 1 |

---

# 🔌 Circuit Connections

| ESP32-CAM Pin | Component |
|---------------|-----------|
| GPIO2 | Servo Signal |
| GPIO4 | Flash LED |
| GPIO33 | Status LED |
| 5V | Servo VCC |
| GND | Common Ground |

---

# 📂 Project Structure

```text
ESP32_CAM_Smart_Door_Lock
│
├── SmartDoorLock.ino
├── images
│   ├── dashboard.png
│   ├── live_stream.png
│   ├── circuit.png
│   └── demo.gif
├── README.md
└── LICENSE
```

---

# 🌐 Default Wi-Fi Settings

| Parameter | Value |
|-----------|-------|
| SSID | SmartDoorLock |
| Password | 12345678 |
| Dashboard | http://192.168.4.1 |
| Stream | http://192.168.4.1:81/stream |

---

# 🚀 Getting Started

### Clone Repository

```bash
git clone https://github.com/Surya-8948/ESP32-CAM-SmartDoorLock.git
```

### Open Arduino IDE

Open

```
SmartDoorLock.ino
```

### Install ESP32 Board Package

```
Boards Manager
↓

ESP32 by Espressif Systems
```

### Select Board

```
AI Thinker ESP32-CAM
```

### Upload

1. Connect GPIO0 to GND
2. Press RESET
3. Upload Code
4. Disconnect GPIO0
5. Press RESET Again

---

# 📱 Using the Smart Door Lock

Connect your phone to:

```
SSID

SmartDoorLock
```

Password

```
12345678
```

Open Browser

```
http://192.168.4.1
```

Enjoy the Smart Dashboard.

---

# 📡 API Endpoints

| Endpoint | Description |
|-----------|-------------|
| `/` | Dashboard |
| `/status` | Device Status |
| `/open` | Unlock Door |
| `/lock` | Lock Door |
| `/flash` | Toggle Flash |
| `/snapshot` | Capture Image |
| `:81/stream` | Live Camera Stream |

---

# 📚 Concepts Covered

- ESP32-CAM Programming
- Wi-Fi Access Point
- Embedded HTTP Server
- MJPEG Video Streaming
- Servo PWM Control
- JSON Communication
- HTML
- CSS
- JavaScript
- Responsive UI
- FreeRTOS Tasks
- Embedded C++
- Event Logging
- Camera Driver
- Non-Blocking Programming

---

# 🎯 Learning Outcomes

After completing this project you will learn:

- ESP32-CAM Programming
- Camera Streaming
- Building Embedded Web Servers
- Servo Motor Control
- FreeRTOS Task Management
- HTML Dashboard Design
- JavaScript Fetch API
- REST API Development
- Embedded IoT Application Design
- Wi-Fi Networking

---

# 📈 Future Improvements

- 😊 Face Recognition
- 👤 Face Detection
- 🔑 RFID Authentication
- 👆 Fingerprint Module
- ☁ Firebase Integration
- 📲 Telegram Notifications
- 📱 Android Application
- 🎤 Voice Commands
- ☁ Cloud Monitoring
- 📦 MQTT Support
- 🤖 AI Person Detection

---

# 💡 Applications

- Smart Home Security
- Office Access Control
- Hostel Room Lock
- College Laboratory Access
- Warehouse Security
- Industrial Gate Monitoring
- Remote Surveillance
- IoT Security Systems

---

# ⭐ Support

If you found this project useful,

please consider giving it a ⭐ **Star**.

Your support motivates me to create more professional **Embedded Systems**, **IoT**, **ESP32**, **STM32**, **FreeRTOS**, and **Robotics** projects.

---
### GitHub

https://github.com/Surya-8948

---

# 📜 License

This project is released under the **MIT License**.

Feel free to use, modify, and distribute it for educational and personal projects.

---

<div align="center">

## 🌟 Don't forget to Star this Repository 🌟

### Made with ❤️ by **Surya Bajpai**

**Happy Coding 🚀**

</div>
````
