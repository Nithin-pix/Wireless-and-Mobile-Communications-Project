# 🌱 IoT-Based Smart Agriculture over 5G

A real-time agricultural monitoring system that integrates environmental sensors with a Raspberry Pi and transmits data to the cloud over a 5G/LTE cellular network.

> **Course Project** — BECE317L: Wireless and Mobile Communications  
> Vellore Institute of Technology (VIT) Chennai | April 2026  
> **Authors:** Deepak A (23BEC1220) · Nithin V (23BEC1346) · Pranav S (23BEC1350)

---

## 📌 Overview

Traditional agriculture relies on manual monitoring, which is time-consuming and inefficient. This project addresses that by deploying a network of sensors on a Raspberry Pi that continuously collects environmental data — rainfall, light intensity, and water level — and uploads it to the **ThingSpeak** cloud platform over a **5G cellular network** for real-time visualization and remote access.

---

## ✨ Features

- 🌧️ **Rain detection** — monitors precipitation to support smart irrigation decisions
- ☀️ **Light intensity sensing** — analyzes day/night cycles to optimize plant growth
- 📏 **Ultrasonic water level measurement** — non-contact distance sensing (2–400 cm)
- 📡 **5G/LTE connectivity** — reliable data transmission via Waveshare 5G HAT, works even without Wi-Fi
- ☁️ **Cloud dashboard** — real-time data visualization on ThingSpeak via HTTP
- 🔒 **Secure transmission** — TLS 1.3 encrypted HTTPS communication confirmed via Wireshark
- 🐍 **Python-based** — clean, modular, and extensible codebase
- 📈 **Scalable architecture** — easily add more sensors or nodes in the future

---

## 🛠️ Hardware Components

| Component | Purpose |
|---|---|
| Raspberry Pi 4 Model B | Central processing unit (SBC) |
| LDR Sensor Module | Light intensity detection (GPIO Pin 27) |
| Rain/Moisture Sensor (MH-RD) | Rainfall detection (GPIO Pin 22) |
| Ultrasonic Sensor HC-SR04 | Water level / distance measurement (GPIO Pins 23 & 24) |
| Waveshare 5G HAT (M.2 to 4G/5G) | Cellular communication module |
| 5G SIM Card | Network authentication and data connectivity |
| 5G Antennas | MiMo RF signal transmission/reception |

---

## 🗂️ System Architecture

```
SENSOR INPUTS          PROCESSING UNIT         COMMUNICATION
┌─────────────┐        ┌──────────────┐        ┌─────────────────┐
│  LDR Sensor │──────▶ │              │        │  Waveshare 5G   │
│  Rain Sensor│──────▶ │ Raspberry Pi │──────▶ │  HAT + SIM Card │──▶ ThingSpeak
│ HC-SR04     │──────▶ │  (Python)    │        │  + Antennas     │    Cloud
└─────────────┘  GPIO  └──────────────┘  USB   └─────────────────┘
                        HTTP Requests            5G Network
```

---

## 💻 Software

**Language:** Python 3  
**Libraries:** `RPi.GPIO`, `time`, `requests`  
**Cloud Platform:** [ThingSpeak](https://thingspeak.com)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/iot-smart-agriculture-5g.git
cd iot-smart-agriculture-5g

# Install dependencies
pip install RPi.GPIO requests
```

### Configuration

Open `main.py` and replace the placeholder with your ThingSpeak API key:

```python
API_KEY = "YOUR_THINGSPEAK_API_KEY"
```

### Run

```bash
python main.py
```

---

## 📄 Code

```python
import RPi.GPIO as GPIO
import time
import requests

# GPIO pin definitions
ldr_pin  = 27   # LDR sensor
rain_pin = 22   # Rain sensor
trig     = 23   # Ultrasonic trigger
echo     = 24   # Ultrasonic echo

API_KEY = "YOUR_API_KEY"

GPIO.setmode(GPIO.BCM)
GPIO.setup(ldr_pin,  GPIO.IN)
GPIO.setup(rain_pin, GPIO.IN)
GPIO.setup(trig,     GPIO.OUT)
GPIO.setup(echo,     GPIO.IN)

def get_distance():
    GPIO.output(trig, False)
    time.sleep(0.5)
    GPIO.output(trig, True)
    time.sleep(0.00001)
    GPIO.output(trig, False)
    while GPIO.input(echo) == 0:
        start = time.time()
    while GPIO.input(echo) == 1:
        end = time.time()
    duration = end - start
    return round(duration * 17150, 2)  # distance in cm

print("System started")

while True:
    ldr      = GPIO.input(ldr_pin)
    rain     = GPIO.input(rain_pin)
    distance = get_distance()

    print(f"Light: {ldr} | Rain: {rain} | Distance: {distance} cm")

    payload = {
        "api_key": API_KEY,
        "field1":  ldr,
        "field2":  rain,
        "field3":  distance
    }
    try:
        requests.get("https://api.thingspeak.com/update", params=payload)
        print("Data sent to ThingSpeak")
    except:
        print("Error sending data")

    time.sleep(15)  # ThingSpeak rate limit: minimum 15 seconds
```

---

## 📡 Network Analysis (Wireshark)

Network traffic was captured and analyzed using **Wireshark** to validate communication reliability.

| Metric | Result |
|---|---|
| DNS Resolution | `api.thingspeak.com` → `54.235.197.69` ✅ |
| TCP Handshake | SYN → SYN-ACK → ACK (port 443) ✅ |
| Encryption | TLSv1.3 (HTTPS) — payload fully encrypted ✅ |
| Latency (RTT) | **~21 ms** (0.300 s − 0.279 s) ✅ |
| Packet Loss | None observed ✅ |
| Data Pattern | Periodic bursts every ~15 s (matches upload interval) ✅ |

The low latency of **21 ms** confirms that 5G cellular connectivity is well-suited for real-time IoT applications.

---

## 📊 ThingSpeak Dashboard

Sensor readings are mapped to three ThingSpeak fields:

| Field | Sensor | Value |
|---|---|---|
| Field 1 | LDR | `0` = Light detected, `1` = Dark |
| Field 2 | Rain Sensor | `0` = Wet (rain), `1` = Dry |
| Field 3 | Ultrasonic (HC-SR04) | Distance in cm |

---

## 🔮 Future Work

- [ ] Automated irrigation control triggered by sensor thresholds
- [ ] Additional sensors: temperature, humidity, soil moisture
- [ ] Mobile app for remote monitoring and control
- [ ] Machine learning for predictive crop analytics
- [ ] Solar-powered deployment for energy efficiency
- [ ] Multi-node expansion for large-scale farms

---

## 📚 References

1. Department of Telecommunications (DoT), *5G Lab Manual*, Edition 1, 2025.
2. Signaltron Systems Pvt. Ltd., *5G Labs Training Manual*, Rev. 2.0, Feb. 2025.
3. V. Mahore et al., "An IoT-Enabled Tractor Data Sensing System for Precision Agriculture," *INCOFT 2023*.
4. S. Muthusundari et al., "A Smart Farming: Implementation of IoT Based Automated Agricultural Monitoring System," *ICTEASD 2023*.
5. B. Ramesh et al., "Farm Easy- IoT based Automated Irrigation, Monitoring and Pest Detection using ThingSpeak," *RTEICT 2020*.

---

## 📝 License

This project was developed for academic purposes at VIT Chennai under the guidance of **Dr. Vydeki D**, School of Electronics Engineering.
