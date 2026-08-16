# Android Automotive Infotainment & Vehicle Telemetry Simulator

A resume sized Android Automotive OS infotainment prototype paired with a Linux C++ vehicle telemetry simulator.

The project demonstrates Android application architecture, vehicle telemetry parsing, TCP communication, Linux SocketCAN simulation, automated testing, and CI without claiming production OEM integration or privileged access to real vehicle hardware.

## Project Overview

This project simulates a simple vehicle telemetry pipeline.

A Linux C++ program generates mock CAN style frames representing:

1. Vehicle speed
2. Battery percentage
3. Ambient temperature
4. Vehicle and system status flags

The simulator can publish the frames through Linux SocketCAN when a virtual CAN interface is available.

The same telemetry is also sent through TCP so an Android Automotive application can consume it.

The Android application receives the messages, validates and parses them, maps them into application state, and displays the resulting values in an infotainment style interface.

## Architecture

```text
Linux / C++ Vehicle Simulator
        |
        | SocketCAN
        v
      vcan0
        |
        | visible with candump
        |
        +---------------------------+
        |
        | TCP telemetry
        v
Android Automotive Application
        |
        v
TelemetryTcpClient
        |
        v
TelemetryStreamService
        |
        v
VehicleRepository
        |
        v
MainViewModel
        |
        v
StateFlow UI State
        |
        v
Infotainment UI
```

The Android application does not directly read a vehicle CAN interface.

The TCP boundary represents the type of gateway or vehicle service boundary that would normally exist between an infotainment application and lower level vehicle systems.

This keeps the project realistic for a personal portfolio project without pretending to implement proprietary OEM APIs or privileged Android Automotive VHAL integration.

## Features

### Android Automotive Application

The Android side includes:

1. Kotlin based Android application
2. Android Automotive oriented interface
3. MVVM architecture
4. Clean Architecture style separation
5. Repository layer
6. Telemetry service layer
7. TCP telemetry client
8. CAN message parser
9. Vehicle data mapping
10. StateFlow based UI state
11. Vehicle speed display
12. Battery percentage display
13. Ambient temperature display
14. Vehicle and system status display
15. Mock media controls
16. Simulated climate controls
17. Warning and error states
18. Reconnection support
19. Java and Kotlin interoperability
20. JUnit tests
21. Espresso instrumentation tests

### Linux C++ Vehicle Simulator

The simulator includes:

1. C++17 implementation
2. Mock CAN frame generation
3. Linux SocketCAN output
4. Virtual CAN support through `vcan0`
5. TCP telemetry server
6. TCP only development mode
7. CMake build configuration
8. CTest validation

### Automation and Documentation

The repository also contains:

1. Gradle build configuration
2. GitHub Actions workflows
3. Android unit test automation
4. Android instrumentation test automation
5. C++ build and CTest automation
6. Software requirements documentation
7. Architecture documentation
8. Architecture decision records
9. Test planning documentation
10. Requirements to tests mapping
11. Resume bullet mapping
12. Demonstration and interview documentation

## Repository Structure

```text
app/
    Android Automotive application

vehicle-simulator/
    Linux C++ telemetry simulator

scripts/
    Build, simulator, SocketCAN, and development helper scripts

docs/
    Requirements, architecture, ADRs, test plans, networking notes,
    resume mapping, and demonstration documentation

.github/workflows/
    GitHub Actions CI configuration

build.gradle.kts
    Root Android Gradle configuration

settings.gradle.kts
    Gradle project configuration

gradlew
gradlew.bat
    Gradle wrapper
```

## Telemetry Protocol

The simulator produces four message types.

| CAN ID  | Payload | Meaning                                   |
| ------- | ------- | ----------------------------------------- |
| `0x100` | 2 bytes | Vehicle speed in 0.1 km/h units           |
| `0x101` | 1 byte  | Battery percentage                        |
| `0x102` | 2 bytes | Signed ambient temperature in 0.1°C units |
| `0x103` | 1 byte  | Vehicle and system status flags           |

### Example Speed Message

```text
100#04D2
```

The payload is hexadecimal.

```text
0x04D2 = 1234 decimal
1234 / 10 = 123.4 km/h
```

The Android parser therefore converts:

```text
100#04D2
```

into:

