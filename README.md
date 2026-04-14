# 🌿 Smart Irrigation System (ESP32 + ESPHome)

An automated soil moisture monitoring and irrigation control system for three plants (**Hibiscus**, **Ficus**, and **Onion**). Powered by ESP32 and fully integrated with Home Assistant.

## 🚀 Key Features
- **Precise Measurement:** ADC sensors utilizing a 5-minute **Median Filter** to eliminate digital noise and interference.
- **Safety First:** Built-in `safety_timer` in ESPHome logic that automatically shuts down the pump and valves after 5 seconds as a fail-safe, independent of Home Assistant connectivity.
- **Stable Data Stream:** Optimized 10s update interval to maintain active API connection even in low-signal environments (~ -86 dBm).
- **Long-term Analytics:** Home Assistant Recorder configuration tuned for 1-year data retention.

## 🛠 Hardware
- **MCU:** ESP32 (DevKit V1)
- **Sensors:** 3x Capacitive Soil Moisture Sensors (GPIO 32, 34, 35)
- **Actuators:** - 3x Solenoid Valves (GPIO 27, 14, 12)
  - 1x Main Submersible Pump (GPIO 26)
- **Mechanical:** Stainless steel (V2A) heavy-duty brackets for mounting float switches and sensors in the water reservoir.

## 📦 Installation
1. Clone this repository into your ESPHome configuration directory.
2. Create a `secrets.yaml` file and provide your credentials:
   ```yaml
   wifi_ssid: "your_wifi_name"
   wifi_password: "your_password"
   api_encryption_key: "your_generated_key"
   ota_password: "your_ota_password"
   fallback_ap_password: "your_fallback_ap_password"
Flash your ESP32:

Bash
esphome run irrigation-system.yaml
📊 Signal Processing Logic
To prevent "jittery" data and false irrigation triggers caused by WiFi radio interference or power ripples, the system uses a robust filtering stack:

Median Filter: Takes the middle value from 300 samples (5 minutes of data at 1s intervals).

Linear Calibration: Maps raw voltage to a 0-100% moisture scale.

Throttling: Sends data to Home Assistant every 10 seconds to reduce database bloat while maintaining a "live" feel.

YAML
filters:
  - median:
      window_size: 300
      send_every: 10
  - calibrate_linear:
      - 2.85 -> 0.0
      - 1.05 -> 100.0
🔒 Fail-Safe Logic
The device-side automation ensures that even if the network goes down during an active irrigation cycle, the hardware remains protected:

Sequential Activation: 500ms delay between valve and pump activation to prevent water hammer and current spikes.

Auto-Shutdown: All outputs are forced OFF if the safety_timer is not reset by a new command, preventing accidental over-watering or pump burnout.
