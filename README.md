# SmartLamp: Embedded Environmental Control System

**SmartLamp** is a comprehensive embedded system designed to monitor study environments and automate lighting and ventilation. It integrates analog and digital sensors with real-time actuation, communicating via a custom UART protocol to a PC interface.

This project demonstrates a full-cycle embedded development process: **Requirements Engineering → SysML Architecture → Hardware Design (PCB) → Firmware Implementation.**

---

## Key Features

* **Real-time Environmental Monitoring:**
    * **Noise Level:** Analog sampling via microphone (categorized as Low/Medium/High) using **Timer 0** interrupts.
    * **Air Quality:** CO2 and TVOC monitoring via **iAQ-Core** (I2C).
    * **Lighting:** Ambient light sensing (Lux) via **VEML7700** (I2C).
    * **Climate:** Temperature and Humidity acquisition (Analog Sensors).
* **Intelligent Actuation:**
    * **Smart Lighting:** Addressable RGB LED strip (SK9822) controlled via **Software SPI** (Bit-banging).
    * **Active Cooling:** DC Fan speed control via **Hardware PWM** (Timer 2).
* **Connectivity:**
    * Robust **UART** communication protocol (9600 baud) with CRC checksums for data integrity.
    * Command-based interface for remote control and telemetry.

---

## Tech Stack & Tools

* **Microcontroller:** Microchip PIC16F886 (8-bit, 20MHz External Oscillator).
* **Firmware Development:** C Language, MPLAB X IDE, XC8 Compiler.
* **Hardware Design:** KiCad (Schematics & PCB Layout).
* **System Modeling:** Eclipse Papyrus (SysML for Block/Activity diagrams).
* **Protocols Implemented:** I2C (MSSP), SPI (Bit-banged), UART (EUSART), PWM (CCP1), ADC.

---

## Repository Structure

The project follows a structured engineering workflow separating development artifacts, documentation, and datasheets.

```text
SmartLamp/
├── Desarrollo/                    # Main Development Directory
│   ├── KiCad/                     # Hardware Design Source
│   │   ├── ProyectoSE.kicad_sch      # Electronic Schematics
│   │   └── ProyectoSE.kicad_pcb      # PCB Layout
│   │   
│   ├── MPLAB/                     # Firmware Source Code
│   │   ├── Code/                  # Application Logic
│   │   │   ├── main.c                # Main Loop & Interrupt Service Routines
│   │   │   └── [Test Files].c        # Unit tests for subsystems
│   │   │
│   │   └── Functions/             # Hardware Abstraction Layer (HAL)
│   │       ├── adc-v1.c/h            # Analog-to-Digital Converter Driver
│   │       ├── i2c-v2.c/h            # I2C Protocol Driver (Hardware MSSP)
│   │       ├── pwm-v1.c/h            # PWM Driver (Timer 2 + CCP)
│   │       ├── spi-master-v1.c/h     # SPI Driver (Software Implementation)
│   │       └── uart-v1.c/h           # UART Driver (Circular Buffer logic)
│   │
│   └── Papyrus/                   # System Architecture Models (UML/SysML)
│
├── Imde-comp/                     # PC Interface Software (Component)
├── Hojas técnicas-20251026/       # Component Datasheets (Sensors/MCU)
└── README.md                      # Project Documentation
```

---

## System Architecture

### 1. Firmware Architecture (HAL Approach)
The firmware is designed with modularity in mind. The `main.c` handles the high-level business logic and state management, while the `Functions/` folder contains reusable drivers that abstract the PIC16F886 hardware registers.

* **Interrupt-Driven Sampling:** Noise levels are sampled every 5ms using **Timer 0 (TMR0)** interrupts to ensure non-blocking operation.
* **State Machine:** The UART receiver utilizes a Finite State Machine (FSM) to parse incoming frames byte-by-byte, preventing buffer overflows and handling packet fragmentation.

### 2. Hardware Design (KiCad)
The hardware includes signal conditioning for analog sensors and logic level translation for the communication bus.
* **I2C Bus:** Pull-up resistors configured for 100kHz standard mode.
* **Power:** Regulated 5V rail for logic and separate rail for inductive loads (Fan).

---

## Communication Protocol

Communication between the SmartLamp and the PC is handled via a binary protocol with the following frame structure:

| Header | Length | Command | Payload | CRC (Dummy) |
|:------:|:------:|:-------:|:-------:|:-----------:|
| `0xAA` | 1 Byte | 1 Byte  | N Bytes |   2 Bytes   |

**Command Examples:**
* `0x04`: Set Fan Speed (Payload: 0-100%).
* `0x05`: Set LED Color (Payload: R, G, B, Intensity).
* `0x02`: Read CO2 Level (Response: ppm).

---

## Setup & Installation

### Prerequisites
* **MPLAB X IDE** (v5.xx or higher).
* **XC8 Compiler**.
* **KiCad** (v6.0 or higher) for hardware viewing.

### Build Instructions
1.  Clone the repository:
    ```bash
    git clone [https://github.com/aghidalgo04/SmartLamp.git](https://github.com/aghidalgo04/SmartLamp.git)
    ```
2.  Open **MPLAB X IDE** and import the project from `Desarrollo/MPLAB`.
3.  Ensure the device is set to **PIC16F886** and the toolchain to **XC8**.
4.  Build the project (`Production > Build Project`).
5.  Flash the `.hex` file using a **PICkit 3/4** programmer.
