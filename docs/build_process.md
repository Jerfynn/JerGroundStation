# Build Process for JerGroundStation

This document outlines the steps required to build and release JerGS for Windows.

## Prerequisites

Before building, ensure you have the following installed:

1.  **Flutter SDK**: Version 3.10.4 or higher.
    *   [Install Flutter](https://docs.flutter.dev/get-started/install/windows)
2.  **Visual Studio 2022**:
    *   Download Community Edition (free).
    *   During installation, select the **"Desktop development with C++"** workload.
    *   Ensure the **MSVC v143 - VS 2022 C++ x64/x86 build tools** component is checked.
3.  **Git**: For version control.

## Development Build

To run the application in debug mode (with hot reload):

1.  Open a terminal in the project root.
2.  Get dependencies:
    ```powershell
    flutter pub get
    ```
3.  Run code generation (for Riverpod/Freezed):
    ```powershell
    dart run build_runner build -d
    ```
4.  Run the app:
    ```powershell
    flutter run -d windows
    ```

## Release Build

To create an optimized release build for distribution:

1.  Clean previous builds:
    ```powershell
    flutter clean
    flutter pub get
    ```
2.  Build the Windows executable:
    ```powershell
    flutter build windows --release
    ```

### Output Location
The built files will be located at:
`build\windows\runner\Release\`

You will see:
*   `JerGroundStation.exe`
*   `flutter_windows.dll`
*   `data/` folder

### Distribution
To distribute the app, you must copy the **entire contents** of the `Release` folder, not just the `.exe`. It is recommended to zip this folder or use an installer creator like **Inno Setup**.