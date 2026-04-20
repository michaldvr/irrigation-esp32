# 🌿 Smart Irrigation System (ESP32 + ESPHome)
An automated soil moisture monitoring and irrigation control system for three plants (**Hibiscus**, **Ficus**, and **Onion**). Powered by ESP32 and fully integrated with Home Assistant.

## 🚀 Key Features
**Precise Measurement:** ADC sensors utilizing a 5-minute median filter to eliminate digital noise, combined with a 5-minute heartbeat to ensure data consistency even when values are stable.

**Hardware Fail-Safe:** Built-in safety_timer in ESPHome logic that automatically shuts down the pump and valves after 5 seconds, independent of Home Assistant connectivity.

**Tank Protection:** Integrated float switch prevents pump operation if the water reservoir is empty, protecting the motor from running dry.

**Smart Sequencing:** Automated 500ms delay between valve and pump activation to prevent water hammer and power spikes.

**Time-Window Logic:** Irrigation is restricted to specific hours (14:00 - 20:00) to remove night time and morning time with sun on moisture sensors.

## 🛠 Hardware
**MCU: ESP32** (DevKit V1)

**Sensors:** 3x Capacitive Soil Moisture Sensors (GPIO 32, 34, 35)

1x Water Level **Float Switch** (GPIO 13)

**Actuators:**
  - 3x Solenoid Valves (GPIO 27, 14, 12)
  - 1x Main Submersible Pump (GPIO 26)

## 📦 Installation
1. Clone this repository into your ESPHome configuration directory.

2. Create a `secrets.yaml` file with your credentials:
```yaml
wifi_ssid: "your_wifi_name"
wifi_password: "your_password"
api_encryption_key: "your_generated_key"
ota_password: "your_ota_password"
fallback_ap_password: "your_fallback_ap_password"
```

3. Flash your ESP32:
```bash
esphome run irrigation-system.yaml
```

## 📊 Signal Processing Logic
To prevent "jittery" data and false triggers caused by WiFi radio interference, the system uses a robust filtering stack:

**Median Filter:** Takes the middle value from 300 samples (300 seconds) to ignore spikes.

**Linear Calibration:** Maps raw voltage (1.05V - 2.85V) to a 0-100% moisture scale.

**Heartbeat/Delta:** Sends data immediately if a 0.1% change is detected, or at least every 5 minutes regardless of change.

## 🔒 Safety & Interlock Logic
The system follows a strict "permission" hierarchy before activating the pump:

**Water Level Check:** water_level_status must be OK.

**State Check:** The pump_main must be currently OFF.

**Cooldown Check:** The cooldown_script must not be running (prevents rapid cycling).

**Time Check:** Must be within the allowed daily time window.

**Hardware Timer:** A physical safety_timer kills all power if the software fails to send a stop command.