```text
123.4 km/h
```

### Status Flags

Message:

```text
103#XX
```

uses individual bits for simulated status information.

```text
Bit 0 = door open
Bit 1 = charging
Bit 2 = warning
```

## Recommended Development Setup

The project was tested using this development environment:

```text
Windows
    |
    + Android Studio
    |
    + Android Automotive Emulator
    |
    + WSL2 Ubuntu
          |
          + C++ Vehicle Simulator
```

The recommended day to day development mode on WSL2 is TCP only mode.

SocketCAN can be demonstrated separately on native Ubuntu or an Ubuntu virtual machine.

## Important WSL2 SocketCAN Limitation

Some standard Microsoft WSL2 kernels do not expose the `vcan` kernel module.

For example, running:

```bash
./scripts/setup_vcan.sh
```

may produce:

```text
modprobe: FATAL: Module vcan not found
```

This is a WSL2 kernel capability issue rather than a problem with the simulator implementation.

Do not rebuild the WSL2 kernel just to develop the Android portion of this project.

Use TCP only mode instead.

```bash
./scripts/run_simulator.sh --tcp-only --port 15556
```

For the final SocketCAN demonstration, use one of the following:

1. Native Ubuntu
2. Ubuntu virtual machine
3. A Linux environment with CAN and VCAN kernel support
4. An optional custom WSL2 kernel with CAN support

The SocketCAN implementation remains part of the project because it represents the Linux vehicle simulation side of the architecture.

# Quick Start on Windows and WSL2

This is the setup that was verified to work with Android Studio on Windows and the C++ simulator running inside WSL2.

## 1. Android Requirements

Install Android Studio.

Configure Gradle to use:

```text
JDK 17
```

Install:

```text
Android SDK Platform 36
Android SDK Platform Tools
Android Emulator
```

The application compiles and targets API 36.

## 2. Create an Android Automotive Emulator

In Android Studio open:

```text
Tools
Device Manager
Create Virtual Device
Automotive
```

A suitable hardware profile is:

```text
Automotive (1080p landscape)
```

A suitable system image is:

```text
Android Automotive with Google APIs
API 35 or newer
x86_64 on Intel or AMD Windows machines
```

Boot the emulator and wait until the Android Automotive interface is fully loaded.

## 3. Start the C++ Simulator in WSL2

Open Ubuntu through WSL2.

Move into the repository.

Example:

```bash
cd "/mnt/c/Users/YOUR_WINDOWS_USERNAME/path/to/AndroidAutomotiveTelemetrySimulator"
```

Start the simulator:

```bash
./scripts/run_simulator.sh --tcp-only --port 15556
```

Expected output includes:

```text
TCP telemetry listening on 0.0.0.0:15556
```

The simulator then continuously produces messages similar to:

```text
100#017C
101#5C
102#00D2
103#00

100#0195
101#5C
102#00D4
103#00
```

Leave this terminal running.

## 4. Prepare ADB in Windows PowerShell

Open Windows PowerShell.

Move into the repository.

Example:

```powershell
cd "C:\Users\YOUR_WINDOWS_USERNAME\path\to\AndroidAutomotiveTelemetrySimulator"
```

Add Android Platform Tools to the current PowerShell session:

```powershell
$env:Path = "$env:LOCALAPPDATA\Android\Sdk\platform-tools;$env:Path"
```

Verify that the emulator is visible:

```powershell
adb devices
```

Expected output resembles:

```text
List of devices attached
emulator-5554   device
```

## 5. Verify Windows Can Reach WSL2

Run:

```powershell
Test-NetConnection 127.0.0.1 -Port 15556
```

Expected result:

```text
TcpTestSucceeded : True
```

If this is `False`, troubleshoot Windows to WSL2 localhost forwarding before troubleshooting the Android application.

## 6. Create the Emulator TCP Bridge

The first Android Emulator commonly uses host port `5555` for ADB communication.

Because of that, this development setup avoids using host port `5555`.

The working configuration is:

```text
Android application port: 15555

Windows and WSL2 simulator port: 15556
```

Create the ADB reverse bridge:

```powershell
adb reverse --remove tcp:15555 2>$null
adb reverse tcp:15555 tcp:15556
adb reverse --list
```

Expected output resembles:

