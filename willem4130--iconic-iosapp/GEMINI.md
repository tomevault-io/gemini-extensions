## iconic-iosapp

> Official mobile app for Iconic Festival 2026 - a tribute band festival at Goffertpark, Nijmegen.

# Iconic Festival iOS App

Official mobile app for Iconic Festival 2026 - a tribute band festival at Goffertpark, Nijmegen.

**Stack:** SwiftUI + Swift 5.9 + iOS 17+ + SwiftData + Claude API
**Target:** 50+ year olds - UI prioritizes clarity and simplicity
**Bundle ID:** `nl.iconic.festivalapp`

## Quick Start

```bash
cd /Users/willemvandenberg/Dev/Iconic/iconic-iosapp
open IconicFestival.xcodeproj  # Then Cmd+R to run
```

**CLI Build & Run:**
```bash
# Build
xcodebuild -scheme IconicFestival -destination 'platform=iOS Simulator,name=Iphone Willem' build

# Install & launch
xcrun simctl install booted ~/Library/Developer/Xcode/DerivedData/IconicFestival-*/Build/Products/Debug-iphonesimulator/IconicFestival.app
xcrun simctl launch booted nl.iconic.festivalapp
```

## Project Structure

```
IconicFestival/
├── App/                    # App entry, RootView
├── Core/
│   ├── Theme/              # AppColors, ThemeManager
│   ├── Navigation/         # Router, Routes
│   ├── Components/         # Reusable UI (CachedAsyncImage, etc.)
│   ├── Storage/            # KeychainManager, UserDefaultsManager
│   └── Extensions/         # Swift extensions
├── Features/
│   ├── Timetable/          # Festival schedule
│   ├── Info/               # FAQ, venue, contact
│   ├── Chat/               # AI assistant
│   └── Settings/           # Settings, About
├── Models/                 # TimetableData, FestivalInfo
├── Services/               # ClaudeService
└── Configuration/          # Debug/Production xcconfig
```

## Brand Colors

| Color | Hex | Usage |
|-------|-----|-------|
| Primary Dark | `#08192C` | Nav bars, headers |
| Primary Gold | `#F29100` | Accents, buttons, highlights |
| Warm Cream | `#EFE9E4` | Optional warm backgrounds |
| Main Stage | `#AA7712` | Stage indicator (gold) |
| Theater | `#E8927C` | Stage indicator (coral) |

**Consistent styling across all views:**
- Headers: Navy (`primaryDark`) background with logo + gold date
- Section cards: `primaryGold.opacity(0.1)` background
- Content cards: `secondaryBackground` with subtle borders
- Accent elements: Stage colors for differentiation

## Key Files

| File | Purpose |
|------|---------|
| `Core/Theme/AppColors.swift` | All brand colors defined here |
| `Models/TimetableData.swift` | Artists, performances, socials |
| `Models/FestivalInfo.swift` | Contact, FAQ, venue details |
| `Services/ClaudeService.swift` | Claude API integration |
| `Configuration/Secrets.xcconfig` | API keys (gitignored) |

## App Tabs

| Tab | View | Purpose |
|-----|------|---------|
| Programma | `TimetableView` | Two-column timeline with both stages |
| Info | `InfoView` | FAQ, venue location, contact |
| Vraag | `ChatView` | AI chat assistant |
| Meer | `SettingsView` | Settings, about, social links |

## UI Patterns

**All main views share:**
- Compact header: Logo (36px) + title + social links
- Navigation bars hidden for maximum content
- Gold accents for interactive elements
- `secondaryBackground` for cards

**Timetable cards:**
- `secondaryBackground` base
- Colored left accent bar (gold/coral per stage)
- Subtle stage-colored border
- Bold hour labels in gold

**Performance detail sheet:** Artist photo, bio, time, social links

## Timetable 2026

**Main Stage:** Beach Boys' Best (14:00) → Coming on Strong → Cosmic Carnival → Treasure → Donna's Hot Stuff → Dirty Daddies (22:15 Headliner)

**Openluchttheater:** ABBA GOLD (15:00) → Urban Solitude → Future Nostalgia → The Dutch Queen (21:00 Headliner)

## Festival Links

- **Website:** https://www.iconicfestival.nl
- **Instagram:** https://www.instagram.com/iconic_festival/
- **Facebook:** https://www.facebook.com/iconictribute/

Social links defined in `FestivalInfo.swift` → `ContactInfo` struct.

## Adding Features

1. Create view in `Features/[Feature]/View/`
2. Add ViewModel if needed in `Features/[Feature]/ViewModel/`
3. Add route to `Core/Navigation/Route.swift`
4. Handle in `Router.destination(for:)`

## CLI Tools

Installed tools for iOS development:

| Tool | Purpose |
|------|---------|
| `tuist` | Project generation & build caching |
| `xcodes` | Xcode version management |
| `fastlane` | Build automation & App Store deployment |

## Commands

```bash
open IconicFestival.xcodeproj     # Open project
xcodebuild -scheme IconicFestival -destination 'platform=iOS Simulator,name=Iphone Willem' build
xcodebuild test -scheme IconicFestival -destination 'platform=iOS Simulator,name=Iphone Willem'
fastlane build                    # Automated build (when configured)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/willem4130) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-09 -->
