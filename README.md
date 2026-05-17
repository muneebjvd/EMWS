# Environmental Monitoring and Warning System

An embedded bare-metal system implemented on the **STM32F4 (ARM Cortex-M4)** platform. The system provides real-time data acquisition, environmental parameter processing, an interactive OLED user interface, and automatic audio-visual hazard warnings.

## 🚀 Features
* **Multi-Sensor Data Fusion:** Interoperability with digital, analog, and time-modulated protocol sensors.
* **Intelligent Alarm Subsystem:** Real-time visual (LED) and audio (Buzzer) indicators triggered instantly when environmental safety thresholds are violated.
* **Bare-Metal Driver Control:** Optimized hardware driver architecture utilizing STM32 timers, input capture interrupts, and I2C registers.
* **Robust UI Dashboard:** Features text elements, customized layouts, and live graphic progress bars rendering the Air Quality Index (AQI).

---

## 🛠️ Hardware Components

| Component | Description | Operating Principle | Estimated Cost (Rs) |
| :--- | :--- | :--- | :--- |
| **STM32F407VG** | 32-bit ARM Cortex-M4 MCU | 168MHz CPU with FPU, multi-peripheral pipeline | *Issued* |
| **DHT11** | Temperature & Humidity Sensor | Capacitive element & thermistor via single-wire line | 400 |
| **MQ135** | Air Quality Gas Sensor | Semiconductor-based electrochemical resistance shifts | 500 |
| **HC-SR04** | Ultrasonic Distance Sensor | Time-of-flight evaluation via sonic echo bounce | 350 |
| **SSD1306** | 128x64 Matrix OLED | Monochrome display using fast-mode $I^2C$ protocol | 600 |
| **Active Buzzer** | Audio Warning Module | Piezoelectric sound pressure vibration | 200 |
| **LED** | Visual Safety Light | Light-emitting diode indication | *MCU Stock* |

---

## 📐 System Pin Connectivity Architecture

| STM32 Pin | Peripheral Component | Dedicated Function |
| :---: | :--- | :--- |
| **PB9** | DHT11 Data | Bidirectional single-wire data stream |
| **PA1** | MQ135 Output | Single-ended analog voltage input |
| **PE10** | HC-SR04 TRIG | High-precision output trigger pulse |
| **PA0** | HC-SR04 ECHO | Timer Input Capture (IC) for echo tracking |
| **PD14** | Warning LED | GPIO digital output driver for visual alarm |
| **PA5** | Buzzer | GPIO digital output driver for audio alarm |
| **PB6** | SSD1306 SCL | Master $I^2C$ hardware clock line |
| **PB7** | SSD1306 SDA | Master $I^2C$ bidirectional data line |

---

## 🔬 Peripheral Configuration & Execution Strategy

* **Timer 1 (TIM1):** Set to run as a 1MHz counter ($72\text{ MHz} / 72$ prescaler matching a 72MHz system clock) to deliver precise microsecond ($\mu\text{s}$) delay generation for bit-banging the proprietary single-wire DHT11 protocol.
* **Timer 2 (TIM2):** Running at 1MHz configured in **Input Capture Mode with Interrupts**. It actively latches the exact rising and falling edges of the HC-SR04 Echo signal pulse to compute real-time distances with millimetric resolution.
* **ADC1:** Configured with 12-bit resolution ($0 \text{ to } 4095$ range) tracking Channel 1. Raw voltages from the MQ135 sensor are sampled and parsed through integrated logarithmic calibration curves to yield PPM and AQI targets.
* **I2C1:** Clocked at Fast Mode (400kHz) utilizing 7-bit addressing schemas to manage full pixel buffer serialization, frame rendering loops, and boot initialization sequences on the SSD1306 OLED screen.
* **Nested Vectored Interrupt Controller (NVIC):** Strict priority-based interrupt hierarchies ensure time-critical inputs (like ultrasonic echoes) preempt non-critical display updates.

---

## 📈 System Execution Flowchart

The firmware runs an initial system hardware configuration check before executing a continuous polling and processing loop:

```text
[System Initialization] 
          │
          ▼
[Initialize Peripherals: GPIO, Timers, ADC, I2C]
          │
          ▼
[Initialize Sensors & OLED Display Screen]
          │
          ▼
    ┌─► [Main Execution Loop]
    │     │
    │     ▼
    │   [Fetch Distance Data via HC-SR04 Input Capture]
    │     │
    │     ▼
    │   [Evaluate Safety Bounds & Threshold Check]
    │     │
    │     ├─► (Hazard Found: Yes) ──► [Trigger LED/Buzzer Alarms & Render Alert UI] ──┐
    │     │                                                                           │
    │     └─► (Hazard Found: No) ─────────────────────────────────────────────────────┼─► [Read Air Quality (ADC) from MQ135]
    │                                                                                 │
    │                                                                                 ▼
    │                                                                       [Poll Temp & Humidity from DHT11]
    │                                                                                 │
    │                                                                                 ▼
    │                                                                       [Update Master UI Graphics Display]
    │                                                                                 │
    │                                                                                 ▼
    └─────────────────────────────────────────────────────────────────────── [Short Execution Delay Loop]

🧩 Complex Engineering Implementations (CEP Attributes)WP1 (Depth of Knowledge): Showcases structural masterclass over bare-metal registers, timing requirements, data processing, and priority-level task management.WP2 (Conflicting Requirements Solution): Solved asynchronous execution timing locks (microsecond-level bit-banging vs long echo wait states vs busy I2C screen drawing cycles) by running dedicated decoupled system timers alongside a priority interrupt layout.WP5 (Custom Algorithms): Features customized drivers including custom bit-banged single-wire protocols, integrated checksum mathematical verifications ($DHT11$), gas resistance curve conversion formulas ($MQ135$), and speed of sound compensation metrics ($HC\text{-}SR04$).WP7 (Interdependence Solutions): Mitigated I2C bus sharing blocks and peripheral race conditions by writing strict programmatic guardrails, verification fallbacks, and prioritized error isolation structures.👥 Contributors (Group 8)Muhammad Muneeb Javed - BSCE23017Hassan Subhani - BSCE23034Taimoor Shahzaman - BSCE23046📚 References & DatasheetsSTMicroelectronics STM32F4 Reference ManualDHT11 Driver ReferenceHC-SR04 Input Capture InterfacingMQ135 Gas Calculation PrinciplesSSD1306 STM32 OLED Library Engine
---

## 4. `LICENSE` File
It's always recommended to add an open-source license. The **MIT License** is a popular and flexible standard for student projects. Create a file named `LICENSE` and paste the text below:

```text
MIT License

Copyright (c) 2026 Group 8: Muhammad Muneeb Javed, Hassan Subhani, Taimoor Shahzaman

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
