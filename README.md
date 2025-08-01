# pico-uart-gpio-hub

> **Multi-client UART-based GPIO control system** for Raspberry Pi Pico, with persistent flash state and automatic UART connection detection.

This project enables a central **Pico-based server** to detect and communicate with multiple **Pico-based clients** over UART. Clients expose GPIOs that can be remotely controlled (ON/OFF/TOGGLE) by the server via UART messages. The server saves the state of all connected devices in its internal Flash memory, allowing full restoration after power cycles.

---

## System Overview

* Server auto-detects all valid TX/RX pin combinations via UART handshake.
* Each client listens for GPIO control commands and applies them.
* Server sends state updates and stores them in persistent Flash.

---

## Features

* Automatic UART handshake
* Scans all Raspberry Pi Pico's UART0 & UART1 pin pairs (up to 5 clients)
* Remote GPIO control
* Server tracks all client states, supports:

  * ON / OFF
  * TOGGLE
  * SAVE active config
  * BUILD preset config
  * LOAD preset into active config
  * 5 PRESET configurations per client
* Persistent flash memory with CRC32 protection
* Menu-based USB CLI interface for live control

---

## How It Works

1. Server powers on and scans UART pin pairs.
2. Client powers on and broadcasts handshake.
3. On success:

   * Server saves connection
   * Server can control client GPIOs
4. States saved to Flash with CRC32.
5. On reboot, states are restored automatically.

---

## UART Protocol

### Handshake

```
Client → Server : "Requesting Connection-[TX,RX]"
Server → Client : "[TX,RX]"
Client → Server : "[Connection Accepted]"
```

### Command Format

```
Server → Client : "[gpio_number,value]"
Example: "[2,1]" → turn GPIO 2 ON
```

---

## Requirements

* Any Raspberry Pi Pico boards (1 server, 1+ clients)
* [Pico SDK](https://github.com/raspberrypi/pico-sdk)
* CMake + Arm GCC toolchain

---

## Toolchain & SDK Setup Guide

### Windows

#### 1. Install ARM GCC

* Download from: [https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
* Install (e.g., `C:\arm-gcc`) and add `C:\arm-gcc\bin` to your PATH

#### 2. Install CMake

* [https://cmake.org/download/](https://cmake.org/download/)
* Add to PATH during installation

#### 3. Install MinGW Make (via MSYS2)

* [https://www.msys2.org](https://www.msys2.org)
* Then in MSYS2 terminal:

  ```bash
  pacman -S mingw-w64-x86_64-make
  ```

#### 4. Clone Pico SDK

```bash
git clone -b master https://github.com/raspberrypi/pico-sdk.git
cd pico-sdk
git submodule update --init
```

___5. SET ENVIRONMENT VARIABLE:___

```bash
set PICO_SDK_PATH=path\to\pico-sdk
```

---

### Linux / macOS

#### 1. Install ARM GCC

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install gcc-arm-none-eabi
```

macOS:

```bash
brew tap ArmMbed/homebrew-formulae
brew install arm-none-eabi-gcc
```

#### 2. Install CMake + Ninja (optional)

```bash
sudo apt install cmake ninja-build
# or
brew install cmake ninja
```

#### 3. Clone Pico SDK

```bash
git clone -b master https://github.com/raspberrypi/pico-sdk.git
cd pico-sdk
git submodule update --init
```

Set environment variable:

```bash
export PICO_SDK_PATH=/full/path/to/pico-sdk
```

---

## How to Build (Multiple Environments)

This project supports multiple build systems using CMake. The `-DPICO_BOARD=pico` flag is used to select the board version. The `CMakeLists.txt` files are configured so that the build outputs flash files for both the server and the client separately, each in its own folder.

### Build with **MinGW Makefiles** (recommended for Windows)

```bash
mkdir build
cd build
cmake -G "MinGW Makefiles" .. -DPICO_BOARD=pico
mingw32-make
```

For multiple board types:

```bash
mkdir build
cd build
mkdir build_pico
cd build_pico
cmake -G "MinGW Makefiles" ../.. -DPICO_BOARD=pico
mingw32-make
cd ..
mkdir build_pico2_w
cd build_pico2_w
cmake -G "MinGW Makefiles" ../.. -DPICO_BOARD=pico2_w
mingw32-make
```

Each board type now has its own build folder, containing both the client and server firmware ready to be flashed.

### Build with **Ninja** (cross-platform, fast builds)

```bash
mkdir build
cd build
cmake -G "Ninja" .. -DPICO_BOARD=pico
ninja
```

### Build with **Unix Makefiles** (Linux/macOS)

```bash
mkdir build
cd build
cmake -G "Unix Makefiles" .. -DPICO_BOARD=pico
make
```

### Build with **Visual Studio 2022** (Windows)

```powershell
mkdir build
cd build
cmake -G "Visual Studio 17 2022" .. -DPICO_BOARD=pico
cmake --build . --config Debug
```

---

## Flashing to Raspberry Pi Pico

### Option 1: Drag-and-drop (BOOTSEL mode)

1. Hold `BOOTSEL` while plugging in the Pico via USB.
2. It will mount as a drive.
3. Drag the `.uf2` file onto it.

### Option 2: Use `picotool`

```bash
picotool load firmware.uf2
```

Requires:

* `picotool` installed
* Pico connected normally (not in BOOTSEL)

---

## Configuration

Edit `types.c` and `config.h` to customize:

* UART pin pairs and instances
* Handshake timeout
* Max GPIOs per client
* Flash memory layout

---

## Design Considerations

* Uses `__not_in_flash_func` for safe Flash writes
* Handshake timeouts are adjustable

---

## Author

**Dragos Hategan**

---

## License

This project is licensed under the [BSD-3-Clause License](https://opensource.org/licenses/BSD-3-Clause).

---

## Author Note

This open-source project was developed by **Dragos Hategan** as part of a professional embedded IoT portfolio.

You are free to use, modify, and redistribute this project, including in commercial or private projects, as long as attribution is maintained. If you build upon this codebase or publish derived works, please consider crediting the original author by name or linking back to this repository.

For freelancing, consulting, or collaborations in embedded systems, feel free to reach out via GitHub or [LinkedIn](https://www.linkedin.com/in/dragos-hategan).
