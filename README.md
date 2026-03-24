# Vehicle Inventory Management App

Cross-platform Flutter app (Android / iOS / Windows) for field technicians to track equipment inside vehicles. Built with Supabase, Riverpod, and GoRouter.

---

## Setup

### 1. Supabase

1. Create a project at [supabase.com](https://supabase.com)
2. Run the migration in the SQL Editor:
   ```
   supabase/migrations/001_create_tables.sql
   ```
3. Copy your **Project URL** and **anon key** from Settings → API

### 2. Configure credentials

Pass credentials via `--dart-define` at build/run time (never hard-code them):

```bash
flutter run \
  --dart-define=SUPABASE_URL=https://YOUR_PROJECT.supabase.co \
  --dart-define=SUPABASE_ANON_KEY=YOUR_ANON_KEY
```

Or create a `launch.json` in VS Code:

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

### 3. Install dependencies

```bash
flutter pub get
```

### 4. Run

```bash
flutter run
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
│   └── services/                # Supabase singleton + file import parser
├── data/                        # Global data layer
│   ├── models/                  # Immutable Dart data classes
│   └── repositories/            # Supabase CRUD (one per entity)
└── features/                    # Self-contained feature modules
    ├── vehicles/                 # Vehicle selection + dashboard
    ├── inventory/                # Live inventory tracking screen
    ├── reports/                  # Report list + detail + export
    └── settings/                 # Placeholder settings screen
```

## Importing a vehicle configuration

From the "Add Vehicle" dialog, tap **Import config** and select a JSON or CSV file.

**JSON format** (`assets/sample_vehicle_config.json`):
```json
{
  "sections": [
    {
      "name": "Section Name",
      "devices": [{ "name": "Device", "target_quantity": 1, "has_fuel": false }],
      "subsections": [...]
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

## Extending the app

- **New feature**: Add `features/<name>/` with screens, widgets, providers. Register one route in `core/routing/app_router.dart`.
- **New export format**: Add alongside `_exportJson` / `_exportPdf` in `report_detail_screen.dart`.
- **Authentication**: Add `core/services/auth_service.dart` + a redirect guard in `app_router.dart`.
- **Barcode scanning**: New `features/barcode_scanner/` module; no existing files need modification.

## Production checklist

- [ ] Replace RLS dev policies in `001_create_tables.sql` with user-scoped policies
- [ ] Configure Supabase Auth (email/magic link or SSO)
- [ ] Set up Supabase Storage for cloud report uploads (`uploadReportToCloud`)
- [ ] Add CI/CD with `flutter build` for Android APK, iOS IPA, and Windows EXE
- [ ] Enable error monitoring (e.g., Sentry)
