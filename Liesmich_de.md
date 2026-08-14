# Ausruestung

Flutter-Anwendung zur Verwaltung von Kletterausrüstung und PSA-Prüfungen.
Zielplattformen sind Linux und Android; entwickelt wird unter Linux mit
IntelliJ.

Diese Datei richtet sich an Entwickler. Die Bedienung beschreibt `HOWTO.md`.

## Aufbau

Fachlogik liegt in `lib/services/`, die Oberfläche in `lib/`. Die Tabs kennen
einander nicht: gemeinsamer Zustand liegt in `app_state.dart` und wird über
Signale weitergereicht. Services kennen keinen `BuildContext` — sie geben
Meldungen zurück, die Oberfläche zeigt sie mit `notify()`.

Die Services werden einmalig in `main.dart` zusammengebaut (ConfigManager,
PlatformInfo, DatabaseManager, NfcService) und als fertiger `AppController`
weitergereicht.

## Datenmodell

Drei Tabellen in SQLite:

| Tabelle | Inhalt |
| --- | --- |
| `artikel` | einzelnes Stück Ausrüstung, fortlaufende `ArtikelID`, eigene Seriennummer |
| `teile` | Bauart (Name, Hersteller, Material, Norm), fortlaufende `TeilNummer` |
| `pruefungen` | Prüftermine, hängen über `ArtikelID` am Artikel |

Ein Artikel verweist über `TeilNummer` auf ein Teil; ein Teil kann zu vielen
Artikeln gehören. Prüfungen gehen bei der Löschung des Artikels per
`ON DELETE CASCADE` mit — dafür muss `PRAGMA foreign_keys = ON` gesetzt sein,
was `db_manager.open()` in `onConfigure` erledigt.

`ArtikelID` und `TeilNummer` werden beim Anlegen vergeben und sind in der
Oberfläche nicht editierbar. Bilder liegen als BLOB in der Datenbank und
werden nach außen als Base64-Text weitergereicht.

## Multi-Kunden

Jeder Kunde hat einen eigenen Ordner:

```
<Basis>/<Kunde>/ini/config.ini
<Basis>/<Kunde>/sql_db/Ausruestung.db
```

Basis ist auf dem Desktop `~/Ausruestung`, mobil der App-Ordner der Plattform
(`getApplicationSupportDirectory()`). Beim Start wird der zuletzt benutzte
Kunde geöffnet — erkennbar an `last_access` in der jeweiligen `config.ini`.

## Besonderheiten

- Gespeichert wird beim Verlassen eines Feldes, nicht bei jedem Tastendruck,
  und nur bei tatsächlicher Änderung.
- Datumsangaben liegen intern als ISO vor und werden im eingestellten Format
  angezeigt. Das Ablegedatum wird aus Kaufdatum und Nutzungsdauer berechnet —
  außer man gibt es von Hand ein.
- Bilder werden nicht mit der Liste geladen, sondern nach kurzer Ruhe für den
  angezeigten Datensatz nachgeholt.
- NFC läuft auf Android über `nfc_manager`, auf dem Desktop über einen
  PC/SC-Leser. Beide horchen dauerhaft; die Oberfläche merkt keinen
  Unterschied. Ein Scan trägt die Seriennummer ein, wenn der Cursor im leeren
  Seriennummernfeld steht — sonst wird der Artikel dazu gesucht.
- PDF in zwei Formen: ein Blatt je Artikel, oder eine Packliste je Lagerort.
- Import ersetzt vorhandene Datensätze, Migration überspringt sie.

## Protokoll

`logger.dart` bietet `logDebug`, `logError`, `logWarning`, `logInfo`.
`logError(was, fehler)` schreibt die volle Meldung ins Protokoll und gibt
einen kurzen Satz für die Anzeige zurück. Nichts geht nur an die Snackbar.
Der Debug-Tab zeigt das Protokoll und ist in den Optionen zuschaltbar.

## Dateien

`lib/`

| Datei | Zweck |
| --- | --- |
| `main.dart` | Einstieg, Services zusammenbauen |
| `splash_screen.dart` | Startbildschirm, Farben aus dem Icon |
| `home_page.dart` | Rahmen mit Reitern, Kundenwechsel |
| `artikel_tab.dart` | Artikel und Prüfungen |
| `teile_tab.dart` | Teile |
| `filter_tab.dart` | universelle Suche |
| `optionen_tab.dart` | Einstellungen und Kundenverwaltung |
| `info_tab.dart` | Systeminformationen, Lizenzen |
| `debug_tab.dart` | Protokollansicht |
| `app_menu.dart` | Seitenmenü, Backup, Import/Export, PDF |
| `app_state.dart` | `SearchState` und `SettingsState` |
| `ui_helpers.dart` | gemeinsame Bausteine der Oberfläche |
| `migration_dialog.dart` | Kunden zusammenführen |
| `pruefung_historie.dart` | ältere Prüfungen auf eigener Seite |

`lib/services/`

| Datei | Zweck |
| --- | --- |
| `app_controller.dart` | kennt die Services, ruft sie auf |
| `config.dart` | `config.ini` je Kunde, Ablageorte |
| `db_manager.dart` | SQLite, alle Lese- und Schreibvorgänge |
| `artikel_service.dart` | Suche, Artikel, Prüfungen, Bilder |
| `teile_service.dart` | Teile anlegen, kopieren, speichern |
| `data_service.dart` | Import/Export als ZIP, Backup |
| `migration_service.dart` | Daten zwischen Kunden übertragen |
| `optionen_service.dart` | Kundenverwaltung |
| `pdf_reporter.dart` | Artikelblätter und Lagerliste |
| `nfc_service.dart` | NFC mobil und über PC/SC |
| `image_service.dart` | Bilder verkleinern und zurückschreiben |
| `graphic_utils.dart` | Bytes, Base64, Verkleinern, Laden |
| `qr_utils.dart` | QR-Code als PNG |
| `date_utils.dart` | Datum deuten, anzeigen, Suchbereiche |
| `icon_colors.dart` | Farbsatz aus Bild oder Theme |
| `platform_info.dart` | eine Stelle für die Plattformabfrage |
| `logger.dart` | zentrales Protokoll |
| `utils.dart` | Sortierung, Pfade, Dateinamen, Backup-ZIP |

Dazu `pubspec.yaml` und `build-appimage.sh`.

## Bauen

```
flutter pub get
flutter build linux --release
flutter build apk --release
```

AppImage für Linux, aus dem Projektstamm heraus nach dem Release-Build:

```
./build-appimage.sh
```

Das Skript baut nichts selbst. Es sucht Bundle, Executable und Icon,
fragt bei Mehrdeutigkeit über zenity nach und lädt linuxdeploy,
das GTK-Plugin und appimagetool bei Bedarf nach `~/.local/bin`.

## Sprache im Quelltext

Bezeichner englisch, Kommentare und Oberfläche deutsch. Kommentare erklären
das Warum, nicht das Was — in normalem Deutsch, ohne Fachjargon und ohne
Umlaute im Quelltext (ae/oe/ue).