```text
host-17 tcp:15555 tcp:15556
```

The resulting development path is:

```text
Android Application
127.0.0.1:15555
        |
        v
ADB Reverse
        |
        v
Windows localhost:15556
        |
        v
WSL2 localhost forwarding
        |
        v
C++ Simulator:15556
```

## 7. Build and Install the Android Application

Open the repository root in Android Studio.

The folder opened in Android Studio should contain:

```text
settings.gradle.kts
build.gradle.kts
app/
vehicle-simulator/
```

Allow Gradle sync to complete.

Select the running Android Automotive emulator.

Run the:

```text
app
```

configuration.

Android Studio will compile, install, and launch the application.

## 8. Launch the Application With the Development Port

After the application has been installed, run:

```powershell
adb shell am force-stop com.example.autotelemetry
```

Then:

```powershell
adb shell am start -n com.example.autotelemetry/.ui.MainActivity --ei telemetry_port 15555
```

Expected PowerShell output:

```text
Starting: Intent { cmp=com.example.autotelemetry/.ui.MainActivity (has extras) }
```

The Android application should now connect to the telemetry simulator.

## 9. Verify Live Telemetry

The application should display changing values such as:

```text
Speed: 67.4 km/h
Battery: 88%
Ambient: 24.4°C
Vehicle Status: Charging
```

The values should change as new simulator frames arrive.

This demonstrates the complete path:

```text
C++ TelemetryGenerator
        |
        v
TCP Server
        |
        v
TelemetryTcpClient
        |
        v
CanFrameParser
        |
        v
VehicleRepository
        |
        v
MainViewModel
        |
        v
StateFlow
        |
        v
Android Automotive UI
```

## 10. Verify Raw Messages in Logcat

Open Logcat in Android Studio.

Filter using:

```text
TelemetryTcpClient
```

Expected messages resemble:

```text
Connecting to 127.0.0.1:15555
Connected to 127.0.0.1:15555

RX 100#0195
RX 101#5C
RX 102#00D4
RX 103#00
```

This provides a useful demonstration that the exact simulator messages are reaching the Android application.

For example:

```text
WSL simulator:
100#0195

Android Logcat:
RX 100#0195

Parser:
0x0195 = 405 decimal

Mapping:
405 / 10 = 40.5 km/h

UI:
40.5 km/h
```

# Short Copy and Paste Run Instructions

## WSL2 Terminal

```bash
cd "/mnt/c/Users/YOUR_WINDOWS_USERNAME/path/to/AndroidAutomotiveTelemetrySimulator"

./scripts/run_simulator.sh --tcp-only --port 15556
```

Leave this terminal running.

## Windows PowerShell

```powershell
cd "C:\Users\YOUR_WINDOWS_USERNAME\path\to\AndroidAutomotiveTelemetrySimulator"

$env:Path = "$env:LOCALAPPDATA\Android\Sdk\platform-tools;$env:Path"

adb devices

Test-NetConnection 127.0.0.1 -Port 15556

adb reverse --remove tcp:15555 2>$null

adb reverse tcp:15555 tcp:15556

adb reverse --list

adb shell am force-stop com.example.autotelemetry

adb shell am start -n com.example.autotelemetry/.ui.MainActivity --ei telemetry_port 15555
```

# Full SocketCAN Demonstration

The TCP only WSL2 setup demonstrates the Android integration.

To demonstrate the Linux SocketCAN portion, use native Linux or a Linux virtual machine with VCAN support.

## 1. Install Dependencies

Ubuntu or Debian:

```bash
sudo apt update
sudo apt install build-essential cmake can-utils iproute2
```

## 2. Create the Virtual CAN Interface

From the repository root:

```bash
./scripts/setup_vcan.sh
```

Verify:

```bash
ip -details link show vcan0
```

## 3. Start the Simulator

```bash
./scripts/run_simulator.sh
```

The simulator publishes the generated frames through SocketCAN and TCP.

## 4. Observe the CAN Traffic

Open another Linux terminal:

```bash
candump vcan0
```

You should see the generated CAN frames appearing on the virtual interface.

This demonstrates:

```text
C++ TelemetryGenerator
        |
        v
SocketCAN
        |
        v
vcan0
        |
        v
candump
```

# Why Both SocketCAN and TCP Exist

