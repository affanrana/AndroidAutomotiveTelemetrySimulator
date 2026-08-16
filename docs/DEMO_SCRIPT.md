# Two-Minute Portfolio Demo

Use this sequence when recording a README GIF/video or showing the project in an interview.

1. **Show architecture quickly.** Open `docs/ARCHITECTURE.md` and point out SocketCAN plus TCP as separate boundaries.
2. **Prove CAN traffic.** Run `candump vcan0` in one terminal.
3. **Start the simulator.** Run `./scripts/run_simulator.sh`; show the same `100`–`103` IDs in both terminals.
4. **Open the AAOS emulator.** Launch the app and show the connection changing to **Connected**.
5. **Show live telemetry.** Point out speed, battery and ambient temperature changing.
6. **Show vehicle state.** Wait for a generated door/charging/warning flag or temporarily modify `TelemetryGenerator.cpp` for a deterministic demo.
7. **Show local infotainment controls.** Use media next/play and climate `+`/`−`.
8. **Show testing evidence.** Run `./gradlew testDebugUnitTest` and `./gradlew connectedDebugAndroidTest`; then show the GitHub Actions workflows.

## Interview explanation

A concise explanation is: “The C++ simulator behaves like a small vehicle gateway. It puts mock frames on Linux SocketCAN so normal CAN tooling can observe them, then mirrors those frames over TCP to model an Ethernet boundary. The Android side treats TCP as transport only: a parser validates the CAN protocol, a repository reduces updates into vehicle state, and the ViewModel exposes immutable UI state.”
