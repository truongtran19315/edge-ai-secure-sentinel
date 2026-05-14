# Intruder-Triggered Room Security Node with Tamper-Proof Storage

A production-grade, multi-protocol embedded vision and secure edge-AI subsystem designed for local space monitoring. This project implements an intelligent, energy-efficient camera node leveraging the STM32N6 platform, FreeRTOS, and Bluetooth/Wi-Fi co-processing to deliver secure event-driven logging coordinated with a high-concurrency Golang backend.

---
## 📋 Functional Use Case

The system acts as an intelligent, autonomous indoor sentinel operating on a dual-factor authentication pipeline to eliminate false alarms:

1. **Edge AI Detection (The Sentinel):** The camera node continuously watches the room in a low-power state. Using an on-chip **People Detection** model running on the NPU, it triggers an event sequence immediately when any human shape or movement enters the space.
2. **BLE Gated Authentication (The Smart Key):** Once a person is detected, the STM32N6 instantly boots its Bluetooth sub-system to scan for the owner's smartphone MAC address/BLE Beacon.
3. **Decision Logic Flows:**
   * **Owner Present (Authorized):** If the owner's phone Bluetooth signal is detected →  The system identifies them as the homeowner →  The alarm is suppressed, no data is recorded, and the system transitions back to low-power mode.
   * **Intruder Detected (Unauthorized):** If no valid BLE signature is found within a specified timeout →  The system declares an active intrusion state →  It captures a burst of evidentiary frames →  Encrypts them immediately via hardware **AES-256** →  Streams the encrypted file over Wi-Fi to the **Golang Backend Server**.

---

## 📌 System Architecture & Operational Pipeline

```text
                     +---------------------------------------+

                     |          STM32N6 Edge Device          |
                     +---------------------------------------+
                                         │
                                         ▼ (DMA)
                                [ Frame Acquisition ]
                                         │
                                         ▼ (Neural-ART NPU)
                             [ AI People Detection ]
                                         │
                        (If Person Detected: Trigger Logic)
                                         │
                                         ▼
                            [ BLE Authentication Scan ]
                                         │
                    ┌────────────────────┴────────────────────┐
                    ▼ (Authorized BLE Key Present)             ▼ (No Valid BLE Key / Intruder)
             [ Owner Home ]                             [ Active Intrusion State ]
                    │                                         │
                    ▼                                         ▼
            [ Suppress Alarm ]                       [ Frame Buffer Lock ]
          (Enter Low-Power Mode)                              │
                                                              ▼ (Hardware CRYP Engine)
                                                    [ AES-256 Encryption ]
                                                              │
                                            ┌─────────────────┴─────────────────┐
                                            ▼ (Local Backup)                    ▼ (LwIP TCP/IP Stack)
                                       [ SD Card Storage ]              [ Wi-Fi Streaming via TLS ]
                                                                                │
                                                                                ▼
                                                                     +---------------------+

                                                                     |  Go Backend Server  |
                                                                     | (Isolated Storage)  |
                                                                     +---------------------+
```

---

## Architectural Highlights & Features

* **Dual-Factor Edge Authentication (AI + BLE):** Minimizes false-alarm metrics by pairing an on-chip AI vision layer with immediate Bluetooth Low Energy proximity gating.
* **On-Device NPU Acceleration:** Executes quantized real-time person detection models utilizing the dedicated STM32N6 Neural-ART hardware engine.
* **Event-Driven Energy Optimization:** Restricts high-frequency peripheral clocking and processing chains, transitioning the MCU into a low-power listening state until awakened by physical vision events.
* **Militarily Secure Local Storage:** Shields evidentiary frames against hardware theft by executing wire-speed AES-256 block encryption via the internal hardware CRYP engine prior to FatFS flash writing.
* **Cryptographic Tamper Prevention:** Appends unique SHA-256 runtime verification hashes and secure hardware-validated timestamp metadata to each captured event.
* **Deterministic RTOS Tasking:** Orchestrates concurrent hardware operations (Camera Pipeline, BLE Scanning, AES Encoding, LwIP Wi-Fi Transmission) using isolated FreeRTOS tasks communicating over thread-safe queues.
* **High-Concurrency Backend Ingestion:** Pairs the edge nodes with a high-throughput asset service written in Golang, utilizing asynchronous goroutines to isolate incoming intrusion streams.

---

## 📂 Repository Structure

```text
├── backend/               # Go Ingestion Backend
│   ├── uploads/           # Target filesystem for decrypted evidence logs
│   └── main.go            # High-concurrency upload handling & network routing
├── firmware/              # STM32N6 Firmware Application
│   ├── Core/              # Application initialization, FreeRTOS Tasks, & BLE logic
│   ├── Drivers/           # Hardware Abstract Layers (HAL) and BSP drivers
│   └── Middlewares/       # Operational stacks (FreeRTOS, LwIP, FatFS)
└── README.md              # Project documentation
```

---

## 🛠️ Tech Stack & Tools

* **Languages:** C, C++, Golang, Markdown
* **Real-Time OS:** FreeRTOS
* **Libraries & SDKs:** STM32CubeHAL, STEdgeAI Suite, LwIP (Lightweight TCP/IP Stack)
* **Hardware Profiles:** STM32N6 Microcontroller, MIPI-CSI Camera Module, Onboard BLE/Wi-Fi Transceivers
* **Development Environments:** STM32CubeIDE, GCC ARM Embedded Toolchain, Go Toolchain

---

## ⚠️ Current Limitations

* **Prototype Bounds:** Configured specifically as an experimental proof-of-concept rather than an enterprise commercial product.
* **BLE Proximity Range Constraints:** Signal gating depends strictly on RSSI approximation thresholds, which can be affected by physical spatial obstructions.
* **Inference Model Boundaries:** Vision detection passes are tuned for lightweight, highly-quantized single-class networks due to local SRAM workspace limits.
