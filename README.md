<div align="center">
<img src="📚 docs/ assets/layout.jpg" alt="Women Auto Safety Bracelet System" width="100%">

# 🛡️ Women Auto Safety Bracelet System

### *Zero-click personal safety — from wrist tap to cloud evidence in under 3 seconds*

[![Build](https://img.shields.io/badge/Build-Passing-10B981?style=flat-square&logo=android)](.)
[![Firebase](https://img.shields.io/badge/Firebase-Connected-FF6F00?style=flat-square&logo=firebase)](https://firebase.google.com)
[![AWS S3](https://img.shields.io/badge/AWS%20S3-eu--north--1-FF9900?style=flat-square&logo=amazons3)](https://aws.amazon.com/s3)
[![Platform](https://img.shields.io/badge/Platform-Android%20%7C%20ESP32-3DDC84?style=flat-square&logo=android)](.)
[![License](https://img.shields.io/badge/License-MIT-6B21A8?style=flat-square)](LICENSE)
[![Stars](https://img.shields.io/github/stars/yourusername/women-auto-safety?style=flat-square&color=BE185D)](.)

<br/>

> **The problem with every existing safety app:** they need you to take action at the exact moment you can't.  
> **Our solution:** triple-tap your bracelet — the system does everything else automatically.

<br/>

[📱 Android App](#-android-app) • [⌚ ESP32 Firmware](#-esp32-wearable-firmware) • [☁️ Cloud Setup](#️-cloud-infrastructure) • [🤖 Node-RED](#-node-red-automation) • [🚀 Quick Start](#-quick-start)

</div>

---

## 📌 Table of Contents

- [What This Is](#-what-this-is)
- [How It Works](#-how-it-works)
- [System Architecture](#-system-architecture)
- [Repository Structure](#-repository-structure)
- [Component Details](#-component-details)
  - [Android App](#-android-app)
  - [ESP32 Wearable Firmware](#-esp32-wearable-firmware)
  - [Cloud Infrastructure](#️-cloud-infrastructure)
  - [Node-RED Automation](#-node-red-automation)
  - [Disha Smart Monitor](#-disha-smart-monitor-police-dashboard)
- [Quick Start](#-quick-start)
- [Hardware](#-hardware)
- [Firebase Schema](#-firebase-database-schema)
- [What Makes This Different](#-what-makes-this-different)
- [Roadmap](#-roadmap)
- [Team](#-team)
- [License](#-license)

---

## 🔍 What This Is

A **fully automatic personal safety system** built around a wearable wrist bracelet. When a threat is detected via a **triple-tap gesture**, the system:

- 🎤 Records **10-second audio clips** continuously from the phone microphone
- 📸 Captures **front + rear camera photos** every 30 seconds
- 📍 Pushes **live GPS coordinates** to the cloud every 5 minutes
- ☁️ Uploads everything to **AWS S3** in real time
- 📡 Saves all media URLs to **Firebase Realtime Database**
- 📲 Sends **Telegram alerts** with live location pin, photo, and playable audio to emergency contacts
- 🖥️ Streams live evidence to the **Disha Smart Monitor** police dashboard

**Zero phone interaction required after the initial tap.**

---

## ⚙️ How It Works

```
👆 Triple tap on bracelet
         │
         ▼
   ┌─────────────┐
   │   ESP32     │  ── writes threat_detect/state = true ──►  Firebase DB
   │  (MPU6050)  │
   └─────────────┘
                                                                    │
         ┌──────────────────────────────────────────────────────────┘
         │  Android app watching Firebase detects state = true
         ▼
   ┌─────────────────────────────────────────────────┐
   │         SafetyForegroundService.kt              │
   │                                                 │
   │  🎤 Audio loop   ──► every 10s                 │
   │  📸 Front photo  ──► every 30s                 │
   │  📸 Back photo   ──► 15s after front           │
   │  📍 GPS          ──► every 5 min               │
   └──────────────┬──────────────────────────────────┘
                  │
                  ▼
          ┌──────────────┐        ┌──────────────────────────────┐
          │   AWS S3     │        │       Firebase DB             │
          │  women-care  │ ──────►│  latest_audio URL            │
          │  eu-north-1  │        │  latest_photo URL            │
          └──────────────┘        │  live_location lat/lng       │
                                  └──────────┬───────────────────┘
                                             │
                          ┌──────────────────┴──────────────────┐
                          │                                     │
                          ▼                                     ▼
                   ┌─────────────┐                   ┌──────────────────┐
                   │  Node-RED   │                   │  Disha Smart     │
                   │  (local PC) │                   │  Monitor Website │
                   └──────┬──────┘                   │  (Police View)   │
                          │                          └──────────────────┘
              ┌───────────┼───────────┐
              ▼           ▼           ▼
          📝 Text     📍 Location  📸 Photo + 🎤 Audio
              └───────────┴───────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  Telegram   │
                   │  Emergency  │
                   │   Group     │
                   └─────────────┘
```

---

## 🗂️ Repository Structure

```
women-auto-safety/
│
├── 📱 android/
│   └── app/src/main/
│       ├── kotlin/com/example/womencare/
│       │   ├── MainActivity.kt              # UI only — Jetpack Compose dashboard
│       │   └── SafetyForegroundService.kt   # All logic — runs when screen is off
│       ├── AndroidManifest.xml
│       └── res/
│
├── ⌚ esp32/
│   └── women_safety_bracelet/
│       └── women_safety_bracelet.ino        # WiFi Manager + tap detection + Firebase
│
├── 🤖 node-red/
│   └── safety_bot_flow.json                 # Full Node-RED flow — import directly
│
├── 🖥️ disha-dashboard/
│   └── index.html                           # Police monitoring dashboard (single file)
│
├── ☁️ firebase/
│   └── database.rules.json                  # Firebase Realtime DB security rules
│
├── 📚 docs/
│   ├── assets/
│   │   ├── layout.jpg                 # System architecture diagram
│   │   └── team.jpeg
                              
│
├── README.md
└── LICENSE
```

---

## 📦 Component Details

### 📱 Android App

**Path:** `android/`  
**Language:** Kotlin  
**Min SDK:** Android 10 (API 29)  
**Tested on:** Xiaomi Redmi Note 8 Pro (MIUI 12)

| File | Responsibility |
|------|---------------|
| `MainActivity.kt` | UI dashboard only — Jetpack Compose, reads from AppState |
| `SafetyForegroundService.kt` | All logic — Firebase listener, camera, audio, GPS, S3 upload |

**Key features:**
- Runs as Android **Foreground Service** with `START_STICKY` — survives screen-off and app-kill
- `onTaskRemoved()` auto-restarts if swiped away
- **Three independent timer loops** — audio (10s), photos (30s), GPS (5min)
- **Camera2 API** with separate `HandlerThread` per camera + `AtomicBoolean` mutex — no shared buffer conflicts
- Filters Xiaomi virtual camera IDs (20, 21, 22, 61) — uses only physical cameras (0, 1)
- **AWS TransferUtility** for background-safe multipart S3 uploads

**Dependencies** (`app/build.gradle`):
```kotlin
implementation("com.amazonaws:aws-android-sdk-s3:2.75.0")
implementation("com.amazonaws:aws-android-sdk-core:2.75.0")
implementation("com.google.android.gms:play-services-location:21.3.0")
implementation("com.google.firebase:firebase-database-ktx")
implementation("androidx.activity:activity-compose:1.9.0")
```

**Recording schedule during active threat:**
```
Audio  ──── new 10s clip every 10 seconds ────────────────────────►
Photos ──── front@0s ── 15s ── back@15s ── 15s ── front@30s ──────►
GPS    ──── push@0s ─────────────────────── push@5min ─────────────►
```

---

### ⌚ ESP32 Wearable Firmware

**Path:** `esp32/women_safety_bracelet/`  
**Board:** ESP32 Dev Module  
**Framework:** Arduino (ESP-IDF via Arduino Core)

**Pin assignments:**
| Pin | Component | Function |
|-----|-----------|----------|
| `10` | Vibration Sensor (MPU6050) | Tap detection input |
| `8` | LED | Status indicator |

**WiFi Manager — no re-flashing needed at new locations:**

```
Boot
 │
 ├─ Hold sensor 3s? ──► Clear saved WiFi ──► Setup mode
 │
 ├─ Saved WiFi found? ──► Try connecting (10s timeout)
 │       ├─ Success ──► Normal operation ✅
 │       └─ Fail ──────► Setup mode
 │
 └─ No saved WiFi ──────► Setup mode
         │
         ▼
  Hotspot: "WomenSafety-Setup"
  Password: safety123
  Open: http://192.168.4.1
         │
         ▼
  Pick WiFi from scan list + enter password
         │
         ├─ Connects ──► Save to flash ──► Normal operation ✅
         └─ Fails ────► Back to hotspot (automatic)
```

**Tap detection logic:**
- 3 taps within a 1-second window = valid trigger
- 150ms debounce between taps
- Any other count (1, 2, 4+) = ignored

---

### ☁️ Cloud Infrastructure

#### Firebase Realtime Database
| Property | Value |
|----------|-------|
| Project | `women-safety-d12d5` |
| URL | `https://women-safety-d12d5-default-rtdb.firebaseio.com/` |
| Region | `us-central1` |

#### AWS S3
| Property | Value |
|----------|-------|
| Bucket | `women-care` |
| Region | `eu-north-1` (Stockholm) |
| Audio path | `recordings/audio_<timestamp>.mp3` |
| Photo path | `photos/front_<timestamp>.jpg` |
| Public URL | `https://women-care.s3.eu-north-1.amazonaws.com/<path>` |

---

### 🤖 Node-RED Automation
<div align="center">
<img src="🤖 node-red/node red.png" alt="node red" width="100%">

**Path:** `node-red/safety_bot_flow.json`  
**Runtime:** Node-RED (local PC or cloud VM)  
**Required package:** `node-red-contrib-telegrambot`

**Import the flow:**
```
Node-RED menu → Import → Paste contents of safety_bot_flow.json → Deploy
```

**The flow runs 3 independent polling loops:**

| Loop | Interval | Action |
|------|----------|--------|
| Location | every 5 min | Sends live Google Maps pin to Telegram |
| Photo | every 15s | Sends latest S3 photo to Telegram |
| Audio | every 10s | Downloads S3 MP3 → sends as playable audio |

**What the Emergency Telegram group receives on threat:**
1. 📝 Text alert with GPS coordinates, Maps link, session duration
2. 📍 Native location pin (one tap → opens in Google Maps)
3. 📸 Latest photo displayed inline
4. 🎤 10-second audio clip — playable directly in Telegram

---

### 🖥️ Disha Smart Monitor (Police Dashboard)
<div align="center">
<img src="🖥️ disha-dashboard/preview.jpeg" alt="disha" width="100%">
**Path:** `disha-dashboard/index.html`  
**Type:** Single HTML file — no server, no framework, no install  
**Usage:** Open directly in Chrome/Edge in police control room or patrol tablet

**Features:**
- 🚨 Live threat status banner (green/red, auto-updates)
- 🗺️ Embedded Google Maps iframe — updates on new GPS fix
- 📸 Latest photo + full scrollable photo timeline with lightbox
- 🎤 HTML5 audio player for latest recording + archive list
- 📊 Live stats — total clips, total photos, session duration, last upload
- 📋 Export Evidence button — compiles all URLs into a timestamped text report
- 🔔 Browser notification on threat activation
- Polls Firebase every 5 seconds — no page reload needed

---

## 🚀 Quick Start

### 1. Clone the repo
```bash
git clone https://github.com/yourusername/women-auto-safety.git
cd women-auto-safety
```

### 2. Set up Firebase
```bash
# Create a Firebase project at console.firebase.google.com
# Enable Realtime Database
# Download google-services.json → place in android/app/
```

### 3. Flash the ESP32
```
1. Open esp32/women_safety_bracelet/women_safety_bracelet.ino in Arduino IDE
2. Install boards: ESP32 by Espressif (Board Manager)
3. Select board: ESP32 Dev Module
4. Upload
5. On first boot → connect to "WomenSafety-Setup" hotspot → go to 192.168.4.1
6. Enter your WiFi credentials → Save
```

### 4. Build & install the Android app
```bash
cd android
# Open in Android Studio
# Add your google-services.json to app/
# Update AWS credentials in SafetyForegroundService.kt
# Build → Run on device
# Grant: Camera, Microphone, Location permissions
# Enable Autostart in phone settings (critical for Xiaomi/MIUI)
```

### 5. Import Node-RED flow
```bash
# Install Node-RED: npm install -g node-red
# Install telegram package: npm install node-red-contrib-telegrambot
# node-red
# Open http://127.0.0.1:1880
# Menu → Import → paste node-red/safety_bot_flow.json
# Configure your Telegram bot token
# Deploy
```

### 6. Open the police dashboard
```bash
# Simply open disha-dashboard/index.html in Chrome
# No server needed — reads Firebase directly
```

---

## 🔧 Hardware

| Component | Function | Approx. Cost |
|-----------|----------|-------------|
| ESP32 Dev Board | Main MCU — WiFi + processing | ₹350 |
| MPU6050 | 6-axis IMU — tap detection | ₹80 |
| 3.7V LiPo Battery (1000mAh) | Power supply | ₹200 |
| TP4056 Module | USB-C charging circuit | ₹30 |
| MT3608 Boost Converter | 3.7V → 5V for ESP32 | ₹25 |
| Vibration Motor | Haptic confirmation on tap | ₹20 |
| LED (8mm red) | Status indicator | ₹5 |
| **Total** | | **~₹710** |

**Wiring diagram:** see [`docs/HARDWARE.md`](docs/HARDWARE.md)

---

## 🔥 Firebase Database Schema

```
threat_detect/
├── state                    ← Boolean  — ESP32 writes true on threat
├── session_start            ← String   — ISO timestamp
├── seconds_active           ← Integer  — live counter
├── last_upload_time         ← String   — timestamp of last S3 upload
├── latest_audio             ← String   — S3 URL of most recent audio
├── latest_photo             ← String   — S3 URL of most recent photo
├── live_location/
│   ├── lat                  ← Float
│   ├── lng                  ← Float
│   ├── accuracy             ← Float    — metres
│   ├── timestamp            ← String
│   └── maps_link            ← String   — Google Maps URL
└── uploads/
    ├── audio/
    │   └── {timestamp}      ← String   — S3 URL (full archive)
    └── photos/
        └── {timestamp}      ← String   — S3 URL (full archive)
```

---

## 🏆 What Makes This Different

| Feature | Athena | Leaf SAFER PRO | Research Papers | **This Project** |
|---------|--------|---------------|-----------------|-----------------|
| No phone interaction needed | ❌ | ❌ | ❌ | ✅ |
| Automatic audio recording | ❌ | ✅ | partial | ✅ |
| Automatic photo evidence | ❌ | ❌ | ❌ | ✅ |
| Cloud archive (S3) | ❌ | ❌ | ❌ | ✅ |
| Real-time Telegram with media | ❌ | ❌ | ❌ | ✅ |
| Police monitoring dashboard | ❌ | ❌ | ❌ | ✅ |
| Works with screen off | ❌ | ✅ | — | ✅ |
| Open source | ❌ | ❌ | — | ✅ |

---

## 🗺️ Roadmap

- [x] ESP32 WiFi Manager (captive portal, no re-flashing)
- [x] Android Foreground Service (survives MIUI background kill)
- [x] Dual camera capture with race condition fix
- [x] AWS S3 upload pipeline (audio + photos)
- [x] Firebase Realtime Database integration
- [x] Node-RED → Telegram alerts (text + location + photo + audio)
- [x] Disha Smart Monitor dashboard design
- [ ] SMD PCB design — ESP32-S3 + IMU + charging IC on one board
- [ ] ML-based automatic threat detection (no tap needed)
- [ ] 4G SIM module (SIM7600) — independent of phone WiFi
- [ ] End-to-end encryption for all S3 media
- [ ] OTA firmware updates via Firebase
- [ ] BIS certification (India market)
- [ ] Provisional patent filing
- [ ] Campus pilot program (50 users)

---

## 👥 Team
<div align="center">
<img src="📚 docs/ assets/team.jpeg" alt="Women Auto Safety Bracelet System" width="100%">

| Name | Role | linked in|
|------|------|----------|
| **Abhishek** | Hardware (ESP32), Android (Kotlin), Cloud (Firebase + AWS) |https://www.linkedin.com/in/abhishek-embaddedelectro/
| **deekshi shetti** | Cloud (Firebase + AWS) |

*Built for Innovation Hub Future Founders 2.0*

---

## 📄 License

```
MIT License — Copyright (c) 2026 Women Auto Safety Team

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software to deal in the Software without restriction, including the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
sell copies of the Software.
```

---

<div align="center">

**Built with ❤️ to make every woman safer**

*If this project helped you or inspired you — leave a ⭐ and share it*

[![GitHub stars](https://img.shields.io/github/stars/yourusername/women-auto-safety?style=social)](.)

</div>
