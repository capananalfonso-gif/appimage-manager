# appimage-manager

Automatic symlink, launcher, and icon management for AppImages on Linux.

Drop an AppImage into a directory, and it appears in your application launcher with the correct icon and category — automatically.

[![demo](https://asciinema.org/a/nbgHMIFLWAlwG3LB.svg)](https://asciinema.org/a/nbgHMIFLWAlwG3LB)

## Features

- **Auto-detection** — scans for `.AppImage` files and registers them in a CSV registry
- **Version-aware** — when multiple versions exist, the latest is linked (by version number, not file date)
- **Symlinks** — creates clean, version-agnostic symlinks (e.g., `obsidian.AppImage` -> `Obsidian-1.9.10.AppImage`)
- **Desktop launchers** — generates `.desktop` files so apps appear in COSMIC, GNOME, KDE, or any freedesktop-compliant launcher
- **Icon extraction** — pulls icons from inside AppImages and installs them to the XDG hicolor theme
- **Category detection** — reads categories from the embedded `.desktop` file, tagged with `X-AppImage;` for easy filtering
- **StartupWMClass** — extracted from AppImages so dock/panel icons match correctly
- **systemd watcher** — optional file watcher auto-triggers on new/removed AppImages
- **CSV registry** — single source of truth, user-editable labels and categories
- **Clean removal** — removed apps are cleaned up (symlink, .desktop, icon), but stay in the CSV as history

## Quick start

```bash
# Download
curl -LO https://raw.githubusercontent.com/pvojnisek/appimage-manager/main/appimage-manager.py
chmod +x appimage-manager.py

# Move to your AppImage directory
mv appimage-manager.py ~/apps/    # or ~/Applications/, or anywhere

# Run it
cd ~/apps
./appimage-manager.py             # Scan, extract icons, create launchers

# Optional: auto-trigger on file changes
./appimage-manager.py --install-watch
```

Hit **Super** and search for your app — it should appear with its icon.

## Usage

```
appimage-manager.py                        # Scan + sync (default)
appimage-manager.py list                   # Show managed apps
appimage-manager.py scan                   # Scan only, update CSV
appimage-manager.py sync                   # Sync symlinks + .desktop from CSV
appimage-manager.py extract-icons          # Extract missing icons + categories
appimage-manager.py extract-icons --force  # Re-extract all
appimage-manager.py --dry-run              # Preview changes
appimage-manager.py --install-watch        # Setup systemd auto-trigger
appimage-manager.py --uninstall-watch      # Remove auto-trigger
appimage-manager.py --apps-dir ~/Applications list  # Custom directory
```

## How it works

```
1. SCAN      Detect *.AppImage files, group by name, pick latest version
                |
2. REGISTER  Save to appimages.csv (id, label, filename, status)
                |
3. EXTRACT   From inside each AppImage, pull:
             - Icon → install to ~/.local/share/icons/hicolor/
             - Categories + StartupWMClass → update CSV
                |
4. SYNC      Create symlinks + .desktop launchers from the enriched registry
                |
5. LAUNCHER  Apps appear in COSMIC / GNOME / KDE with proper icons + dock support
```

**Updating an app:** Drop the new `.AppImage` version. The script detects the highest version number and re-links the symlink automatically. You can delete the old version whenever you like — the script only tracks the latest.

**Multiple versions:** If multiple versions of the same app exist in the directory, the one with the highest version number is used (mtime as tiebreaker). Old versions are not deleted automatically — that's up to you.

**Removing an app:** Delete all `.AppImage` files for that app. The script marks it as `removed`, cleans up the symlink, `.desktop` file, and icon from the hicolor theme. The CSV entry is preserved as history.

## Configuration

Edit the `CONFIG` section in the script, or use CLI flags:

| Setting | Default | CLI flag | Description |
|---------|---------|----------|-------------|
| `apps_dir` | `~/apps` | `--apps-dir` | Directory to scan for AppImages |
| `desktop_dir` | `~/.local/share/applications` | `--desktop-dir` | Where to write `.desktop` files |
| `csv_file` | `appimages.csv` | `--csv` | Registry filename |
| `desktop_prefix` | `appimage-` | — | Prefix for managed `.desktop` files |
| `default_categories` | `Utility;X-AppImage;` | — | Default categories for new apps |

Icons are installed to `~/.local/share/icons/hicolor/` following the [freedesktop Icon Theme Specification](https://specifications.freedesktop.org/icon-theme/latest/).

## Registry

The `appimages.csv` file is the single source of truth. User-editable columns:

| Column | Purpose |
|--------|---------|
| `label` | Display name in launcher |
| `icon` | Icon theme name (e.g., `appimage-obsidian`) |
| `categories` | `.desktop` categories (e.g., `Graphics;`, `Development;`) |
| `startup_wm_class` | Window class for dock/panel matching |
| `terminal` | `true` / `false` — launch in terminal |
| `status` | `active` / `ignored` / `removed` |

After editing the CSV, run `./appimage-manager.py sync` to apply changes.

### States

| Status | Symlink | .desktop | Icon (hicolor) | CSV row |
|--------|---------|----------|----------------|---------|
| `active` | created | created | installed | kept |
| `ignored` | removed | removed | removed | kept |
| `removed` | removed | removed | removed | kept |

## systemd watcher

The `--install-watch` command creates a systemd user path unit that monitors the AppImage directory. It triggers when files are added or removed (not on content edits).

```bash
systemctl --user status appimage-manager.path     # Check watcher
journalctl --user -u appimage-manager.service -n 20  # View logs
```

To update after editing the CSV: run `./appimage-manager.py sync` manually (CSV edits don't trigger the watcher, by design).

## Why this tool?

| Tool | Status | Dependencies | Approach |
|------|--------|-------------|----------|
| **appimage-manager** | Active | Python 3.9+ (stdlib only) | Single file, CSV registry, systemd |
| AppImageLauncher | Archived | C++, system daemon | Intercepts all AppImage launches |
| Gear Lever | Active | Flatpak, GTK4 | GUI app, requires Flatpak |
| AM / AppMan | Active | Bash, curl | Package manager (download + install) |

This tool does one thing well: make your existing AppImages visible in the launcher and dock. No daemon, no GUI, no package management. One file, zero dependencies.

## Requirements

- Python 3.9+
- Linux with a freedesktop-compliant desktop (COSMIC, GNOME, KDE, etc.)
- systemd (optional, for the file watcher)

## Security

This tool executes AppImage files (via `--appimage-extract`) to extract icons and metadata. **Only place AppImages from trusted sources in the managed directory.** See [SECURITY.md](SECURITY.md) for details.

## Contributing

Contributions are welcome! Please open an issue first to discuss what you'd like to change.

## License

[MIT](LICENSE)
