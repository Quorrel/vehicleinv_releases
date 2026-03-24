# Vehicle Inventory Management App

Cross-platform Flutter app (Android / iOS / Windows) for field technicians to track equipment inside vehicles. Built with Supabase or a local SQLite database, Riverpod state management, and GoRouter navigation.

---

## Features

### Fleet management
- Vehicle list with custom per-vehicle icons (tap the icon to upload)
- Long-press a vehicle to delete it (with confirmation)
- Global device search across the entire fleet

### Vehicle dashboard
- Device count and report count stats
- Quick actions: start a new inventory count, view past reports, manage configuration
- Last 5 reports with status badges — tap to open or resume

### Vehicle configuration
- Add, edit, and delete sections, subsections, and devices
- Devices support target quantity and fuel-level tracking
- Import a full section/device tree from a JSON or CSV file
- Config change history log

### AI pack generator
- Photograph vehicle sections; the AI (OpenRouter vision model) identifies all equipment and generates a complete device list
- Each detected device automatically gets a thumbnail cropped from the section photo and set as its icon
- Thumbnails uploaded to Supabase Storage (cloud mode) or saved locally to the device (offline mode)
- Preview the full detected device list before applying it to a vehicle

### Inventory tracking
- Swipe right → **Present**, swipe left → **Missing** (haptic feedback, instant local update)
- Quantity counter for multi-unit devices
- Fuel-level slider (green / amber / red thresholds)
- Filter toggle: All items vs. Unconfirmed only
- Live progress header: present / missing / remaining counts + percentage bar
- Complete button locks the report and saves it

### Reports
- Chronological list of past reports per vehicle (resume in-progress ones)
- Detail view: stat cards + collapsible discrepancy sections (missing items, low fuel, quantity shortages)
- Export as **JSON**, **PDF**, or share via the system share sheet

### Auto-update (Android)
- On startup the app silently checks the GitHub releases API for a newer APK
- Shows a "New Version Available" dialog with release notes and a download progress bar
- After the download completes a persistent **INSTALL** button appears — tap it to launch the native package installer
- If Android redirects to the "Install unknown apps" permission screen, grant the permission and tap **INSTALL** again to complete the installation

### Settings
- **App mode**: Administration (full CRUD) or Inventory Tracking (read-only, no edits)
- **Database backend**: Local SQLite (offline, no account needed) or Supabase (cloud sync)
- **Supabase credentials**: Saved to device, swapped at runtime without restarting
- **Language**: English, French, German, Spanish, or system default
- **Appearance**: Custom background image (dimmed behind the UI)
- **AI service**: None, or OpenRouter with a configurable model

---

## Setup

### Option A — Local database (no account needed)

1. Open the app and go to **Settings**
2. Set **Database** to **Local**
3. Start adding vehicles immediately

### Option B — Supabase (cloud sync)

