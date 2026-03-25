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
- Export the full vehicle pack (`.vehiclepack.json`) via the system save dialog (SAF) on Android or the native share sheet on iOS
- Config change history log

### Device list comparison & import
- Upload a document or image (PDF, JPG, PNG) containing a list of devices, or capture a photo directly with the device camera
- Extracted device names and sections are compared against the vehicle's existing configuration using an AI Vision / OCR service
- Diff viewer clearly separates devices only in the uploaded list vs. devices only in the existing config
- Select individual or multiple devices with checkboxes; bulk-assign them to any section or subsection
- Devices whose section cannot be determined are flagged with a dropdown for manual section assignment
- Tap **Add Selected** to append the chosen devices directly into the vehicle's live configuration

### AI pack generator
- Photograph vehicle sections; the AI (OpenRouter vision model) identifies all equipment and generates a complete device list
- Each detected device automatically gets a thumbnail cropped from the section photo and set as its icon
- Thumbnails uploaded to Supabase Storage (cloud mode) or saved locally to the device (offline mode)
- Preview the full detected device list before applying it to a vehicle
- Per section: choose **New section** (creates a new top-level section), **New subsection** (creates a subsection under any existing section or subsection), or **Add to existing** (insert AI-detected devices into any existing section or subsection)
- Photos can be taken directly with the **camera** or selected from the gallery — both options available side by side

### Inventory tracking
- Swipe right → **Present**, swipe left → **Missing** (haptic feedback, instant local update)
- Quantity counter for multi-unit devices
- Fuel-level slider (green / amber / red thresholds)
- Filter toggle: All items vs. Unconfirmed only
- Live progress header: present / missing / remaining counts + percentage bar
- Complete button locks the report and saves it
- Device thumbnail shown on each card (48 px, supports cloud URLs and offline local files)
- Tap thumbnail → full-screen image viewer with pinch-to-zoom (up to 6×)
- Card background colour indicates status at a glance: green (present), red (missing), yellow (present but fuel below 100 %)

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
│   ├── l10n/                    # Localizations (en, de, es, fr) + translations/
│   ├── widgets/                 # Generic shared UI components
│   └── services/                # Supabase, local DB, file import/export, AI, update
├── data/                        # Global data layer
│   ├── models/                  # Immutable Dart data classes
│   └── repositories/            # CRUD abstraction (Supabase + local/ SQLite mirrors)
└── features/                    # Self-contained feature modules
    ├── vehicles/                 # Fleet list, dashboard, config, AI generator
    ├── inventory/                # Live inventory tracking screen
    ├── reports/                  # Report list, detail, and export
    ├── device_comparison/        # Device list comparison & import
    └── settings/                 # App settings screen
