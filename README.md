# Smart-Energy-Management-System
An IoT-enabled, ESP32-powered Smart Energy Management System (SEMS) featuring high-precision AC power metering (PZEM-004T v3), zero-wear solid-state load switching, local color TFT visualization (ST7735), web-based dashboard management, and deterministic rule-based safety protection.
[github_readme_file.md](https://github.com/user-attachments/files/34116563/github_readme_file.md)
# IoT-Based Smart Energy Management System (SEMS)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: ESP32](https://img.shields.io/badge/Platform-ESP32-red.svg)](https://www.espressif.com/en/products/socs/esp32)
[![Language: C++](https://img.shields.io/badge/Language-C%2B%2B-green.svg)](https://www.arduino.cc/)
[![Hardware: PZEM-004T](https://img.shields.io/badge/Sensor-PZEM--004T%20v3-orange.svg)](https://pzem-004t.com/)

An integrated, low-cost (~₦49,400) IoT smart energy meter and load controller engineered on the ESP32 platform. The system provides real-time single-phase AC electrical parameter measurement, local graphical display feedback via an ST7735 TFT screen, zero-wear load switching using a Solid-State Relay (SSR), embedded web dashboard monitoring, and local non-volatile credit tracking.

---

## 🌟 Key Features

* **High-Precision Multi-Parameter Sensing:** Real-time sampling of Root Mean Square (RMS) Voltage, RMS Current, Active Power ($P$), Apparent Power ($S$), Reactive Power ($Q$), Power Factor ($\text{PF}$), Line Frequency ($f$), and Energy Accumulation ($E_{\text{kWh}}$) using the PZEM-004T v3 metering IC.
* **Deterministic Rule-Based Protection Engine:**
  * **Voltage Guardrails:** Automated relay cut-off during brownouts ($V < 190\text{V}$) or over-voltage spikes ($V > 253\text{V}$).
  * **Overload Latching:** Immediate trip upon overcurrent ($I > 10\text{A}$) with a 30-second cooldown timer. Latches permanently if tripped 3 times in 10 minutes.
  * **Bypass / Energy Theft Detection:** Triggers an instant alert if load current ($I > 0.05\text{A}$) flows while the relay is driven `OFF`.
  * **Credit Limit Enforcement:** Automatic relay disconnection when credit balance reaches $\le \text{NGN } 0.00$, accompanied by a low-credit warning at $\le \text{NGN } 200.00$.
* **Zero-Wear Solid-State Switching:** Driven via a 25A AC Solid-State Relay (SSR) with zero-crossing switching logic, eliminating mechanical contact arcing and contact welding.
* **Dual-Domain Local & Cloud Visualization:**
  * **Local UI:** 1.8-inch ST7735 128x160 color TFT display driven via Hardware SPI.
  * **Embedded Web Server:** Hosts a local HTTP REST dashboard (`http://smartenergy.local`) for live monitoring, prepaid account recharge, parameter tuning, and relay manual override.
* **Persistent Non-Volatile Memory (NVS):** Synchronized saving of credit balance, energy indices, and parameter bounds via the ESP32 `Preferences` API to ensure resilience against sudden grid power failure.
* **Multi-Tier Network Resilience:** Auto-reconnects to local Wi-Fi router; automatically spins up a fallback Access Point (`SmartEnergy-Setup`) if router connectivity is unavailable.

---

## 📌 Hardware Pinout Connections

The ESP32 microcontroller coordinates peripherals using hardware UART2, SPI, and dedicated GPIO lines:

| Subsystem / Module | Module Pin | ESP32 GPIO Pin | Protocol / Signal Type |
| :--- | :--- | :--- | :--- |
| **PZEM-004T v3 Meter** | TX Pin | **GPIO 16** (RX2) | Hardware UART2 Receive (3.3V TTL) |
| **PZEM-004T v3 Meter** | RX Pin | **GPIO 17** (TX2) | Hardware UART2 Transmit (3.3V TTL) |
| **ST7735 TFT Display** | CS (Chip Select) | **GPIO 5** | Hardware SPI Output |
| **ST7735 TFT Display** | SCLK (Clock) | **GPIO 18** | Hardware SPI Output |
| **ST7735 TFT Display** | DC (Data/Command) | **GPIO 21** | Hardware SPI Output |
| **ST7735 TFT Display** | RST (Reset) | **GPIO 22** | Hardware SPI Output |
| **ST7735 TFT Display** | MOSI (Master Out) | **GPIO 23** | Hardware SPI Output |
| **Solid-State Relay (SSR)**| Control Signal | **GPIO 4** | Digital Output (Active-LOW) |
| **Manual Override Button** | Switch Input | **GPIO 32** | Digital Input (Internal Pull-Up) |

> ⚠️ **Critical Safety Wiring Note:** The PZEM-004T voltage sensing taps and split-core Current Transformer (CT) clamp **MUST** be connected on the *supply side* (upstream) of the relay contacts. This enables continuous grid voltage sampling while the relay is open, allowing grid stability verification before energizing loads and enabling bypass/theft detection.

---

## 📐 Circuit Architecture & System Analysis

### Single-Phase AC Formulas
* **RMS Calculations:**
  $$V_{\text{RMS}} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} v_i^2}, \quad I_{\text{RMS}} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} i_i^2}$$
* **Power Quantities:**
  $$\text{Apparent Power } (S) = V_{\text{RMS}} \times I_{\text{RMS}} \quad [\text{VA}]$$
  $$\text{Active Power } (P) = V_{\text{RMS}} \times I_{\text{RMS}} \times \cos\phi \quad [\text{W}]$$
  $$\text{Reactive Power } (Q) = \sqrt{S^2 - P^2} \quad [\text{VAR}]$$
* **Discrete Energy Accumulation:**
  $$E_{\text{kWh}} = \frac{\sum (P_i \times \Delta t)}{3,600,000}$$
  *(Where $P_i$ is active power in Watts, and $\Delta t$ is the sampling interval in seconds).*

---

## 🛠️ Software Installation & Flashing Guide

### 1. Prerequisites & Dependencies
Install the [Arduino IDE](https://www.arduino.cc/en/software) (v2.0+) and add the ESP32 board manager URL (`https://dl.espressif.com/dl/package_esp32_index.json`).

Install the following libraries via the Arduino Library Manager:
* `PZEM004Tv30` (by Jakub Mandula)
* `Adafruit GFX Library` (by Adafruit)
* `Adafruit ST7735 and ST7789 Library` (by Adafruit)
* Built-in ESP32 core libraries (`WiFi`, `WebServer`, `ESPmDNS`, `Preferences`, `SPI`)

### 2. Flashing Procedure
1. Clone this repository:
   ```bash
   git clone https://github.com/YOUR-USERNAME/smart-energy-management-system.git
   ```
2. Open `firmware/SmartEnergySystem.ino` in Arduino IDE.
3. Select your ESP32 board target: **Tools -> Board -> ESP32 Arduino -> ESP32 Dev Module**.
4. Configure Wi-Fi credentials inside `SmartEnergySystem.ino`:
   ```cpp
   const char* WIFI_SSID = "YOUR_WIFI_SSID";
   const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
   ```
5. Connect your ESP32 via Micro-USB/USB-C, select the correct COM port, and click **Upload**.

---

## 🌐 Web Dashboard Endpoints

When connected to Wi-Fi, navigate to `http://smartenergy.local` or the assigned IP address:

| REST Endpoint | HTTP Method | Description |
| :--- | :--- | :--- |
| `/` | `GET` | Renders the primary live monitoring dashboard (HTML/JS). |
| `/login` | `GET / POST` | User session authentication handler. |
| `/data` | `GET` | Returns live JSON telemetry (Voltage, Current, Power, Energy, Balance). |
| `/recharge` | `POST` | Processes credit top-up requests (`amount`). |
| `/setRelay` | `POST` | Toggles relay position manually (`state=1` or `state=0`). |
| `/settings` | `POST` | Updates system voltage/current protection boundaries and tariffs. |
| `/password` | `POST` | Updates admin access credentials. |
| `/factoryReset` | `POST` | Erases NVS memory and restores default factory parameters. |

---

## 💰 Bill of Engineering Measurement and Evaluation (BEME)

| S/N | Material / Component Description | Quantity | Unit Cost (₦) | Total Cost (₦) |
| :---: | :--- | :---: | :---: | :---: |
| 1 | ESP32 Dev Module Microcontroller | 1 | 10,000 | 10,000 |
| 2 | PZEM-004T v3 Energy Metering Module + CT Clamp | 1 | 15,000 | 15,000 |
| 3 | LM2596 DC-DC Buck Converter Module | 1 | 6,000 | 6,000 |
| 4 | 1.8-inch ST7735 Color TFT Display Module | 1 | 5,500 | 5,500 |
| 5 | 3.7V Lithium-Ion Battery (3000mAh) | 2 | 1,500 | 3,000 |
| 6 | Perforated Circuit Board (5x7 cm Perfboard) | 1 | 1,500 | 1,500 |
| 7 | Female / Male Header Pin Strips Set | 1 Pack | 1,200 | 1,200 |
| 8 | Solid-State Relay (SSR 25A 24-380V AC) | 1 | 1,200 | 1,200 |
| 9 | Tactile Override Switch / Power Pushbutton | 1 | 500 | 500 |
| 10 | System Project Protective Enclosure Casing | 1 | 4,000 | 4,000 |
| 11 | Assembly Fasteners & Adhesive Glue | 1 Set | 1,500 | 1,500 |
| **TOTAL** | **BILL OF ENGINEERING MEASUREMENT & EVALUATION** | - | - | **₦49,400** |

---

## 📁 Repository Structure

```text
smart-energy-management-system/
├── firmware/
│   └── SmartEnergySystem.ino       # ESP32 Production C++ Firmware
├── hardware/
│   ├── schematics/                 # Schematic Diagram Files (.PDF / .KiCAD)
│   ├── pcb/                        # Dual-Layer PCB Gerber Fabrication Files
│   └── enclosure/                  # 3D CAD Enclosure Models (.STL / .STEP)
├── docs/
│   └── SEMS_Project_Documentation.docx  # Full Audited Technical Documentation
├── media/
│   ├── block_diagram.png           # Hardware Architecture Diagram
│   └── prototype_photo.jpg         # Assembled Hardware Prototype Image
├── .gitignore
├── LICENSE                         # Open-Source MIT License
└── README.md                       # Repository Overview Document
```

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
