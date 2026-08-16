# Android Automotive Infotainment & Vehicle Telemetry Simulator

A portfolio-sized Android Automotive OS infotainment prototype paired with a Linux C++ telemetry simulator. The project demonstrates MVVM/Clean Architecture, mixed Kotlin/Java, SocketCAN, a TCP vehicle-data boundary, JUnit, Espresso, Gradle and GitHub Actions without pretending to have production OEM/VHAL access.

## What it does

The Linux simulator generates four mock CAN messages (speed, battery, ambient temperature and status flags), publishes them to a virtual SocketCAN interface (`vcan0`), and mirrors the same frames over TCP. The Android app connects to the simulator, validates/maps the CAN messages, and renders vehicle status alongside mock media controls, system notifications and simulated climate controls.

```text
C++ TelemetryGenerator -> vcan0 -> candump
                      \
                       -> TCP :5555 -> Android service -> repository -> ViewModel -> UI
```

## Repository structure

```text
app/                    Android AAOS-oriented app (Kotlin + Java)
vehicle-simulator/      Linux C++17 SocketCAN/TCP simulator
scripts/                vcan setup and simulator build/run helpers
docs/                   requirements, architecture, ADR, test plan, resume mapping
.github/workflows/      Android and C++ CI
```

## Prerequisites

### Android side

- Android Studio with JDK 17 configured for Gradle.
- Android SDK Platform 36 (the app compiles/targets API 36).
- The included `gradlew` / `gradlew.bat` bootstrap Gradle 8.13 from the official distribution.
- An Android Automotive OS AVD is recommended for the resume/demo recording. In Device Manager choose an **Automotive** hardware profile and a stable Automotive system image (API 35 or newer is fine).

### Linux simulator side

Native SocketCAN requires Linux. Ubuntu/Debian example:

```bash
sudo apt update
sudo apt install -y build-essential cmake can-utils iproute2
```

Windows users should run the simulator in WSL2/Linux and run Android Studio on Windows. macOS users can use a small Linux VM for the SocketCAN portion. The simulator also has `--tcp-only`, but that mode does not demonstrate SocketCAN.

## Run the complete demo

### 1. Create the virtual CAN bus

From the repository root on Linux:

```bash
./scripts/setup_vcan.sh
```

You can verify it with:

```bash
ip -details link show vcan0
```

### 2. Build and test the C++ simulator

```bash
./scripts/build_simulator.sh
```

This runs CMake, builds the C++17 code, and executes the small CTest suite.

### 3. Start the simulator

```bash
./scripts/run_simulator.sh
```

It listens on TCP port `5555`, continuously prints frames such as `100#017C`, and publishes them on `vcan0`.

In a second Linux terminal, prove the CAN traffic exists:

```bash
candump vcan0
```

### 4. Start an Android Automotive OS emulator

In Android Studio:

1. Open **Tools → Device Manager**.
2. Create a virtual device using an **Automotive** hardware profile.
3. Select **Android Automotive with Google APIs x86_64, API 35-ext15** (or a newer stable Automotive image available in your Android Studio).
4. The **Automotive (1080p landscape)** hardware profile is a simple fit for this UI.
5. Boot the AVD.

### 5. Bridge Windows/WSL2 telemetry to the emulator

When Android Studio runs on Windows and the simulator runs inside WSL2, do not assume the emulator's `10.0.2.2` address automatically reaches the Linux process. This repository uses an explicit development bridge instead.

Keep the WSL2 simulator running, then open **Windows PowerShell** from the repository root and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\windows\prepare_android_telemetry.ps1
```

The script verifies that Windows can reach WSL2 on `localhost:5555` and then configures `adb reverse tcp:5555 tcp:5555`. The Android app therefore defaults to `127.0.0.1:5555` on the emulator.

See `docs/WINDOWS_WSL2_NETWORKING.md` for diagnostics and the `netsh portproxy` fallback if Windows localhost forwarding is unavailable.

### 6. Run the Android app

Open this repository in Android Studio, allow Gradle sync to finish, select the Automotive AVD, then run the `app` configuration.

Expected behavior:

- connection changes to **Connected**;
- Logcat under tag `TelemetryTcpClient` shows raw lines such as `RX 100#017C`;
- speed, battery and ambient temperature update roughly every 500 ms;
- door/charging/warning flags occasionally change;
- media previous/play/next buttons modify local mock state;
- climate `−`/`+` changes the target by 0.5°C and clamps it to 16–30°C;
- malformed frames or TCP failures appear in the error banner, and **Reconnect** retries the stream.

## Command-line Android build/test

With Android SDK configured in `ANDROID_HOME` or `local.properties`:

```bash
./gradlew assembleDebug
./gradlew testDebugUnitTest
./gradlew lintDebug
```

With an emulator/device running:

```bash
./gradlew connectedDebugAndroidTest
```

The Espresso tests start an in-process TCP fixture on `127.0.0.1`, so they do not require the C++ simulator.

## Run with a different telemetry host

The default development endpoint is `127.0.0.1:5555` and is intended to be paired with `adb reverse`. For diagnostics, a physical test device, or a simulator on another machine, launch the activity with extras:

```bash
adb shell am start \
  -n com.example.autotelemetry/.ui.MainActivity \
  --es telemetry_host 192.168.1.50 \
  --ei telemetry_port 5555
```

Make sure routing/firewall rules permit the TCP connection.

## Protocol

| ID | Payload | Meaning |
|---|---|---|
| `0x100` | 2 bytes | speed, unsigned big-endian, 0.1 km/h |
| `0x101` | 1 byte | battery percentage |
| `0x102` | 2 bytes | signed big-endian ambient temperature, 0.1°C |
| `0x103` | 1 byte | bit 0 door open, bit 1 charging, bit 2 warning |

Example: `100#04D2` = raw `1234` = **123.4 km/h**.

## Testing and TDD

- `CanFrameParserTest`: valid and invalid CAN payloads.
- `MainViewModelTest`: repository → UI state transitions and local controls.
- `ClimateControllerTest`: validation/clamping business rules.
- `MainActivityTest`: Espresso workflow with real local TCP lines and an error case.
- `simulator_tests`: C++ frame-generation sanity checks.

GitHub Actions runs Android build/lint/JUnit, emulator-based Espresso validation, and a separate C++ CMake/CTest job.

## Important scope note

This is an **infotainment/telemetry simulator**, not a production vehicle integration. It deliberately does not request privileged AAOS car permissions or claim direct CAN access from an Android app. In a production architecture, an OEM vehicle service/VHAL or gateway would expose approved vehicle properties; the TCP boundary here stands in for that gateway and lets the project demonstrate the software architecture safely and reproducibly.

See `docs/RESUME_MAPPING.md` for a one-to-one mapping from the resume bullets to code and tests.
