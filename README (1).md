# ESP32-Based Heart Rate Monitoring System

This project uses an ESP32 microcontroller to monitor heart rate in real time using an analog Pulse Sensor. The ESP32 reads the sensor's PPG signal through its ADC, applies digital filtering and peak detection to calculate beats per minute (BPM), displays the result on an I²C OLED screen, and triggers a buzzer alert when the heart rate goes outside a safe range.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Hardware Requirements](#hardware-requirements)
3. [Software Requirements](#software-requirements)
4. [Circuit Diagram and Pin Connections](#circuit-diagram-and-pin-connections)
5. [Installation and Setup](#installation-and-setup)
6. [Usage](#usage)
7. [How It Works](#how-it-works)
8. [Calibration and Tuning](#calibration-and-tuning)
9. [Troubleshooting](#troubleshooting)
10. [Future Improvements](#future-improvements)
11. [Contributing](#contributing)
12. [License](#license)

---

### Project Overview

This project uses the ESP32 microcontroller to acquire a photoplethysmogram (PPG) signal from a Pulse Sensor, process it in software to detect individual heartbeats, and display the live heart rate (BPM) on an OLED screen. An onboard buzzer sounds an alert if the measured BPM exceeds or drops below configurable thresholds. This setup allows for a fully self-contained, real-time heart rate monitor with no external app or cloud connection required.

### Hardware Requirements

- **ESP32 Development Board**
- **Pulse Sensor** (analog PPG output, typically has 3 pins: VCC, GND, Signal)
- **128x64 I²C OLED Display** (SSD1306, address 0x3C)
- **Buzzer** (active or passive)
- **Micro USB Cable** (for programming and power)
- **Breadboard and jumper wires**
- **Computer** (with USB port and Arduino IDE installed)

### Software Requirements

- [Arduino IDE](https://www.arduino.cc/en/software)
- **ESP32 Board Support** for Arduino IDE
- **Adafruit GFX Library** (for Arduino)
- **Adafruit SSD1306 Library** (for Arduino)

### Circuit Diagram and Pin Connections

**Pulse Sensor → ESP32**

| Pulse Sensor Pin | ESP32 Pin |
| ----------------- | --------- |
| VCC                | 3V3       |
| GND                | GND       |
| Signal             | GPIO 34   |

**OLED (SSD1306, I²C) → ESP32**

| OLED Pin | ESP32 Pin |
| -------- | --------- |
| VCC      | 3V3       |
| GND      | GND       |
| SDA      | GPIO 21   |
| SCL      | GPIO 22   |

**Buzzer → ESP32**

| Buzzer Pin   | ESP32 Pin |
| ------------ | --------- |
| + (Signal)   | GPIO 25   |
| − (GND)      | GND       |

GPIO 34 is used for the Pulse Sensor because it belongs to ADC1, which avoids conflicts with the Wi-Fi radio (unlike ADC2 pins). See [`docs/wiring.md`](docs/wiring.md) for a full wiring reference and signal-acquisition notes.

### Installation and Setup

#### Step 1: Install Arduino IDE and ESP32 Board Support

1. Download and install the [Arduino IDE](https://www.arduino.cc/en/software).
2. In **Arduino IDE**, go to **File > Preferences**.
3. In **Additional Board Manager URLs**, add:

   ```
   https://dl.espressif.com/dl/package_esp32_index.json
   ```

4. Go to **Tools > Board > Board Manager**, search for "ESP32," and install the **ESP32 by Espressif Systems** package.

#### Step 2: Install Required Libraries

1. In **Arduino IDE**, go to **Sketch > Include Library > Manage Libraries**.
2. Search for and install:
   - **Adafruit GFX Library**
   - **Adafruit SSD1306**

#### Step 3: Clone or Download This Project

Clone this repository or download it as a ZIP file:

```
git clone https://github.com/<your-username>/ESP32-Heart-Rate-Monitor.git
```

Or download the ZIP file and extract it.

#### Step 4: Open the Project in Arduino IDE

1. Open `src/ESP32_HeartRate_Monitor.ino` in Arduino IDE.
2. Select your ESP32 board:
   - Go to **Tools > Board** and choose **ESP32 Dev Module**.
3. Select the correct port:
   - Go to **Tools > Port** and select the port where your ESP32 is connected.

#### Step 5: Wire the Hardware

Connect the Pulse Sensor, OLED, and buzzer to the ESP32 as described in [Circuit Diagram and Pin Connections](#circuit-diagram-and-pin-connections).

#### Step 6: Upload the Code to ESP32

1. Connect your ESP32 to your computer using a USB cable.
2. Click **Upload** in Arduino IDE to compile and upload the code to your ESP32.
3. Open the **Serial Monitor** (Ctrl + Shift + M) to view debug output. Set the baud rate to `115200`.

### Usage

1. Power the ESP32 via USB.
2. Wait for the OLED to display **"Initializing..."**, then the main monitoring screen.
3. Place your fingertip gently and steadily on the Pulse Sensor.
4. The OLED will update with the live BPM reading once a stable pulse is detected.
5. If BPM rises above or falls below the configured thresholds, the buzzer will sound an intermittent alert.
6. If no valid signal is detected (e.g., no finger on the sensor), the OLED will show a prompt to place a finger on the sensor.

### How It Works

1. **Signal Acquisition** — The ESP32 ADC samples the Pulse Sensor's analog output at a fixed interval (~200 Hz).
2. **DC Offset Removal** — A slow moving average tracks the signal baseline, which is subtracted to isolate the pulsatile (AC) component.
3. **Low-Pass Filtering** — A fast moving average smooths the signal to reduce motion and electrical noise.
4. **Adaptive Peak Detection** — Beats are detected when the filtered signal crosses a dynamically scaled threshold, with a refractory period to prevent double-counting.
5. **BPM Calculation** — The interval between beats (IBI) is converted to BPM and smoothed with a rolling average.
6. **Display & Alert** — BPM is shown on the OLED in real time, and the buzzer activates if BPM falls outside the safe range.

### Calibration and Tuning

These constants can be adjusted at the top of the sketch:

| Constant | Description | Default |
| --- | --- | --- |
| `SAMPLE_INTERVAL_MS` | ADC sampling period | 5 ms |
| `LOWPASS_ALPHA` | Low-pass filter smoothing factor | 0.3 |
| `PEAK_THRESHOLD_FACTOR` | Peak detection sensitivity | 0.6 |
| `REFRACTORY_PERIOD_MS` | Minimum time between accepted beats | 300 ms |
| `BPM_LOW_THRESHOLD` | Buzzer alert lower bound | 50 BPM |
| `BPM_HIGH_THRESHOLD` | Buzzer alert upper bound | 100 BPM |

> **Note:** This project is an educational/prototype device and is **not a certified medical instrument**. Do not use it for medical diagnosis.

### Troubleshooting

- **Issue: OLED does not turn on / "SSD1306 allocation failed" in Serial Monitor**
  - Verify SDA/SCL wiring (GPIO 21 / GPIO 22).
  - Confirm the OLED's I²C address (commonly `0x3C`); update `OLED_ADDRESS` in the sketch if needed.

- **Issue: BPM reading is unstable or inaccurate**
  - Ensure the fingertip is pressed gently and steadily against the sensor with minimal movement.
  - Adjust `PEAK_THRESHOLD_FACTOR` and `LOWPASS_ALPHA` for your specific sensor module.
  - Shield the sensor from strong ambient light.

- **Issue: Buzzer does not sound during an alert**
  - Confirm buzzer wiring on GPIO 25 and shared ground with the ESP32.
  - If using a passive buzzer, confirm it responds to a simple digital HIGH/LOW signal, or adapt the code to use `tone()`.

### Future Improvements

- Add SpO2 estimation using a dual-wavelength (red/IR) PPG sensor
- Stream data to a cloud dashboard via Wi-Fi (MQTT/HTTP)
- Add data logging to an SD card or NVS flash
- Implement an FFT-based / bandpass-filter signal pipeline
- Add battery power management for a wearable form factor
- Add a Bluetooth Low Energy (BLE) companion app for mobile viewing

### Contributing

Contributions are welcome! Please fork this repository, make your changes, and submit a pull request.

---

### License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

---

**Tech Stack:** ESP32 · Embedded C/C++ · Pulse Sensor · ADC · PPG · I²C · Signal Processing
