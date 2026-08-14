# Ausruestung

Flutter application for managing climbing equipment and PPE inspections.
Target platforms are Linux and Android; development takes place on Linux with
IntelliJ.

This file is aimed at developers. Operation is described in `HOWTO.md`.

## Structure

Business logic lives in `lib/services/`, the user interface in `lib/`. The tabs
do not know about each other: shared state lives in `app_state.dart` and is
passed on via signals. Services do not know about `BuildContext` — they return
messages, and the interface displays them with `notify()`.

The services are assembled once in `main.dart` (ConfigManager, PlatformInfo,
DatabaseManager, NfcService) and passed on as a ready-made `AppController`.

## Data model

Three tables in SQLite:

| Table | Contents |
| --- | --- |
| `artikel` | single piece of equipment, consecutive `ArtikelID`, own serial number |
| `teile` | model (name, manufacturer, material, standard), consecutive `TeilNummer` |
| `pruefungen` | inspection dates, attached to the item via `ArtikelID` |

An item refers to a part via `TeilNummer`; a part can belong to many items.
Inspections go with the item when it is deleted, by way of
`ON DELETE CASCADE` — for that, `PRAGMA foreign_keys = ON` must be set, which
`db_manager.open()` takes care of in `onConfigure`.

`ArtikelID` and `TeilNummer` are assigned on creation and are not editable in
the interface. Pictures are stored as BLOBs in the database and are passed to
the outside as Base64 text.

## Multi-client

Every client has its own folder:

```
<base>/<client>/ini/config.ini
<base>/<client>/sql_db/Ausruestung.db
```

The base is `~/Ausruestung` on the desktop, and the platform's app folder on
mobile (`getApplicationSupportDirectory()`). On startup the client used last is
opened — identifiable by `last_access` in the respective `config.ini`.

## Particulars

- Saving happens when a field is left, not on every keystroke, and only if
  something actually changed.
- Dates are held internally as ISO and displayed in the configured format. The
  retirement date is calculated from the purchase date and the service life —
  unless it is entered by hand.
- Pictures are not loaded together with the list; instead they are fetched for
  the displayed record after a short pause.
- NFC runs via `nfc_manager` on Android and via a PC/SC reader on the desktop.
  Both listen continuously; the interface notices no difference. A scan enters
  the serial number if the cursor is in the empty serial number field —
  otherwise the matching item is searched for.
- PDF in two forms: one sheet per item, or one packing list per storage
  location.
- Import replaces existing records, migration skips them.

## Logging

`logger.dart` provides `logDebug`, `logError`, `logWarning`, `logInfo`.
`logError(what, error)` writes the full message to the log and returns a short
sentence for display. Nothing goes to the snackbar only. The debug tab shows
the log and can be switched on in the Options.

## Files

`lib/`

| File | Purpose |
| --- | --- |
| `main.dart` | entry point, assembling the services |
| `splash_screen.dart` | splash screen, colours taken from the icon |
| `home_page.dart` | frame with the tabs, switching clients |
| `artikel_tab.dart` | items and inspections |
| `teile_tab.dart` | parts |
| `filter_tab.dart` | universal search |
| `optionen_tab.dart` | settings and client management |
| `info_tab.dart` | system information, licences |
| `debug_tab.dart` | log view |
| `app_menu.dart` | side menu, backup, import/export, PDF |
| `app_state.dart` | `SearchState` and `SettingsState` |
| `ui_helpers.dart` | shared building blocks of the interface |
| `migration_dialog.dart` | merging clients |
| `pruefung_historie.dart` | older inspections on a page of their own |

`lib/services/`

| File | Purpose |
| --- | --- |
| `app_controller.dart` | knows the services, calls them |
| `config.dart` | `config.ini` per client, storage locations |
| `db_manager.dart` | SQLite, all read and write operations |
| `artikel_service.dart` | search, items, inspections, pictures |
| `teile_service.dart` | creating, copying and saving parts |
| `data_service.dart` | import/export as ZIP, backup |
| `migration_service.dart` | transferring data between clients |
| `optionen_service.dart` | client management |
| `pdf_reporter.dart` | item sheets and storage list |
| `nfc_service.dart` | NFC on mobile and via PC/SC |
| `image_service.dart` | scaling pictures down and writing them back |
| `graphic_utils.dart` | bytes, Base64, scaling down, loading |
| `qr_utils.dart` | QR code as PNG |
| `date_utils.dart` | interpreting and displaying dates, search ranges |
| `icon_colors.dart` | colour set from image or theme |
| `platform_info.dart` | a single place for the platform query |
| `logger.dart` | central log |
| `utils.dart` | sorting, paths, file names, backup ZIP |

Plus `pubspec.yaml` and `build-appimage.sh`.

## Building

```
flutter pub get
flutter build linux --release
flutter build apk --release
```

AppImage for Linux, from the project root after the release build:

```
./build-appimage.sh
```

The script does not build anything itself. It looks for the bundle, the
executable and the icon, asks via zenity in case of ambiguity, and downloads
linuxdeploy, the GTK plugin and appimagetool to `~/.local/bin` as needed.

## Language in the source code

Identifiers in English, comments and interface in German. Comments explain the
why, not the what — in plain German, without jargon and without umlauts in the
source code (ae/oe/ue).

