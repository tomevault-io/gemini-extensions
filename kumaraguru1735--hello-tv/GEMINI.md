## hello-tv

> Android IPTV streaming app supporting both **Android TV** (leanback) and **Mobile** devices. Built with Kotlin, Jetpack Compose, and ExoPlayer (Media3). StarPlay-inspired UI with gold accent theme.

# HelloTV - IPTV Streaming App

## Project Overview
Android IPTV streaming app supporting both **Android TV** (leanback) and **Mobile** devices. Built with Kotlin, Jetpack Compose, and ExoPlayer (Media3). StarPlay-inspired UI with gold accent theme.

## Architecture
- **Pattern**: MVVM (ViewModel + Compose state)
- **Namespace**: `com.shadow.hellotv`
- **SDK**: Min 24, Target 36, Compile 36
- **Player**: ExoPlayer/Media3 v1.7.1 (HLS, DASH, RTMP, DRM)
- **DB**: Room (local cache for offline-first)
- **API**: REST POST JSON at `https://iptv.helloiptv.in/api`

## File Structure
```
app/src/main/java/com/shadow/hellotv/
├── MainActivity.kt              # Entry point, TV/mobile detection, orientation
├── model/ApiModels.kt           # All API data classes (Channel, Category, etc.)
├── network/ApiService.kt        # HTTP client, all API calls
├── viewmodel/MainViewModel.kt   # Single ViewModel, all app state
├── data/
│   ├── local/AppDatabase.kt     # Room database
│   ├── local/dao/               # ChannelDao, CategoryDao, LanguageDao
│   ├── local/entity/            # Room entities
│   └── repository/ChannelRepository.kt
├── utils/
│   ├── SessionManager.kt        # SharedPreferences auth/session storage
│   ├── KeepScreenOn.kt          # Screen wake lock
│   └── calculateDistance.kt
├── ui/
│   ├── ExoPlayerView.kt         # Reusable ExoPlayer composable (headers, DRM, HEVC)
│   ├── common/
│   │   ├── LoginScreen.kt       # Responsive login (portrait/landscape)
│   │   └── SessionKickoutScreen.kt
│   ├── mobile/
│   │   ├── MobilePlayerScreen.kt    # Full mobile player (portrait + fullscreen)
│   │   ├── MobileChannelSheet.kt
│   │   ├── MobileSettingsSheet.kt
│   │   └── MobileSessionSheet.kt
│   ├── tv/
│   │   ├── TvPlayerScreen.kt       # TV player with D-pad navigation
│   │   ├── TvChannelMenu.kt        # TV channel browser
│   │   ├── TvSettingsPanel.kt
│   │   └── TvChannelNumberOverlay.kt
│   └── theme/
│       ├── Color.kt, Theme.kt, Type.kt, PlayerTheme.kt
```

## Design System

### Colors (StarPlay-inspired Gold Accent)
- **AccentGold**: `#FFB800` - Primary accent for selections, indicators, active states
- **SurfaceDark**: `#121212` - Main dark background
- **SurfaceCard**: `#1A1A2E` - Card/panel backgrounds
- **DarkCard**: `#1C1C2A` - Channel row backgrounds
- **TvSurface**: `#0D0D0D` - TV-specific dark background
- **TvSurfaceCard**: `#1A1A1A` - TV card background

### UI Patterns
- **Channel rows**: Dark card bg, channel logo, name, gold star icon, equalizer on selected
- **Category items**: Gold left border indicator on selected, white text
- **Language tabs**: Bordered chips, gold fill when selected, black text on selected
- **Fullscreen controls**: 10s rewind/forward (circle outline), clean play/pause, seek bar with timestamps
- **Bottom buttons**: Quality, Lock, Audio & Subtitles, More Channels
- **Floating panels**: Rounded rect cards (`RoundedCornerShape(12.dp)`, `Color(0xCC303040)`), scrollable, dismiss on outside tap
- **Screen fit toggle**: Top-right icon, toggles `RESIZE_MODE_FIT`/`RESIZE_MODE_ZOOM`

### Compose Patterns Used
- `AnimatedVisibility` with slide animations for panels
- `detectTapGestures` / `detectDragGestures` for player controls
- `LazyColumn` / `LazyRow` with `rememberLazyListState`
- `CompositionLocalProvider` for TV detection (`LocalIsTv`)
- `AndroidView` for ExoPlayer `PlayerView` integration
- `DisposableEffect` for lifecycle-aware player management
- `BackHandler` for navigation stack

## ExoPlayer Configuration
- **HEVC fallback**: `DefaultRenderersFactory` with `EXTENSION_RENDERER_MODE_ON` + `setEnableDecoderFallback(true)`
- **Headers**: Parsed from `JsonPrimitive` or `JsonObject` channel headers
- **DRM**: Widevine, PlayReady, ClearKey support
- **Resize modes**: `RESIZE_MODE_FIT` (default) and `RESIZE_MODE_ZOOM` (screen fill)
- **Track selection**: `TrackSelectionOverride` for quality/audio switching via `trackSelectionParameters`

## API Endpoints (all POST, JSON body)
- `auth.php?action=login` - `{phone, pin, device_id, device_type, device_model, device_name}`
- `auth.php?action=validate` - `{session_token}`
- `auth.php?action=logout` - `{session_token}`
- `auth.php?action=sessions` - `{session_token}`
- `auth.php?action=kick_device` - `{session_token, target_session_id}`
- `auth.php?action=replace_device` - `{session_token, target_session_id}`
- `channels.php` - `{session_token}` → channels list
- `categories.php` - `{session_token}` → categories list
- `languages.php` - `{session_token}` → languages list

## Development Guidelines

### When modifying UI:
1. Always maintain the gold accent (#FFB800) color scheme
2. Keep TV and mobile UIs separate (tv/ vs mobile/ packages)
3. Use `ExoPlayerView.kt` for all player instances - don't create new player composables
4. Test both portrait and landscape on mobile
5. Floating panels should dismiss on outside tap (transparent clickable Box behind)
6. Channel panels stay open until explicitly dismissed

### When modifying player:
1. `ExoPlayerView` handles headers, DRM, HEVC - modify there for player config changes
2. `resizeMode` parameter controls fit/zoom - set on `PlayerView`, not `ExoPlayer`
3. Track selection uses `currentTracks.groups` to enumerate, `TrackSelectionOverride` to select
4. Lifecycle handling: pause on background, resume on foreground via `LifecycleEventObserver`

### When modifying API/data:
1. All models in `ApiModels.kt` - uses kotlinx.serialization
2. `ApiService` is a singleton object with OkHttp
3. Room DB is offline cache - always try DB first, then sync from API
4. Session management in `SessionManager` - handles auto re-login

### Code style:
- Single ViewModel (`MainViewModel`) holds all app state
- Compose state with `mutableStateOf` / `mutableIntStateOf`
- Private logging helper: `private fun log(msg: String) { Log.e(TAG, msg) }`
- Minimal comments - code should be self-explanatory
- No unnecessary abstractions - keep it simple

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kumaraguru1735) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-09 -->
