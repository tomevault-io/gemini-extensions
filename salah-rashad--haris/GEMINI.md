## haris

> │── main.dart                # App entrypoint, setup GetIt + runApp


# 📂 Project Folder Structure — Haris (Phase 1 MVP)

```
lib/
│── main.dart                # App entrypoint, setup GetIt + runApp
│
├── core/                     # Shared resources across app
│   ├── di/                   # Dependency injection (GetIt)
│   │    └── injector.dart
│   ├── services/             # Generic services
│   │    ├── notification_service.dart
│   │    ├── overlay_service.dart
│   │    ├── floating_widget_service.dart
│   │    └── background_service.dart
│   ├── utils/                # Helpers, constants
│   │    ├── app_colors.dart
│   │    ├── app_strings.dart
│   │    └── date_time_helper.dart
│   └── database/             # Drift setup
│        ├── app_database.dart
│        └── tables/
│             ├── tasks.dart
│             └── relapses.dart
│
├── features/                 # MVC split per feature
│   ├── tasks/                # Daily Tasks System
│   │    ├── data/            # Data Layer (models + repos)
│   │    │    ├── task_model.dart
│   │    │    └── task_repository.dart
│   │    ├── logic/           # BLoC
│   │    │    └── task_bloc.dart
│   │    └── presentation/    # UI (Views + Widgets)
│   │         ├── tasks_screen.dart
│   │         └── widgets/
│   │              └── task_item.dart
│   │
│   ├── overlay/              # Mandatory Overlay
│   │    ├── data/
│   │    ├── logic/
│   │    │    └── overlay_bloc.dart
│   │    └── presentation/
│   │         └── overlay_screen.dart
│   │
│   ├── floating_widget/      # Floating Quick Button
│   │    ├── data/
│   │    ├── logic/
│   │    │    └── floating_widget_bloc.dart
│   │    └── presentation/
│   │         └── floating_menu.dart
│   │
│   ├── relapses/             # Relapse Logger
│   │    ├── data/
│   │    │    ├── relapse_model.dart
│   │    │    └── relapse_repository.dart
│   │    ├── logic/
│   │    │    └── relapse_bloc.dart
│   │    └── presentation/
│   │         ├── relapse_log_screen.dart
│   │         └── relapse_history_screen.dart
│   │
│   └── home/                 # Main entry screen
│        └── home_screen.dart
│
└── config/                   # App-level configuration
     ├── app_routes.dart
     └── app_theme.dart
```

---

# ⚙️ Key Points in This Structure

* **MVC separation inside each feature**:

  * `data/` → models, repositories (talks to Drift DB).
  * `logic/` → BLoC for state management.
  * `presentation/` → screens + widgets (UI).

* **Drift database** in `core/database/`:

  * `tasks.dart` → Drift table for daily tasks.
  * `relapses.dart` → Drift table for relapse logs.
  * `app_database.dart` → central DB setup.

* **GetIt (DI)** in `core/di/injector.dart`:

  * Register services, repositories, blocs.
  * Makes features decoupled + testable.

* **Services** in `core/services/`:

  * `overlay_service.dart` → handles native overlay.
  * `floating_widget_service.dart` → floating bubble (platform channel).
  * `background_service.dart` → scheduled triggers.

---

# 🛠️ Example Workflow (Phase 1 MVP)

1. **User opens app** → sees `HomeScreen` with today’s tasks.
2. **Task time arrives** → `OverlayService` triggers `OverlayScreen`.
3. **Overlay** blocks exit until user completes the step.
4. **User wants quick dhikr before sleep** → taps **floating button** → launches `FloatingMenu`.
5. **User relapses** → logs it via `RelapseLogScreen`.
6. All data (tasks/relapses) saved in **Drift DB**, managed via repos + injected by **GetIt**.

---

👉 With this structure, you can **code MVP immediately**, keep features isolated, and expand smoothly into Phase 2 (Smart Guardian) later without breaking things.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/salah-rashad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-13 -->