```

---

## Extending the app

- **New feature**: Create `features/<name>/` with screens, widgets, and providers. Register one route in `core/routing/app_router.dart`.
- **New repository**: Extend `base_repository.dart` for Supabase; add a matching counterpart under `data/repositories/local/` for SQLite. No other files need to change.
- **New export format**: Add alongside the existing exporters in `core/services/file_export_service.dart` and surface it in `features/reports/widgets/export_buttons.dart`.
- **Authentication**: Add `core/services/auth_service.dart` + a redirect guard in `app_router.dart`.
- **Barcode scanning**: New `features/barcode_scanner/` module; no existing files need modification.

---

## Release notes

### v1.1.6 — AI generator: subsection support

- **NEW SUBSECTION mode**: each section entry in the AI generator now offers three modes — **NEW SECTION** (creates a top-level section, unchanged), **NEW SUBSECTION** (new), and **ADD TO EXISTING** (unchanged)
- In **NEW SUBSECTION** mode a parent section dropdown appears (the full flattened section tree is shown, so any existing section or subsection is a valid parent), followed by a name field for the new subsection
- On import the subsection is created under the chosen parent and all AI-detected devices for that entry are inserted directly into it
- The three mode chips are rendered in a `Wrap` so they reflow correctly on narrow screens
- No changes to the generate step — the subsection name is passed to the AI as the section label so device naming remains context-aware

### v1.1.5 — Device list comparison & import
- **File upload**: accept documents and images (PDF, JPG, PNG) from the file picker as the source for a device list
- **Camera capture**: take a photo directly from the comparison screen — same camera integration used by the AI pack generator
- **AI extraction**: uploaded file or photo is sent to an AI Vision / OCR service; the response is parsed into a structured list of device names grouped by section
- **Diff viewer**: side-by-side comparison showing devices only in the uploaded list ("to add") vs. devices only in the existing vehicle configuration ("not found in upload")
- **Multi-select & bulk assign**: checkbox selection on each diff entry; a single section dropdown applies to all selected items at once
- **Unknown section handling**: devices whose section cannot be determined by the AI are flagged in the UI with a manual section selector
- **One-tap import**: tap **Add Selected** to write the chosen devices directly into the vehicle's live configuration (section and device records created immediately)

### v1.1.3 — Build fixes & code quality
- Fixed Android release build failing with "3 issues found when checking AAR metadata": `image_picker` transitively pulls in `androidx.activity:activity-ktx:1.12.4` and `androidx.navigationevent:navigationevent-android:1.0.2`, both of which embed metadata requiring AGP 8.9.1+; resolved by pinning `androidx.activity` to 1.9.3 and excluding `navigationevent-android` via `resolutionStrategy` in `app/build.gradle.kts`
- Resolved all Flutter static analysis warnings: added missing `@override` annotations across all repository implementations (`DeviceRepository`, `SectionRepository`, `VehicleRepository`, `ReportRepository`, `EntryRepository`, `ConfigChangeRepository`)
- Removed unused `dart:io` import in `vehicle_pack_service.dart`
- Replaced deprecated `Switch.activeColor` with `activeThumbColor` in vehicle config screen
- Added `const` to a `TextStyle` constructor to improve widget rebuild performance

### v1.1.2 — AI generator improvements & inventory colour coding
- **Inventory card colours**: device cards now show a background tint based on their confirmation state — green (present), red (missing), yellow (present but fuel level below 100 %). Unconfirmed cards remain the default dark surface colour
- **AI section target**: each section entry in the AI generator now has a **NEW SECTION / ADD TO EXISTING** toggle. When "Add to existing" is selected a dropdown lists all sections and subsections of the vehicle; AI-detected devices are inserted directly into the chosen section without creating a new one
- **Camera capture**: section photos in the AI generator can now be taken directly with the device camera via a dedicated **CAMERA** button, in addition to the existing gallery picker
- **AI thumbnail quality**: updated AI prompt to request bounding boxes with 10–15 % padding on each side so the full device is always visible; crop code adds an additional 8 % safety margin at extraction time
- **Thumbnail viewer**: device thumbnails in the inventory count are now shown at 48 px and support both Supabase Storage URLs and local file paths (offline mode); tapping a thumbnail opens a full-screen pinch-to-zoom viewer

### v1.1.1 — Vehicle pack save fix & CI signing fix
- Fixed vehicle pack (and JSON/CSV config exports) not appearing in the Downloads folder after saving via a file manager
- Root cause: the share sheet fires `ACTION_SEND` (open/consume intent); when a file manager was picked from the share sheet it opened the file instead of saving it, so `ShareResultStatus` returned `success` and the SAF fallback never triggered
- Android export now goes directly to the Storage Access Framework save dialog (`ACTION_CREATE_DOCUMENT`) — the user picks the destination folder once and the file is guaranteed to be written there
- No additional permissions required; SAF is system-mediated and works on all Android versions without `WRITE_EXTERNAL_STORAGE`
- Fixed GitHub Actions release build failing with "Keystore file not found" — the keystore was decoded to `android/release.jks` but Gradle resolves relative paths from the app module directory (`android/app/`); decode step now writes to `android/app/release.jks`

### v1.0.9 — Inventory thumbnail viewer
- Device thumbnails are now displayed on every inventory card (48 px square, left of the device name)
- Both cloud-hosted thumbnails (Supabase Storage URL) and locally stored thumbnails (offline SQLite mode) are rendered correctly
- Tapping a thumbnail opens a full-screen image viewer with pinch-to-zoom support (up to 6×) and a close button
- Thumbnails generated by the AI pack generator are immediately visible in inventory counts without any additional steps

### v1.0.8 — Inventory image support groundwork
- Internal version bump to align build numbers after signing configuration changes

### v1.0.7 — Vehicle pack download fix
- Fixed exported `.vehiclepack.json` files not appearing in the Downloads app on Android 10+
- Root cause: the previous fallback used `getExternalStorageDirectory()` which writes to the app-private directory (`Android/data/…`), hidden from the system Files app under Android scoped storage
- Fallback now uses the Storage Access Framework (`ACTION_CREATE_DOCUMENT`) via a native save dialog — the user picks the destination (e.g. Downloads) and the file is immediately visible there
- Share sheet remains the primary export method; the SAF save dialog only appears if the share sheet is dismissed

### v1.0.6 — Version code fix
- Corrected the Android `versionCode` to ensure the auto-update mechanism and Play-compatible distribution recognise this as a newer build

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