SocketCAN represents the Linux vehicle network simulation.

TCP represents the application communication boundary.

The same simulated telemetry can therefore be observed in two ways:

```text
TelemetryGenerator
        |
        + SocketCAN to vcan0 to candump
        |
        + TCP to Android Automotive application
```

The Android application intentionally does not directly consume Linux SocketCAN traffic.

In a real vehicle, an infotainment application would normally obtain approved vehicle data through platform services, an OEM gateway, middleware, or the Android Automotive Vehicle HAL rather than opening an arbitrary CAN interface directly.

The TCP layer in this project models that separation.

# Android Architecture

## TelemetryTcpClient

Responsible for:

```text
Opening the TCP connection

Reading telemetry lines

Reporting connection changes

Reporting received frames

Handling disconnect and error conditions
```

## TelemetryStreamService

Responsible for exposing the telemetry connection as an application friendly stream.

It separates raw TCP behavior from repository logic.

## VehicleRepository

Responsible for translating telemetry events into application state.

The ViewModel does not need to know how sockets work.

This provides a boundary between data acquisition and presentation logic.

## CanFrameParser

Responsible for validating and parsing messages such as:

```text
100#04D2
```

It checks message formatting, CAN identifiers, payload lengths, hexadecimal values, and telemetry specific rules.

## MainViewModel

Responsible for exposing UI state and handling UI related business behavior.

It does not directly open sockets.

It consumes state from the repository and exposes values to the interface through StateFlow.

## StateFlow

StateFlow is used to represent observable application state.

When telemetry changes:

```text
Repository state changes
        |
        v
ViewModel receives new state
        |
        v
StateFlow emits new UI state
        |
        v
Android UI updates
```

# Why MVVM

MVVM separates user interface code from application state and business logic.

In this project:

```text
View
    Android UI

ViewModel
    MainViewModel

Model and data layer
    Repository
    telemetry service
    TCP client
    parser
```

This makes application logic easier to understand and test without requiring every test to launch the Android interface.

# Clean Architecture Scope

This project uses Clean Architecture principles rather than claiming a large enterprise implementation.

The main separation is:

```text
Presentation
    Activity
    ViewModel
    UI state

Application and domain behavior
    State reducers
    validation
    climate logic

Data
    VehicleRepository

Infrastructure
    TelemetryStreamService
    TelemetryTcpClient
```

Dependencies are kept separated so networking details do not need to be embedded directly in the Android UI.

# Media and Climate Simulation

The Android application also includes local infotainment features.

Media controls simulate actions such as:

```text
Previous
Play
Pause
Next
```

Climate controls allow the user to modify the target temperature.

The climate logic includes validation and clamping behavior so the target remains within the supported range.

These controls are intentionally simulated and do not claim to control a real vehicle.

# Testing

## Android Unit Tests

Run:

```bash
./gradlew testDebugUnitTest
```

On Windows:

```powershell
.\gradlew.bat testDebugUnitTest
```

The unit tests cover areas such as:

```text
CAN frame parsing

Valid telemetry messages

Malformed messages

Payload validation

Vehicle data mapping

ViewModel state changes

Climate control rules

Business logic

Status flag interpretation

Connection related state transitions
```

## Android Instrumentation Tests

With an Android emulator running:

```bash
./gradlew connectedDebugAndroidTest
```

On Windows:

```powershell
.\gradlew.bat connectedDebugAndroidTest
```

The instrumentation suite uses Espresso to validate important Android user interface behavior.

The test fixture can provide local TCP telemetry without requiring the external C++ simulator for every UI test.

This makes the instrumentation tests repeatable and suitable for automation.

## C++ Tests

Build and test the simulator:

```bash
./scripts/build_simulator.sh
```

CTest can also be run from the simulator build directory.

The C++ tests validate core simulator behavior such as telemetry frame generation.

# Build Commands

## Android Debug Build

Linux or macOS:

```bash
./gradlew assembleDebug
```

Windows:

```powershell
.\gradlew.bat assembleDebug
```

## Android Unit Tests

Linux or macOS:

```bash
./gradlew testDebugUnitTest
```

Windows:

```powershell
.\gradlew.bat testDebugUnitTest
```

## Android Lint

Linux or macOS:

```bash
./gradlew lintDebug
```

