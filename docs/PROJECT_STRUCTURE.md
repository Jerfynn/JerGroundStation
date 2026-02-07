# JerGS Professional Project Structure

## Directory Organization

```
lib/
├── core/                          # Low-level protocol & network layer
│   ├── mavlink/
│   │   └── mavlink_parser.dart   # MAVLink frame parsing & decoding
│   └── network/
│       └── network_connection.dart # UDP/Serial/TCP connection handlers
│
├── models/                         # Data structures (Freezed immutable classes)
│   └── telemetry_models.dart      # Complete data models for all telemetry
│
├── providers/                      # State management (Riverpod)
│   ├── telemetry_provider.dart    # Real-time data streams
│   └── connection_provider.dart   # Connection state & actions
│   └── vehicle_provider.dart      # Vehicle state (Parameters, Calibration)
│
├── services/                       # High-level business logic
│   ├── connection/
│   │   └── mavlink_connection_service.dart  # Protocol-agnostic connection manager
│   └── mission/
│       └── mission_service.dart   # Mission planning & waypoint management
│
├── theme/                          # UI styling & theming
│   └── tactical_theme.dart        # Dark tactical color scheme
│
├── ui/
│   ├── screens/                    # Full-page views (Fly, Plan, Setup, Analyze)
│   │   ├── fly_screen.dart
│   │   ├── plan_screen.dart
│   │   ├── motors_screen.dart
│   │   ├── setup_screen.dart
│   │   └── analyze_screen.dart
│   │   └── setup/                  # Setup sub-screens
│   │       ├── calibration_screen.dart
│   │       ├── firmware_screen.dart
│   │       ├── parameters_screen.dart
│   │       └── safety_screen.dart
│   │
│   ├── widgets/
│   │   ├── hud/                    # Head-Up Display components
│   │   │   └── artificial_horizon.dart   # Pitch-roll indicator
│   │   │
│   │   ├── gauges/                 # Instrument gauges
│   │   │   ├── altitude_gauge.dart
│   │   │   ├── speed_gauge.dart
│   │   │   └── compass.dart
│   │   │
│   │   ├── map/                    # Mapping widgets
│   │   │   ├── flight_map.dart
│   │   │   └── mission_map.dart
│   │   │
│   │   ├── panels/                 # Status & information panels
│   │   │   ├── system_status_panel.dart
│   │   │   ├── telemetry_panel.dart
│   │   │   └── bottom_status_bar.dart
│   │   │
│   │   └── navigation/             # Navigation widgets
│   │       ├── left_sidebar.dart   # View selector
│   │       └── top_toolbar.dart    # Quick actions
│   │
│   └── dashboard_screen.dart       # Main layout orchestrator
│
├── main.dart                       # App entry point & Riverpod initialization
└── pubspec.yaml                    # Dependencies
```

## Layer Responsibilities

### 1. **Core Layer** (`/lib/core`)
**Purpose**: MAC- and network-level abstractions

**MAVLink Parser** (`mavlink/mavlink_parser.dart`):
- Binary frame parsing
- Message type identification
- Payload decoding (Attitude, GPS, Battery, etc.)
- Handles MAVLink v2 format

**Network Connection** (`network/network_connection.dart`):
- UDP socket management
- Serial port (USB) handling
- TCP socket support
- Abstraction via `NetworkConnection` interface

### 2. **Models Layer** (`/lib/models`)
**Purpose**: Type-safe, immutable data structures

**Freezed Models**:
- `TelemetryData` - Complete vehicle state
- `BatteryData` - Power system info
- `GpsData` - Position, satellites, fix quality
- `AttitudeData` - Roll/pitch/yaw angles
- `AltitudeData` - Height above ground/home
- `VelocityData` - NED velocity components
- `SystemStatus` - Flight mode, armed state
- `ConnectionInfo` - Connection statistics

### 3. **Providers Layer** (`/lib/providers`)
**Purpose**: State management & reactive data

**Riverpod Providers**:
```dart
// Real-time telemetry stream
telemetryStreamProvider → Stream<TelemetryData>

// Specific data streams
batteryStatusProvider → AsyncValue<BatteryData>
gpsStatusProvider → AsyncValue<GpsData>
attitudeProvider → AsyncValue<AttitudeData>

// Connection management
connectionProvider → StateNotifier<ConnectionInfo>
isConnectedProvider → Provider<bool>
```

### 4. **Services Layer** (`/lib/services`)
**Purpose**: Business logic & protocol handling

**MAVLink Connection Service**:
```dart
• connect(protocol, host, port)
• disconnect()
• sendCommand(MAVLinkCommand)
• telemetryStream accessor
• Packet statistics tracking
```

**Mission Service**:
```dart
• createMission(name, description)
• addWaypoint(name, waypoint)
• validateMission(name)
• cloneMission(source, newName)
• getAllMissions()
```

### 5. **Theme Layer** (`/lib/theme`)
**Purpose**: Centralized styling & color scheme

**Tactical Dark Theme**:
- Background: `#0F0F0F`
- Surfaces: `#1A1A1A`
- Status Colors: Green, Orange, Red, Cyan
- Typography: Consistent font sizes & weights

