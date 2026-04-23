# C2 Stealth Toolkit: "Halo-Persistent"

![C2 Architecture](https://img.shields.io/badge/Architecture-Parent--Child-blue)
![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey)
![Security](https://img.shields.io/badge/Purpose-Educational%20Training-red)

A lightweight, dual-process C2 (Command & Control) framework developed for cybersecurity training environments. This toolkit demonstrates realistic malware techniques including stealth execution, parent-child process persistence, and automated data exfiltration.

> **Note:** This project is a Work-in-Progress (v0.1 Beta). Current development focuses on establishing stable beaconing and stealth persistence.

---

## 🛠 Architecture & Components

The system utilizes a **Parent-Child process model** to ensure modularity and reduce the forensic footprint of individual binaries.

| Component | Bin Size | Main Dependencies | Subsystem | Role |
| :--- | :--- | :--- | :--- | :--- |
| `pure_no_exit32.exe` | ~12KB | `kernel32`, `user32` | Windows (GUI) | Global Key Capture & Persistence |
| `halo_persistent32.exe`| ~18KB | `wininet`, `kernel32` | Windows (GUI) | C2 Beacon & HTTP Exfiltration |

---

## 🚀 Technical Specifications

### 1. The Core Logger (`pure_no_exit32.exe`)
* **Capture Engine:** Implements global input monitoring via `GetAsyncKeyState()`.
* **Stealth Subsystem:** Compiled for the `-mwindows` subsystem to run without a console window or taskbar icon.
* **Process Management:** Triggers the spawning of the C2 beacon upon the first `ENTER` keypress.
* **Integrity:** Features automated buffer management, flushing logs to `C:\Temp\log.log` at 255-character intervals to prevent data loss.

### 2. The C2 Beacon (`halo_persistent32.exe`)
* **Exfiltration Logic:** Periodically reads local log artifacts and transmits data via HTTP POST.
* **Heartbeat/Timing:** Configured for 5-second beacon intervals.
* **Watchdog Logic:** Parent-aware design; the beacon automatically terminates if the primary logger process is killed, minimizing "orphan" process detection.
* **Native Net-Stack:** Leverages `WinINet` for a minimal footprint, avoiding the need for third-party sockets or libraries.

---

## 🏗 Build Instructions

### Cross-Compilation (Linux to Windows)
The toolkit is optimized for the `MinGW-w64` toolchain.

**1. Install Dependencies (Ubuntu/Debian):**
```bash
sudo apt update && sudo apt install gcc-mingw-w64-i686 nasm-mingw-w64
