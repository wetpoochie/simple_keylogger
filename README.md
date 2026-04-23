# 🛡️ Endpoint Telemetry Lab: “Halo-Persistent” (Defensive Training)

![Architecture](https://img.shields.io/badge/Architecture-Parent--Child-blue)
![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey)
![Purpose](https://img.shields.io/badge/Purpose-Educational%20Training-green)

A lightweight, dual-process **endpoint telemetry and detection lab** developed for controlled cybersecurity training environments. This project simulates realistic system behaviors such as continuous input handling, parent–child process relationships, and periodic HTTP communication—allowing students to practice **detection, analysis, and forensic investigation**.

> ⚠️ This project is designed for **defensive security training only**. All behavior should be transparent, consent-based, and executed within isolated lab environments.

---

## 🛠 Project Architecture & Component Breakdown

The system utilizes a **Parent–Child process model** to generate correlated telemetry across process, file, and network layers. Both components are compiled for the `Windows (GUI)` subsystem.

| Component | Bin Size | Dependencies | Role |
| :--- | :--- | :--- | :--- |
| **pure_no_exit32.exe** | ~12KB | `kernel32`, `user32` | Telemetry Agent (Input + Logging) |
| **halo_persistent32.exe** | ~18KB | `wininet`, `kernel32` | Network Emitter (HTTP Communication) |

---

## ⚙️ Component Mechanics

### 1. Telemetry Agent (`pure_no_exit32.exe`)

* **Input Monitoring Engine:** Demonstrates high-frequency input polling using standard Windows APIs.
* **Trigger Mechanism:** Buffers input data until a defined event (e.g., `ENTER` key) occurs:
  * Writes buffered data to `C:\Temp\log.log` (append mode)
  * Spawns the Network Emitter process
  * Resets internal buffer and continues operation
* **Overflow Protection:** Automatically writes to disk if buffer exceeds 255 characters to prevent data loss.
* **Execution Model:** Compiled with `-mwindows` (GUI subsystem).

---

### 2. Network Emitter (`halo_persistent32.exe`)

* **Data Transmission:** Periodically reads `C:\Temp\log.log` and sends contents to a configured lab endpoint via HTTP POST.
* **Beacon Timing:** Default interval set to 5 seconds for consistent, observable traffic patterns.
* **Process Awareness:** Monitors parent process; exits automatically if parent terminates (useful for studying process relationships).
* **Networking Stack:** Uses Windows `WinINet` API to simulate standard application-layer traffic.

---

## 🏗 Build & Deployment

### Cross-Compilation (Linux → Windows)

Optimized for the `MinGW-w64` toolchain on Ubuntu/Debian.

### 1. Install Toolchain

```bash
sudo apt update
sudo apt install gcc-mingw-w64-i686 nasm-mingw-w64
