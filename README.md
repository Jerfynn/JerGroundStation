# JerGroundStation (JerGS)

JerGS is a professional, cross-platform Ground Control Station (GCS) for Unmanned Aerial Systems (UAS) built with Flutter and MAVLink.

## Features

*   **Real-time Telemetry**: Monitor attitude, GPS, battery, altitude, and velocity.
*   **Tactical UI**: Dark mode interface designed for high-contrast visibility.
*   **Vehicle Setup**:
    *   **Motor Test**: Spin motors individually to verify direction and mapping.
    *   **Calibration**: Calibrate Accelerometer, Gyroscope, Compass, and Level Horizon.
    *   **Parameters**: View, search, and edit vehicle parameters.
    *   **Safety**: Configure failsafes and geofences.
*   **Connectivity**:
    *   **UDP**: Auto-connect or manual IP/Port for WiFi/Ethernet links.
    *   **Serial**: USB/Telemetry Radio support (Windows/Linux/macOS).
*   **Mission Planning**: Waypoint management and mission upload/download.

## Getting Started

### Prerequisites

*   [Flutter SDK](https://flutter.dev/docs/get-started/install) (>= 3.10.4)
*   **Windows**: Visual Studio 2022 with "Desktop development with C++" workload.
*   **Linux**: `libserialport-dev`, `pkg-config`, `gtk+-3.0`.

### Installation

1.  Clone the repository:
    ```bash
    git clone https://github.com/yourusername/JerGroundStation.git
    cd JerGroundStation
    ```

2.  Install dependencies:
    ```bash
    flutter pub get
    ```

3.  Run the application:
    ```bash
    # Windows
    flutter run -d windows
    ```

## Project Structure

See PROJECT_STRUCTURE.md for a detailed breakdown of the architecture.

## Documentation

Detailed technical documentation can be found in the `docs` folder:
- [🚀 Architecture Overview](docs/ARCHITECTURE.md)
- [🏗️ Detailed Project Structure](docs/PROJECT_STRUCTURE.md)
- [🛠️ Build & Release Process](docs/build_process.md)

## Built With

*   **Flutter & Dart**: UI and logic.
*   **Riverpod**: State management.
*   **flutter_libserialport**: Serial communication.
*   **dart_mavlink**: MAVLink protocol handling.
*   **flutter_map**: Mapping and mission planning.