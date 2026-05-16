## ruko

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This App Does

**Ruko** is a Flutter mobile app that gamifies camera roll cleanup. Users swipe right to keep and left to delete photos/videos — dating-app mechanics applied to photo management. Currently released on iOS; Android coming soon.

## Commands

```bash
# Install dependencies
flutter pub get
cd ios && pod install && cd ..

# Run in development
flutter run --target=lib/main_development.dart

# Code generation (routes, freezed, json) — watch mode
dart run build_runner watch

# Code generation — one shot
dart run build_runner build --delete-conflicting-outputs

# Build iOS release (from pubspec scripts)
flutter build ipa --obfuscate --split-debug-info=debug-info --target=lib/main_production.dart

# Lint (no separate lint command — analysis runs via IDE or)
flutter analyze
```

**Entry points:** `lib/main_development.dart` and `lib/main_production.dart` (build flavors: development, staging, production via `lib/core/flavors.dart`).

## Architecture

### Layer Structure

```
UI Layer         → lib/app/**/ui/          (pages, widgets)
State Layer      → lib/app/**/cubit/       (Cubits with Freezed states)
Core/Infra       → lib/core/               (DI, routing, theme, utilities)
```

### Key Cubits (State Management via `flutter_bloc`)

| Cubit | Scope | Purpose |
|---|---|---|
| `AlbumsCubit` | Root (preloaded, `lazy: false`) | Album list with thumbnails |
| `AssetPathsCubit` | Root | Album path entities from photo library |
| `AssetsPaginatorCubit` | Per swiper page | Thread-safe paginated photo stream (uses `Mutex`) |
| `ImageDeleteCubit` | Per swiper page | Track marked-for-deletion assets, undo support |

All states are immutable via `freezed`. `hydrated_bloc` persists state across restarts.

### Navigation (`auto_route`)

Routes are defined in `lib/core/router/router.dart` and **generated** into `router.gr.dart` (do not edit manually). All routes use fade transitions. Key routes:

- `SplashRouteRoute` → checks permissions → redirects to `HomeRoute` or `PermissionRequestRoute`
- `HomeRoute` → default all-photos swiper
- `GenericSwiperRoute` — reusable swiper for any album (albums, shuffled, videos, old-first, etc.)
- `CategoriesBottomSheetRoute` — modal for selecting cleanup mode
- `FSAssetRoute` — full-screen photo/video detail

### App Initialization Flow

`Bootstrap.production()` → `setupServiceLocator()` (GetIt) → `InstalledAppsRepository.loadInstalledApps()` → `DotEnv.load()` → `HydratedBloc.storage` init → `runApp(App())` → `SplashScreen`

### Dependency Injection

`get_it` service locator configured in `lib/core/di/service_locator.dart`. New services must be registered there.

### Photo Library Integration

`photo_manager` (v3.6.4) accesses the iOS photo library. `FilterOptions` in `lib/app/gallery/utils/filter_options.dart` defines preset `PMFilter` configurations. Deletion uses a native method channel (`photo_utils`) to check iOS deletability before allowing swipe-to-delete.

### Cleanup Modes

All modes feed into `GenericSwiperPage`. Modes include: By Month, By Location (geohashing via `dart_geohash`), Screenshots, Videos Only, Shuffle, Oldest First, Albums.

## Code Generation

After modifying any file annotated with `@freezed`, `@RoutePage()`, or `@JsonSerializable()`, run build_runner to regenerate. Generated files (`*.freezed.dart`, `*.g.dart`, `router.gr.dart`) are committed to the repo.

## Key Utilities

- **`core_extensions.dart`** — `.let()`, `.letOrElse()`, `.withDefault()`, `.p()` (padding shorthand), `.blur()`
- **`Throttler` / `Debouncer`** — in `lib/core/limiters/`
- **`Paginator<T>`** — generic, mutex-protected pagination base class in `lib/core/paginator/`
- **`TaskStatus` enum** — `initial | running | success | error` for loading states

## Environment

A `.env` file is bundled as a Flutter asset (contains `FACEBOOK_APP_ID`). Loaded via `flutter_dotenv` in bootstrap.

---
> Source: [sheeroo/ruko](https://github.com/sheeroo/ruko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-14 -->
