# Vital Signs Monitor

Windows desktop app that displays live heart rate and SpO2 from an ESP32 + MAX30102, raises alarms, and logs data.

> **Not a medical device.** Portfolio/educational project. Synthetic data only; never use with real patients or for clinical decisions.

See [spec.md](spec.md) for the design.

## Development setup (Windows)

Develop on Windows: the WPF app only builds and runs there, and the ESP32 shows up as a normal COM port (WSL can't see USB devices without extra tooling).

### 1. Install tools
- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Git for Windows](https://git-scm.com/download/win)
- [VS Code](https://code.visualstudio.com/) with extensions:
  - **C# Dev Kit** (Microsoft)
  - **PlatformIO IDE** (installs the ESP32 toolchain on first use)
- USB-serial driver for your ESP32 board (check the chip next to the USB port):
  - CP2102: [Silicon Labs CP210x driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)
  - CH340: [WCH CH340 driver](https://www.wch-ic.com/downloads/CH341SER_EXE.html)

Check in a new terminal:
```
dotnet --list-sdks   # shows 8.0.x
git --version
```

### 2. Get the code
Clone or copy the repo to a normal Windows path, e.g. `C:\Users\<you>\projects\Vitals`, and open that folder in VS Code.

### 3. Build and test the app
```
dotnet build VitalSigns.sln
dotnet test
dotnet run --project src/VitalSigns.App
```

### 4. Firmware
1. Plug in the ESP32 with a data-capable USB cable. It should appear in Device Manager under **Ports (COM & LPT)**.
2. In VS Code, open the `firmware/` folder with PlatformIO (PlatformIO Home → Open Project).
3. Use the PlatformIO toolbar: **Build**, then **Upload**. If upload stalls at "Connecting...", hold the board's **BOOT** button until it starts.
4. **Serial Monitor** runs at 115200 baud. Close it before running the app — only one program can open the COM port.
