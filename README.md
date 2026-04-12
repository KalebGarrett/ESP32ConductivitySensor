# 🧪 ESPHome Saltwater Hot Tub Monitor (TDS → Salinity)

A simple, **real-world calibrated** system for monitoring salt levels in a hot tub using an ESP32 + analog TDS probe.

---

# 📌 Overview

This project uses:

- ESP32 (WROOM-32)
- CQRobot / DFRobot TDS probe (analog)
- ESPHome
- Home Assistant

To measure:

- **TDS (ppm)** → calibrated to match real salt levels  
- **Salinity (ppm)** → derived directly from calibrated TDS  

---

# 🧠 Key Concept

❌ Generic formulas are unreliable  
✅ Calibration against a real salt test = accurate results  

---

# 🔌 Wiring

| Probe Wire | ESP32 Pin |
|-----------|----------|
| Red       | 3.3V     |
| Black     | GND      |
| Green     | GPIO34   |

---

# 📸 Photos & Setup

---

## 🔌 ESP32 WROOM Pin Layout

![ESP32 Pinout](Documentation/Images/esp32_pinout.png)

### Pins Used

- GPIO34 → Analog input (TDS signal)
- 3.3V → Sensor power
- GND → Ground

⚠️ GPIO34 is input-only and cannot be used as an output.

---

## 🧪 Physical Setup

![Setup](Documentation/Images/setup.jpg)

### Notes

- Keep the probe submerged in circulating water  
- Avoid placing directly in front of jets (bubbles cause noise)  
- Ensure solid wiring connections  

---

## 📊 Home Assistant Dashboard

![Dashboard](Documentation/Images/dashboard.png)

---

## 🧂 Test Strip Calibration

![Test Strip](Documentation/Images/test_strip.png)

Calibration is done by matching the sensor reading to the test strip result.

---

# ⚙️ ESPHome Configuration

```yaml
sensor:
  - platform: adc
    pin: GPIO34
    id: tds_voltage
    name: "TDS Voltage"
    update_interval: 5s
    attenuation: 12db
    filters:
      - sliding_window_moving_average:
          window_size: 10
          send_every: 5
      - multiply: 3.3

  - platform: template
    name: "TDS ppm"
    unit_of_measurement: "ppm"
    update_interval: 5s
    lambda: |-
      float voltage = id(tds_voltage).state;

      // Calibration factor (tune this)
      float scale = 410.0;

      float tds = voltage * scale;

      return tds;