### 6. **UI Layer** (`/lib/ui`)
**Purpose**: User interface components

#### **Screens** (Full-page views):
- `FlyScreen` - Real-time flight operations & map
- `PlanScreen` - Mission planning & waypoint editor
- `SetupScreen` - Calibration & parameter tuning
- `AnalyzeScreen` - Flight log analysis

#### **Widgets** (Reusable components):

**HUD Widgets** (`/hud`):
- `ArtificialHorizon` - Pitch-roll indicator

**Gauges** (`/gauges`):
- `AltitudeGauge` - Vertical speed & altitude
- `SpeedGauge` - Airspeed indicator
- `Compass` - Heading indicator

**Map Widgets** (`/map`):
- `FlightMap` - Real-time position tracking
- `MissionMap` - Waypoint editor on map

**Panels** (`/panels`):
- `SystemStatusPanel` - Battery, GPS, status indicators
- `TelemetryPanel` - Detailed sensor readouts
- `BottomStatusBar` - System health summary

**Navigation** (`/navigation`):
- `LeftSidebar` - View selector (Fly, Plan, Setup, Analyze)
- `TopToolbar` - Connection status, quick actions

## Data Flow

```
┌─────────────────────────────────────────────────────────┐
│              Vehicle (UAV/Drone)                        │
│         Sends MAVLink messages via UDP/Serial           │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│            NetworkConnection Layer (core/network)       │
│  UDP/RawDatagramSocket or Serial or TCP Socket         │
│  Raw byte streams → Uint8List                          │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│           MAVLinkParser (core/mavlink)                  │
│  Parses binary frames → MAVLinkMessage                  │
│  Decodes payload → BatteryData, GpsData, AttitudeData   │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│    MAVLinkConnectionService (services/connection)       │
│  Aggregates messages → TelemetryData                   │
│  Streams to: telemetryStreamProvider                   │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│          Ripvod Providers (/providers)                  │
│  telemetryStreamProvider                               │
│  batteryStatusProvider                                 │
│  gpsStatusProvider                                     │
│  attitudeProvider                                      │
│ → Caches, notifies subscribers on change              │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│              UI Widgets (/ui/widgets)                   │
│  ConsumerWidget watches providers                       │
│  Rebuilds only when data changes                       │
│  Displays real-time Attitude, Battery, GPS, etc.       │
└─────────────────────────────────────────────────────────┘
```

## Implementation Examples

### 1. **Connect to Vehicle (UDP)**
```dart
final service = MAVLinkConnectionService();
await service.connect(
  protocol: 'UDP',
  host: '192.168.1.100',
  port: 14550,
);
```

### 2. **Display Battery Status**
```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  final battery = ref.watch(batteryStatusProvider);
  
  return battery.whenData((data) {
    return Text('${data.percentage}% @ ${data.voltage}V');
  });
}
```

### 3. **Create & Upload Mission**
```dart
final missionService = MissionService();

// Create mission
final mission = missionService.createMission(
  name: 'Aerial Survey',
  description: 'Site mapping mission',
);

// Add waypoints
missionService.addWaypoint(
  'Aerial Survey',
  Waypoint(
    number: 0,
    latitude: 37.7749,
    longitude: -122.4194,
    altitude: 100,
    autoContinue: true,
    command: WaypointCommand.takeoff,
  ),
);

// Validate & upload
final errors = missionService.validateMission('Aerial Survey');
// Send via: mavlinkService.sendCommand(...)
```

## Key Design Principles

1. **Separation of Concerns**
   - Core handles low-level protocols
   - Services handle business logic
   - UI concerns only display

2. **Reactive Updates**
   - Riverpod watches for changes
   - UI rebuilds automatically
   - No manual setState() calls

3. **Type Safety**
   - Freezed models prevent runtime errors
   - Strong typing across layers

4. **Testability**
   - Providers can be overridden in tests
   - Services accept mocked connections
   - Pure functions for business logic

5. **Scalability**
   - Clear structure for new features
   - Easy to add new message types
   - Performance optimized rendering

## Future Enhancements

### Map Integration
```dart
// /lib/ui/widgets/map/flight_map.dart
- fl_utter_map with OpenStreetMap tiles
- Real-time drone position
- Home location marker
- Telemetry overlay
```

### Auto-Mission Generator
```dart
// /lib/services/mission/auto_mission_generator.dart
- Grid survey missions
- Corridor scan patterns
- Target-based missions
```

### Parameter Tuning UI
```dart
// /lib/ui/screens/parameters_screen.dart
- PID adjustment sliders
- Safety limits enforcement
- Preset management
```

### Flight Log Analysis
```dart
// /lib/services/logging/flight_log_analyzer.dart
- Binary log parsing
- Graph generation
- Statistics calculation
```

## Building & Running

```bash
# Get dependencies
flutter pub get

# Generate code (Freezed, Riverpod)
flutter pub run build_runner build

# Run on Windows
flutter run -d windows

# Run on macOS
flutter run -d macos
```

## Performance Metrics

- **Update Rate**: 10Hz (100ms telemetry)
- **render Time**: < 16ms per frame
- **Memory**: ~150MB (steady state)
- **Network**: UDP preferred for low-latency
