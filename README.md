# Smart Door Security System — ESP32-C6

A dual-mode security system using two ESP32-C6 microcontrollers communicating over ESP-NOW. In **Armed mode**, door intrusions trigger a PWM piezo siren, NeoPixel LED alerts, and real-time Telegram push notifications. In **Disarmed mode**, the system passively tracks foot traffic with a running tally on a 16×2 LCD. Built for Georgia Tech's ECE 4180 (Embedded Systems Design).

## Demo

**[Watch the demo on YouTube →](https://youtu.be/YOUR_VIDEO_ID)**

## Features

- **Dual operating modes** — Armed (security alarm) and Disarmed (foot-traffic counter), toggled via a potentiometer combination lock + button press
- **Real-time phone alerts** — HTTP push notifications via Telegram Bot API over Wi-Fi when an intrusion is detected
- **Audible alarm** — PWM-driven piezo buzzer producing a wailing siren tone pattern
- **Visual alarm** — NeoPixel strip flashing red/blue police-light pattern in alarm; solid red when armed; green when disarmed
- **Door sensing** — Reed switch (magnetic contact sensor) detects door open/close events via hardware interrupt
- **Persistent state** — Armed/disarmed mode and foot-traffic count survive power cycles via ESP32 NVS (non-volatile storage)
- **Wireless architecture** — Two ESP32-C6 nodes communicate over ESP-NOW with no wired data link required

## System Architecture

### Sensor Node (ESP32-C6 #1 — mounted at door)

The Sensor Node is responsible for detecting door events and transmitting alerts to the Base Station.

| Component | Role |
|---|---|
| Reed switch + magnet | Detects door open/close via GPIO interrupt (magnet on door, switch on frame) |
| ESP-NOW TX | Sends event packets (door open, door close) wirelessly to the Base Station |

### Base Station (ESP32-C6 #2 — indoors, wall-powered)

The Base Station handles all user interaction, alarm output, and phone notifications.

| Component | Interface | Role |
|---|---|---|
| 16×2 LCD + PCF8574N | I2C | Displays current mode, foot-traffic count, and last event |
| NeoPixel strip (WS2812B) | GPIO | Visual status — green (disarmed), solid red (armed), flashing red/blue (alarm) |
| Passive piezo buzzer | PWM | Audible siren with rising/falling tone pattern during alarm |
| Potentiometer (50kΩ) | ADC | Combination lock input — user dials to a target value to authorize arm/disarm |
| Push button | GPIO (ISR) | Confirms arm/disarm after correct potentiometer position |
| ESP-NOW RX | Wi-Fi radio | Receives door event packets from the Sensor Node |
| Wi-Fi (STA mode) | HTTP | Sends Telegram Bot API push notifications on intrusion |

### Communication Flow

```
┌─────────────┐     ESP-NOW      ┌──────────────┐    HTTP/Wi-Fi    ┌──────────┐
│ Sensor Node │ ──────────────► │ Base Station │ ──────────────► │ Telegram │
│  Reed Switch│                  │  LCD + Siren │                  │  (Phone) │
│  (at door)  │                  │  NeoPixels   │                  │          │
└─────────────┘                  └──────────────┘                  └──────────┘
```

## How It Works

### Armed Mode

1. Reed switch ISR fires on the Sensor Node when the door opens
2. Sensor Node sends an ESP-NOW packet to the Base Station
3. Base Station receives the alert and simultaneously:
   - Drives the piezo buzzer with a PWM wailing siren pattern
   - Flashes the NeoPixel strip in a red/blue alternating pattern
   - Sends an HTTP POST to the Telegram Bot API with an intrusion alert
   - Updates the LCD with "ALARM — DOOR OPENED"
4. User disarms by dialing the potentiometer to the correct value + pressing the button

### Disarmed Mode

1. Reed switch fires on each door open event
2. Sensor Node sends an ESP-NOW packet to the Base Station
3. Base Station increments the foot-traffic counter
4. LCD updates with the new running total (e.g., "Visitors: 48")
5. NeoPixels stay green — no alarm triggered

### Arm/Disarm Control

The potentiometer acts as a physical combination lock. The user dials to a specific analog value (read via ADC), then presses the button to confirm. If the value is within the target range, the mode toggles. This provides basic access control without needing a phone or app.

## Software Design

The Base Station firmware uses **FreeRTOS** for concurrent task management:

| Task / Thread | Responsibility |
|---|---|
| **Sensor Monitor** | Handles incoming ESP-NOW packets and reed switch event processing |
| **Alarm Control** | Manages buzzer PWM patterns and NeoPixel animations |
| **Display Update** | Refreshes the LCD with mode, count, and event status |
| **Wi-Fi Notifications** | Sends HTTP requests to Telegram on alarm events |

**Interrupt-driven sensing:** The reed switch on the Sensor Node uses `attachInterrupt()` for immediate response to door events — no polling loops.

**Non-volatile storage:** The ESP32 Preferences library persists the current mode (armed/disarmed) and foot-traffic count to NVS flash, so the system resumes its state after power loss.

## Parts List

| Part |
|---|---|
| ESP32-C6 DevKit (×2) | 
| Reed switch + magnet | 
| 16×2 LCD Screen | 
| IO Expander (PCF8574N) | 
| NeoPixel LED Strip (WS2812B) | 
| Passive Piezo Buzzer | 
| Potentiometer (50kΩ) | 
| Push Buttons (SPST) | 
| Jumper Wires | 
| Breadboard | 

## Technologies

C/C++ · ESP-IDF / Arduino · FreeRTOS · ESP-NOW · Wi-Fi (HTTP) · I2C · PWM · GPIO Interrupts · NVS · Telegram Bot API

## Course

ECE 4180 — Embedded Systems Design, Georgia Tech (Spring 2026)