1. Create a project at [supabase.com](https://supabase.com)
2. Run the migration in the SQL Editor:
   ```
   supabase/migrations/001_create_tables.sql
   ```
3. Copy your **Project URL** and **anon key** from Settings → API
4. Enter them in the app's Settings screen, or pass them at build time:

```bash
flutter run \
  --dart-define=SUPABASE_URL=https://YOUR_PROJECT.supabase.co \
  --dart-define=SUPABASE_ANON_KEY=YOUR_ANON_KEY
```

VS Code `launch.json`:
```json
{
  "configurations": [
    {
      "name": "vehicleinv",
      "request": "launch",
      "type": "dart",
      "args": [
        "--dart-define=SUPABASE_URL=https://YOUR_PROJECT.supabase.co",
        "--dart-define=SUPABASE_ANON_KEY=YOUR_ANON_KEY"
      ]
    }
  ]
}
```

### Install dependencies & run

```bash
flutter pub get
flutter run
```

---

## Importing a vehicle configuration

From the **Add Vehicle** dialog or the **Config** screen, tap **Import config** and select a JSON or CSV file.

**JSON format** (`assets/sample_vehicle_config.json`):
```json
{
  "sections": [
    {
      "name": "Engine Compartment",
      "devices": [
        { "name": "Battery", "target_quantity": 1, "has_fuel": false }
      ],
      "subsections": [
        {
          "name": "Fluid Systems",
          "devices": [
            { "name": "Engine Oil", "target_quantity": 1, "has_fuel": true }
          ]
        }
      ]
    }
  ]
}
```

**CSV format** (`assets/sample_vehicle_config.csv`):
```
section,subsection,device_name,target_quantity,has_fuel
Engine Compartment,,Battery,1,false
Engine Compartment,Fluid Systems,Engine Oil,1,true
```

---

## Architecture

```
lib/
├── main.dart / app.dart         # Entry point + MaterialApp
├── core/                        # Shared, feature-agnostic code
│   ├── theme/                   # Dark industrial theme (IBM Plex fonts)
│   ├── constants/               # Spacing, durations, sizes
│   ├── extensions/              # BuildContext, DateTime, String helpers
│   ├── routing/                 # GoRouter config + route name constants
│   ├── widgets/                 # Generic shared UI components
│   └── services/                # Supabase, local DB, file import, AI, update
├── data/                        # Global data layer
│   ├── models/                  # Immutable Dart data classes
│   └── repositories/            # CRUD abstraction (one per entity)
└── features/                    # Self-contained feature modules
    ├── vehicles/                 # Fleet list, dashboard, config, AI generator
    ├── inventory/                # Live inventory tracking screen
    ├── reports/                  # Report list, detail, and export
    └── settings/                 # App settings screen
```

---

## Extending the app

- **New feature**: Create `features/<name>/` with screens, widgets, and providers. Register one route in `core/routing/app_router.dart`.
- **New repository**: Extend `base_repository.dart` — no other files need to change.
- **New export format**: Add alongside `_exportJson` / `_exportPdf` in `report_detail_screen.dart`.
- **Authentication**: Add `core/services/auth_service.dart` + a redirect guard in `app_router.dart`.
- **Barcode scanning**: New `features/barcode_scanner/` module; no existing files need modification.

---

## Release notes

### v1.0.5 — Auto-update installation fix
- Fixed APK installation failing after granting the "Install unknown apps" permission on Android 8.0+
- Download and install are now separate steps: the download runs first, then a persistent **INSTALL** button appears
- Tapping **INSTALL** can be repeated — if Android redirects to the permission settings screen, grant the permission and tap again to complete the install
- An info banner in the dialog explains the two-step flow to the user

### v1.0.4 — AI device thumbnails
- The AI pack generator now automatically extracts a thumbnail for each detected device
- The AI returns a normalized bounding box indicating where each device appears in the section photo; the app crops that region, resizes it to 256 px, and stores it as the device icon
- Supabase backend: thumbnails are uploaded to Supabase Storage alongside manually uploaded icons
- Local (offline) backend: thumbnails are saved to the device's documents directory
- Thumbnail extraction is best-effort — individual failures are silently skipped without blocking the import
- Success message shows how many thumbnails were extracted alongside the section count

### v1.0.3 — In-app auto-update
- App now checks the GitHub releases API on every startup
- Shows a "New Version Available" dialog with version number and release notes when a newer APK is found
- Downloads the APK in-app with a real-time progress bar and cancel support
- Hands off to the native Android package installer once the download is complete
- Android `FileProvider` and `REQUEST_INSTALL_PACKAGES` permission wired up correctly for Android 7.0+

### v1.0.2 — Stability release
- Packaging and version bump fixes following v1.0.1

### v1.0.1 — First release
- Full inventory tracking: swipe right (present) / left (missing), quantity counter for multi-unit devices, fuel-level slider with green/amber/red thresholds
- Supabase cloud backend with real-time sync across all screens
- Local SQLite database — fully offline, no account required
- Vehicle, section, and device photo attachments
- Vehicle configuration import from JSON or CSV files; config export for sharing
- Supabase schema auto-migration on app update
- Report export as PDF or JSON; system share sheet integration
- AI pack generator via OpenRouter — describe sections in plain text (or attach a photo) to generate a full device list
- Application mode: Administration (full CRUD) or Inventory Tracking (read-only, locked UI)
- AI provider selection: None or OpenRouter with configurable model
- Language support: English, French, German, Spanish, system default
- Global device search across the entire fleet
- Background image customization
- Inventory count can only be completed at 100% — partial completion is blocked
- Connection error handling with retry and settings shortcut
- De-duplicate search results fix
- Download function hotfix

---

## Building a release APK

```bash
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
```

Upload the APK as a GitHub release asset tagged `vX.Y.Z` — the auto-update service will find it automatically.

---

## Production checklist

- [ ] Replace RLS dev policies in `001_create_tables.sql` with user-scoped policies
- [ ] Configure Supabase Auth (email / magic link or SSO)
- [ ] Set up Supabase Storage for cloud report uploads (`uploadReportToCloud`)
- [ ] Add a release keystore and configure `signingConfig` in `android/app/build.gradle.kts`
- [ ] Add CI/CD with `flutter build` for Android APK, iOS IPA, and Windows EXE
- [ ] Enable error monitoring (e.g., Sentry)
