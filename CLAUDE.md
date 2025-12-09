# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rastec Tracker é o aplicativo móvel oficial Flutter da Rastec Tecnologia para gerenciamento e monitoramento de dispositivos de rastreamento GPS com a plataforma Traccar. O app oferece rastreamento ao vivo, gerenciamento de dispositivos, notificações de eventos e funcionalidades de histórico/relatórios.

**Servidor:** https://tracker.rastecnologia.com.br

**Identificadores:**
- Android Package: `rastec.Android`
- iOS Bundle ID: `br.com.rastecnologia`
- Deep Link Schemes: `rastec.android://` (Android), `br.com.rastecnologia://` (iOS)

**Principais Funcionalidades:**
- Arquitetura baseada em WebView envolvendo interface web Traccar
- Integração Firebase (Analytics, Crashlytics, Messaging)
- Armazenamento seguro de tokens com autenticação biométrica
- Suporte a deep linking para fluxos OAuth e notificações
- Download/compartilhamento de arquivos (XLSX, KML, CSV, GPX)
- Notificações push com tratamento de eventos

**Atribuição:**
Este projeto é baseado no [Traccar Manager](https://github.com/traccar/traccar-manager-flutter) de Anton Tananaev, licenciado sob Apache License 2.0. Modificado e rebrandizado pela Rastec Tecnologia.

## Development Commands

### Setup and Dependencies

```bash
# Install Flutter dependencies
flutter pub get

# Install iOS dependencies (requires CocoaPods)
cd ios && pod install && cd ..

# Clean build artifacts
flutter clean

# Check Flutter environment
flutter doctor -v
```

### Running the App

**iOS:**
```bash
# IMPORTANT: FlutterFire CLI must be in PATH
export PATH="$PATH:$HOME/.pub-cache/bin"

# List iOS simulators
flutter devices

# Run on iOS simulator
flutter run -d ios

# Run on specific iOS device
flutter run -d <device-id>

# Open Xcode workspace (for native debugging)
open ios/Runner.xcworkspace
```

**Android:**
```bash
# List Android emulators
flutter emulators

# Launch specific emulator
flutter emulators --launch <emulator_id>

# Run on Android emulator
flutter run -d android

# Run on specific device
flutter run -d <device-id>
```

### Building

```bash
# Build APK (Android)
flutter build apk --release

# Build iOS (requires Xcode and code signing)
flutter build ios --release

# Build for specific flavor/configuration
flutter build apk --release --target-platform android-arm64
```

### Linting and Analysis

```bash
# Run static analysis
flutter analyze

# Format code
dart format .

# Check for outdated dependencies
flutter pub outdated
```

## Architecture

### Core Structure

The app uses a **hybrid WebView architecture** where most UI and business logic lives in the Traccar web application, with native mobile features bridged through JavaScript channels.

**Main Components:**

1. **MainScreen** (`lib/main_screen.dart`) - The primary screen containing:
   - WebViewController managing the embedded Traccar web interface
   - JavaScript-to-native bridge (`appInterface` channel)
   - Navigation interception for OAuth, downloads, and external links
   - Deep link handling via AppLinks
   - Firebase messaging integration

2. **TokenStore** (`lib/token_store.dart`) - Secure credential storage:
   - Uses FlutterSecureStorage for encrypted token persistence
   - Supports biometric authentication (Face ID/Touch ID) for token access
   - Manages login/logout token lifecycle

3. **ErrorScreen** (`lib/error_screen.dart`) - Fallback UI:
   - Displayed when WebView encounters loading errors
   - Allows server URL reconfiguration

### WebView Bridge Protocol

Communication between native code and web interface uses a pipe-delimited message format via `window.appInterface.postMessage()`:

- `login|<token>` - Save authentication token
- `authentication` - Request stored login token
- `authenticated` - Signal authentication completed
- `logout` - Clear stored token
- `download|<base64>` - Save and share downloaded file
- `server|<url>` - Update server URL and reload

### Key Integrations

**Firebase:**
- Initialized in `main.dart` with Crashlytics error handling
- Push notifications open app to specific event URLs (`/event/{eventId}`)
- Token refresh automatically synced to web interface via `updateNotificationToken()`

**Deep Linking:**
- Custom scheme: `org.traccar.manager://`
- Handles OAuth redirect flows by rewriting redirect_uri
- Processes notification deep links to event pages

**File Downloads:**
- Detects Excel blob creation via JavaScript injection
- Intercepts URLs ending in xlsx/kml/csv/gpx
- Uses Bearer token authentication for file requests
- Shares files via native share sheet

**Permissions:**
- Camera: WebView camera permission mapped to native permission
- Location: Geolocation prompts mapped to native location permission
- Notifications: Firebase messaging permissions requested on auth

### Platform-Specific Configurations

**iOS:**
- Minimum deployment target: iOS 15.0
- Uses WKWebView (webview_flutter_wkwebview)
- CocoaPods manages native dependencies
- FlutterFire requires flutterfire CLI in PATH for build scripts

**Android:**
- Minimum SDK: 21 (defined by Flutter)
- NDK version: 27.0.12077973
- Uses AndroidView for WebView (webview_flutter_android)
- Release signing configured via `../../environment/key.properties` (if exists)
- SharedPreferences backend: native SharedPreferences library

### State Management

The app uses **minimal state management** relying on:
- WebView for UI state (managed by web interface)
- SharedPreferencesWithCache for server URL (key: `url`)
- FlutterSecureStorage for auth token (key: `token`)
- Completers for initialization (`_initialized`) and authentication (`_authenticated`) flow control

### Navigation Flow

1. App launches → Initialize WebView with saved URL (default: https://demo.traccar.org)
2. Check for Firebase notification intent → Load specific event if present
3. WebView loads → Inject download interception script
4. User navigates → Native intercepts OAuth/downloads/external URLs
5. Back button → WebView history navigation (unless at root/login → exit app)

## Important Notes

### Apple Silicon (M1/M2/M3/M4) Setup

- Android emulators: Use ARM64 system images for best performance
- iOS simulators: Run natively on Apple Silicon (no Rosetta needed)
- FlutterFire CLI must be in PATH: Add `export PATH="$PATH:$HOME/.pub-cache/bin"` to shell profile

### Firebase Configuration

- Firebase projects configured via:
  - `android/app/google-services.json`
  - `ios/Runner/GoogleService-Info.plist`
- FlutterFire options auto-generated in `lib/firebase_options.dart`

### Release Builds

**Android:** Requires keystore at `../../environment/key.properties` with keys:
- keyAlias
- keyPassword
- storeFile
- storePassword

**iOS:** Requires Apple Developer account and code signing configuration in Xcode.

### WebView JavaScript Injection

The app injects JavaScript on every page load to intercept Excel file blob creation. Any changes to file download handling must update this injection script in the `onPageFinished` callback.

## License

Apache License, Version 2.0