Windows:

```powershell
.\gradlew.bat lintDebug
```

## Android Instrumentation Tests

Linux or macOS:

```bash
./gradlew connectedDebugAndroidTest
```

Windows:

```powershell
.\gradlew.bat connectedDebugAndroidTest
```

# GitHub Actions

The repository contains GitHub Actions configuration for automated validation.

The CI configuration is intended to demonstrate:

```text
Android application build

Android unit tests

Android lint

Android instrumentation tests using an emulator runner

C++ CMake build

C++ CTest execution
```

Android instrumentation tests require an emulator environment.

The workflow therefore uses emulator based validation rather than assuming a normal Linux runner automatically provides a running Android device.

# Error and Reconnection Behavior

If the TCP connection fails, the application exposes the failure through UI state rather than silently ignoring it.

The architecture supports:

```text
Connecting

Connected

Disconnected

Error

Reconnect
```

This makes connection state observable and testable.

# Running With Another Telemetry Endpoint

The application supports launch extras for telemetry diagnostics.

Example:

```bash
adb shell am start \
  -n com.example.autotelemetry/.ui.MainActivity \
  --es telemetry_host 192.168.1.50 \
  --ei telemetry_port 5555
```

This can be useful for:

```text
Physical Android devices

Another development computer

A Linux VM

Alternative TCP routing tests
```

Routing and firewall rules must allow access to the selected TCP endpoint.

# Stopping the Development Demo

Stop the C++ simulator in WSL2 with:

```text
Ctrl+C
```

Stop the Android application:

```powershell
adb shell am force-stop com.example.autotelemetry
```

Remove the reverse bridge:

```powershell
adb reverse --remove tcp:15555
```

# Requirements and Design Documentation

Additional documentation is available under:

```text
docs/
```

The repository contains documentation covering:

```text
Software requirements

System architecture

Architecture decisions

Networking

Testing strategy

Requirements to tests mapping

Resume bullet mapping

Demonstration notes

Interview preparation
```

# Resume Scope

This project is intended to support resume statements such as:

```text
Developed an Android Automotive infotainment prototype in Kotlin using MVVM, Clean Architecture principles, and object oriented design.

Created responsive infotainment interfaces for media controls, vehicle status, notifications, and simulated climate functionality.

Implemented ViewModel, repository, and service layers to separate interface state, application logic, and vehicle data processing.

Built a Linux based C++ telemetry simulator that produces mock CAN frames through a virtual SocketCAN interface.

Developed Kotlin parsing and mapping components that convert simulated CAN messages into speed, battery, temperature, and system status values.

Added a TCP based telemetry interface to model communication between the simulated vehicle service and infotainment application.

Wrote JUnit tests for message parsing, ViewModel state transitions, validation, and application business logic.

Created Espresso instrumentation tests covering important interface workflows and error states.

Used Gradle and GitHub Actions to automate builds and test validation.

Documented software requirements, architecture decisions, and test cases for the major functional requirements.
```

Every statement should be interpreted within the scope of a personal simulator project.

This repository does not claim production vehicle integration.

# Limitations

This is intentionally a personal portfolio project.

It does not implement:

```text
Production OEM vehicle services

Privileged Android Automotive VHAL access

A physical CAN transceiver

Automotive safety certification

Production cybersecurity infrastructure

Real climate control

Real media hardware integration

Production vehicle diagnostics
```

The goal is to demonstrate software architecture, Android Automotive development, Linux vehicle simulation, communication boundaries, testing, and CI in a project that can reasonably be explained during a technical interview.

# Demonstrated End to End Flow

The verified Windows and WSL2 development setup demonstrates:

```text
C++ vehicle simulator
        |
        v
TCP :15556
        |
        v
Windows and WSL2 localhost forwarding
        |
        v
ADB reverse
        |
        v
Android :15555
        |
        v
TelemetryTcpClient
        |
        v
TelemetryStreamService
        |
        v
VehicleRepository
        |
        v
MainViewModel
        |
        v
StateFlow
        |
        v
Android Automotive infotainment UI
```

Live simulator values can be observed changing in the Android Automotive interface for speed, battery, ambient temperature, and vehicle status.

For detailed requirement and implementation mapping, see:

```text
docs/RESUME_MAPPING.md
```
