# Resume Bullet Mapping

| Resume claim | Concrete evidence in this repository |
|---|---|
| Kotlin + MVVM + Clean Architecture + OOP | Kotlin UI/ViewModel, domain interfaces/models, repository/service/data classes. |
| Responsive infotainment interfaces | Base layout plus `layout-sw600dp` two-column AAOS layout; media, vehicle, notification and climate cards. |
| ViewModel/repository/service layers | `MainViewModel` → `VehicleRepository` → `TelemetryStreamService`. |
| Linux C++ + SocketCAN | `vehicle-simulator` publishes Linux `can_frame` objects to `vcan0`. |
| Kotlin parsing/data mapping | `CanFrameParser` + `VehicleTelemetryReducer`. |
| TCP Ethernet-style telemetry | `TcpTelemetryServer` ↔ `TelemetryTcpClient`. |
| JUnit | Parser, ViewModel and climate tests. |
| Espresso | Live local TCP fixture exercises telemetry UI and malformed-frame error state. |
| TDD | Deterministic parser/validation/state components and documented test-first workflow. |
| Gradle + GitHub Actions | Gradle Android build and `.github/workflows/android.yml`. |
| Requirements/architecture/test docs | `docs/REQUIREMENTS.md`, `ARCHITECTURE.md`, ADR and `TEST_PLAN.md`. |
| Java | `TelemetryProtocol.java` is used directly by Kotlin parser code. |
