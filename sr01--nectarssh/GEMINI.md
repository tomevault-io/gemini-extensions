## nectarssh

> Android SSH tunnel app using Kotlin, Jetpack Compose, Material3, and SSHJ library.

# Claude Code Rules for NectarSSH

## Overview
Android SSH tunnel app using Kotlin, Jetpack Compose, Material3, and SSHJ library.

**App ID**: `com.rosi.nectarssh` | **Min SDK**: 24 | **Target SDK**: 36

## Project Structure
```
com.rosi.tunnel/
├── MainActivity.kt              # Tab navigation (Port Forwards, Connections, Identities)
├── ConnectionActivity.kt        # Create/edit SSH connections
├── IdentityActivity.kt          # Create/edit identities (user, password, keys)
├── data/
│   ├── Identity.kt              # Identity data class
│   ├── IdentityStorage.kt       # Identity persistence (identities.json)
│   ├── Connection.kt            # Connection data class
│   ├── ConnectionStorage.kt     # Connection persistence (connections.json)
│   ├── SessionState.kt          # Active SSH session state
│   ├── LogEntry.kt              # Log entry model
│   └── PassphraseRequest.kt     # Passphrase dialog request
├── service/
│   ├── SSHTunnelService.kt      # Foreground service for SSH connections
│   └── NotificationHelper.kt    # Notification management
└── ui/
    ├── connection/
    │   ├── ConnectionLogActivity.kt
    │   └── ConnectionLogScreen.kt
    └── theme/                   # Material3 theme
```

## Critical Rules

### BouncyCastle Initialization
Always initialize in `MainActivity.onCreate()` before any SSH operations:
```kotlin
Security.removeProvider("BC")
Security.insertProviderAt(BouncyCastleProvider(), 1)
```

### SSH Operations
- Use `Dispatchers.IO` for all SSH/network operations
- Use `SSHClient()` with `PromiscuousVerifier()` for host verification
- Run in foreground service (`SSHTunnelService`) for persistent connections
- Use `SharedFlow` for real-time log streaming to UI

### Data Storage
- Store in app private storage (`Context.filesDir`)
- Use `@Serializable` with `kotlinx-serialization-json`
- Private keys stored in `keys/` with UUID filenames

### UI Patterns
- `LazyColumn` with Cards, 16dp padding, 8dp spacing
- `PasswordVisualTransformation()` for password fields
- `LaunchedEffect` with refresh trigger for data reload
- Back navigation: `Icons.AutoMirrored.Filled.ArrowBack`

## Build Commands
```bash
./gradlew assembleDebug                    # Build
./gradlew installDebug                     # Install
adb shell am start -n com.rosi.nectarssh/.MainActivity  # Launch
```

## Don'ts
- Don't run SSH operations on Main thread
- Don't store passwords in SharedPreferences
- Don't hardcode file paths
- Don't skip building after code changes
- Don't log passwords

---
> Source: [sr01/NectarSSH](https://github.com/sr01/NectarSSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-24 -->
