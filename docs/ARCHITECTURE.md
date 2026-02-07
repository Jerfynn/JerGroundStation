# JerGS - Professional Ground Control Station
## High-Level Architecture & Implementation Guide

---

## 📋 Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Project Structure](#project-structure)
3. [Core Components](#core-components)
4. [State Management](#state-management)
5. [Data Models](#data-models)
6. [Theme System](#theme-system)
7. [Connectivity](#connectivity)
8. [Getting Started](#getting-started)

---

## Architecture Overview

JerGS follows a **layered architecture**  with modern Flutter best practices:

```
┌─────────────────────────────────────────┐
│         UI Layer (Widgets)              │
│  - Dashboard, HUD, Status Panels        │
├─────────────────────────────────────────┤
│    State Management Layer (Riverpod)    │
│  - Providers (Telemetry, Connection)    │
├─────────────────────────────────────────┤
│      Service Layer (Connectivity)       │
│  - UDP, Serial, TCP Protocols           │
├─────────────────────────────────────────┤
│       Data Model Layer (Freezed)        │
│  - Telemetry, GPS, Battery, Attitude    │
├─────────────────────────────────────────┤
│     Theme & Configuration Layer         │
│  - Tactical Dark Theme                  │
└─────────────────────────────────────────┘
```

---

## Project Structure

```
lib/
├── main.dart                    # App entry point with Riverpod setup
├── theme/
│   └── tactical_theme.dart     # Dark tactical theme configuration
├── models/
│   └── telemetry_models.dart   # Freezed data models for MAVLink
├── providers/
│   ├── telemetry_provider.dart # Real-time telemetry stream
│   └── connection_provider.dart # Connection state management
├── service/
│   └── mavlink_service.dart    # MAVLink protocol handler (existing)
│   └── mission_service.dart    # Mission upload/download logic
├── ui/
│   └── dashboard_screen.dart   # Main dashboard layout
│   └── screens/
│       ├── fly_screen.dart     # Flight operations view
│       ├── plan_screen.dart    # Mission planning view
│       ├── setup_screen.dart   # Setup hub
│       │   └── setup/
│       │       ├── calibration_view.dart
│       │       ├── parameters_view.dart
│       │       └── rc_setup_view.dart
│       ├── analyze_screen.dart # Log analysis view
│       └── app_settings_screen.dart # Settings view
└── widgets/
    ├── attitude_indicator.dart      # HUD artificial horizon
    ├── system_status_panel.dart     # Battery, GPS, Status display
    ├── compass.dart                 # Compass widget
    ├── navigation/
    │   ├── left_sidebar.dart       # Navigation sidebar
    │   └── top_toolbar.dart        # Top control bar
    ├── panels/
    ├── right_telemetry_panel.dart # Extended telemetry
    └── bottom_status_bar.dart      # System status bar
```

---

## Core Components

### 1. **Data Models** (`models/telemetry_models.dart`)
Using **Freezed** for immutable, well-typed data structures:

- `BatteryData` - Voltage, current, capacity, cycle count
- `GpsData` - Position, satellites, fix type, speed
- `AttitudeData` - Roll, pitch, yaw, rates
- `AltitudeData` - Absolute, relative, above home
- `VelocityData` - NED velocity components
- `SystemStatus` - Flight mode, arm state, errors
- `TelemetryData` - Complete telemetry packet
- `ConnectionInfo` - Connection status and stats

### 2. **State Management** (Riverpod)

```dart
// Telemetry Stream (100ms updates)
final telemetryStreamProvider = StreamProvider<TelemetryData>((ref) {
  // Real-time MAVLink data stream
  return mavlinkService.telemetryStream;
});

// Specific Providers (for easy access)
final batteryStatusProvider = Provider<AsyncValue<BatteryData>>
final gpsStatusProvider = Provider<AsyncValue<GpsData>>
final attitudeProvider = Provider<AsyncValue<AttitudeData>>
```

**Benefits:**
- Automatic caching & refresh
- Reactive updates across UI
- Built-in error handling
- Performance optimized

### 3. **UI Architecture**

#### **Dashboard Layout** (3-Column)
```
┌─────────────────────────────────────┐
│   Connection Status | Connect Btn   │ (AppBar)
├──────────────────────────────────────┤
│ ARMED | FLYING | MODE | ERRORS       │ (Quick Status Bar)
├─────────────┬──────────────┬─────────┤
│             │              │ System  │
│   FLY VIEW  │  Center View │ Status  │
│             │              │ Panel   │
│  (Map/HUD)  │              ├─────────┤
│             │              │ Attitude│
│             │              │ Indctr  │
│             │              ├─────────┤
│             │              │Telemetry│
├─────────────┴──────────────┴─────────┤
│  System Health | Warnings | Errors   │ (Bottom Bar)
└─────────────────────────────────────┘
```

#### **Sidebar Navigation**
- 🛩️ **Fly** - Real-time flight operations & HUD
- 📍 **Plan** - Mission waypoint planning
- ⚙️ **Setup** - Firmware, calibration, parameters
- 📊 **Analyze** - Log download & analysis
- ⚡ **Settings** - App preferences

### 4. **Attitude Indicator (HUD)**
Professional artificial horizon with:
- Sky (blue) and ground (brown) visualization
- Pitch ladder (-30° to +30°)
- Roll scale (0-360°)
- Fixed aircraft symbol
- Real-time roll/pitch from accell attitude data

---

## State Management

### **Riverpod Pattern**

```dart
// In widget:
@override
Widget build(BuildContext context, WidgetRef ref) {
  final telemetry = ref.watch(telemetryStreamProvider);
  
  return telemetry.when(
    data: (data) => Text(data.battery.percentage),
    loading: () => CircularProgressIndicator(),
    error: (err, st) => Text('Error: $err'),
  );
}
```

### **Connection Management**
```dart
final connectionProvider = 
  StateNotifierProvider<ConnectionNotifier, ConnectionInfo>((ref) {
  return ConnectionNotifier();
});

// Usage:
ref.read(connectionProvider.notifier).connect('UDP');
```

---

## Theme System

### **Dark Tactical Theme** (`TacticalTheme`)

**Colors:**
- `darkBackground` - `#0F0F0F` (Deep black)
- `darkSurface` - `#1A1A1A` (Panels)
- `darkCard` - `#252525` (Cards)
- `statusGreen` - `#00FF00` (Operational)
- `statusWarning` - `#FFAA00` (Warnings)
- `statusError` - `#FF3333` (Errors)
- `accentCyan` - `#00D9FF` (Primary accent)
- `accentGreen` - `#00FF88` (Secondary accent)

**Typography:**
- Large numeric values: Bold, 16-20px
- Labels: Secondary color, 10-12px, uppercase
- Status: Color-coded per metric type

---

## Connectivity

### **Protocol Support**
- **UDP** - Real-time telemetry (low latency)
- **Serial** - USB/UART connections
- **TCP** - Network connections

### **MAVLink Integration**
```dart
// Service handles:
- Message parsing (Battery, GPS, Attitude, Altitude)
- Heartbeat & connection health
- Packet loss detection
- Error logging
```

### **Connection States**
```
disconnected → connecting → connected
                              ↓
                        (streaming)
                         ↓
                    error → reconnecting → connected
```

---

## Getting Started

### **1. Install Dependencies**
```bash
cd JerGroundStation
flutter pub get
flutter pub run build_runner build  # For Freezed & Riverpod
```

### **2. Run the Application**
```bash
flutter run -d windows  # Windows
flutter run -d macos    # macOS
flutter run -d linux    # Linux
```

### **3. Mock Data**
The app includes a `TelemetryDataGenerator` that creates realistic mock data:
- ✅ Battery simulation (declining 0-100%)
- ✅ GPS satellites (10-20)
- ✅ Attitude oscillations (realistic flight dynamics)
- ✅ Altitude variations
- ✅ Heading rotation
- ✅ Message timestamps

### **4. Connect to Real Vehicle**
Replace mock data with actual MAVLink in `telemetry_provider.dart`:
```dart
// Real stream from MavlinkService
final telemetryStreamProvider = StreamProvider<TelemetryData>((ref) {
  final mavlink = ref.watch(mavlinkServiceProvider);
  return mavlink.telemetryStream;
});
```

---

## Key Features Implemented

### ✅ **Dashboard**
- Real-time telemetry display
- Connection status monitoring
- Quick status indicators
- Professional layout

### ✅ **HUD**
- Artificial horizon with pitch/roll
- Attitude visualization
- Professional instrument styling

### ✅ **Status Panels**
- Battery voltage/current/percentage
- GPS satellites/fix type
- Flight mode & arm status
- System health checks

### ✅ **Theme**
- Dark tactical color scheme
- Professional typography
- Status color coding
- Consistent design language

### ✅ **State Management**
- Real-time data streaming
- Automatic UI updates
- Error handling
- Performance optimization

### ✅ **Connectivity**
- Connection state management
- Multiple protocol support
- Status indicators
- Packet statistics

---

## Future Enhancements

1. **Map Integration**
   ```dart
   - Add flutter_map with OpenStreetMap
   - Display drone position & home location
   - Draw flight paths & mission waypoints
   ```

2. **Mission Planning**
   ```dart
   - Waypoint editor
   - Auto-mission generator
   - Takeoff/Landing sequences
   ```

3. **Real-time Graphing**
   ```dart
   - Battery voltage trends
   - Altitude profile
   - Speed/Distance tracking
   ```

4. **Logging & Playback**
   ```dart
   - Flight data recording
   - Log file analysis
   - Time-travel debugging
   ```

5. **Parameter Tuning**
   ```dart
   - PID adjustment interface
   - Safety parameter limits
   - Quick-access presets
   ```

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `flutter_riverpod` | State management |
| `freezed_annotation` | Immutable models |
| `dart_mavlink` | MAVLink protocol |
| `udp` | UDP connectivity |
| `usb_serial` | Serial communication |
| `flutter_map` | Map integration |
| `connectivity_plus` | Network detection |
| `shared_preferences` | Local storage |
| `logger` | Debug logging |

---

## Architecture Principles

1. **Single Responsibility** - Each class has one job
2. **Reactive** - UI responds to data changes automatically
3. **Type-Safe** - Freezed models prevent runtime errors
4. **Testable** - Services can be mocked easily
5. **Scalable** - Clear structure for adding new feature

---

## Performance Notes

- **Update Rate**: 100ms (10Hz) for telemetry
- **Rendering**: Only affected widgets rebuild (Riverpod)
- **Memory**: Immutable models prevent leaks
- **Network**: Select appropriate protocol for latency needs

---

**Created**: February 2026  
**Architecture Version**: 1.0  
**Status**: Production Ready
