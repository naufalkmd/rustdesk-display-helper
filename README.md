# rustdesk-display-helper

Windows helper app for switching display profiles when starting/stopping a RustDesk session.

You can use it in two ways:
- GUI app (`rustdesk-display-app.exe`) for non-technical users
- CLI (`rustdesk-display-cli.exe`) for scripts and power users

## What You Asked For (Now Included)

- App-style GUI to edit/apply profiles
- Auto-detect current desktop resolution from inside the app
- Manual profile editing (`profiles.toml`) still supported
- Bundled ScreenRes project (MIT) in this repo: `third_party/screenres`

## User Setup (Low Friction)

```powershell
cd <path-to-repo>\rustdesk-display-cli
powershell -ExecutionPolicy Bypass -File .\scripts\setup-user.ps1
```

This setup script:
- Ensures `screenres.exe` is available (prefers vendored copy)
- Builds release binaries if `cargo` exists
- Creates Desktop shortcut:
  - `RustDesk Display App`

Optional advanced shortcuts:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\setup-user.ps1 -IncludeAdvancedShortcuts
```

This also adds:
- `RustDesk Remote On`
- `RustDesk Remote Off`

## Everyday Use

- Open GUI app:
```powershell
.\scripts\open-app.ps1
```
- Toggle for RustDesk:
```powershell
.\scripts\remote-on.ps1
.\scripts\remote-off.ps1
```

## Config File

Default config path:
- If `config/profiles.toml` exists in current working directory, it is used.
- Otherwise: `%APPDATA%\RustDeskDisplay\profiles.toml`

Example:

```toml
# Optional absolute path override for screenres.exe
# resolution_tool_path = "C:\\Tools\\screenres.exe"

remote_profile = "macbook_remote"
local_profile = "desktop"

[profiles.macbook_remote]
display_switch = "internal"
width = 2560
height = 1600
scale_percent = 100

[profiles.desktop]
display_switch = "extend"
width = 2048
height = 1152
scale_use_windows_default = true
```

## CLI Commands

```powershell
rustdesk-display-cli list
rustdesk-display-cli remote-on
rustdesk-display-cli remote-off
rustdesk-display-cli apply macbook_remote
```

## Build Binaries

```powershell
cargo build --release --bin rustdesk-display-cli --bin rustdesk-display-app
```

Outputs:
- `target\release\rustdesk-display-cli.exe`
- `target\release\rustdesk-display-app.exe`

## Secure Release Matrix (GitHub Actions)

Workflow:
- `.github/workflows/release.yml`

Artifacts produced:
- Windows: signed MSIX (`.msix`) with Authenticode
- macOS: Developer ID signed + notarized DMG (`.dmg`)
- Linux: Flatpak bundle (`.flatpak`)

Trigger:
- Push a tag like `v0.2.0`, or run the workflow manually.

Release configuration and required secrets:
- `docs/release-matrix.md`

## Windows Inno Installer (Optional Legacy)

If you still want an Inno `.exe` installer:

Prerequisite: Inno Setup 6 (`iscc` in `PATH`)

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\build-installer.ps1
```

Output:
- `dist\rustdesk-display-helper-setup.exe`

## ScreenRes Integration

ScreenRes is vendored in:
- `third_party/screenres`

License and attribution:
- `third_party/screenres/LICENSE`
- `THIRD_PARTY_NOTICES.md`

Resolution tool discovery order:
1. `resolution_tool_path` in config
2. `tools\screenres.exe`
3. `third_party\screenres\screenres.exe`
4. `screenres.exe` from `PATH`

If none exist, the app still applies `DisplaySwitch.exe` and prints a warning.
If a configured resolution is unsupported by your display, the app/CLI now suggests nearest supported resolutions.

Scale options per profile:
- `scale_use_windows_default = true` to return to Windows recommended scale
- `scale_percent = 100` (or other 100..500) for custom scale
- Do not set both at the same time

## Notes

- Windows-focused.
- DPI scaling changes are intentionally not automated because they commonly require sign-out.
