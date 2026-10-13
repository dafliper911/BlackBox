# BlackBox
A Windows PC diagnostic and system information tool.

Everything stays on your PC. Nothing is uploaded. It does not record keystrokes,
clipboard contents, command lines, browser history, credentials or message
contents.

## What it does

- Samples CPU, memory, disk and network throughput, uptime, battery state and the
  busiest process **names** on a 2-second cadence (configurable).
- Reads notable Windows event-log entries and explains them honestly.
- Classifies how each recording session ended, using evidence, never guesswork:
  `clean`, `bugcheck_evidence`, `unexpected_shutdown_evidence`,
  `system_shutdown_or_restart`, `unclean_stop` (the same codes the original
  Python app wrote, so existing history stays valid).
- Recovers a session that was interrupted by a crash, without inventing data.
- Discovers existing crash-dump files (it never reads, deletes, moves or uploads
  them) and can plan a guided crash-dump configuration change for you to review.
- Exports a redacted diagnostic report (Markdown, JSON, CSV, optional ZIP).
- Enforces a storage limit with retention and incident protection, and reports a
  conflict instead of silently deleting something important.
- Ships a dashboard that fits any screen: it sizes to the desktop area, maximises
  on small displays, offers light and dark themes, and lets you rescale the output
  text (70%–180%) and drag the output panes to size.

## Honest limits

- A classification describes the **evidence found**; it does not prove a root
  cause. A *Kernel-Power 41* event only means the previous shutdown was not clean
  — it is **not** proof of a blue screen and names no cause.
- PC Black Box does not download or run WinDbg and does not analyse dumps for
  you. See [docs/KNOWN-LIMITATIONS.md](docs/KNOWN-LIMITATIONS.md).

## Install

Download one of the artifacts from a release:

- `PCBlackBox-<version>-win-x64-setup.exe` — per-user installer (**no
  administrator rights required** for a normal install).
- `PCBlackBox-<version>-win-x64-portable.zip` — unzip and run
  `PCBlackBox.App.exe`.

Both are **self-contained**: no .NET runtime needs to be installed.

## Data locations

Everything lives in `%LOCALAPPDATA%\PCBlackBox` (override with the
`PCBLACKBOX_DATA_DIR` environment variable):

| File | Purpose |
| --- | --- |
| `samples.db` | metrics, events, sessions (SQLite, WAL) |
| `config.json` | user settings (clamped on load and save) |
| `status.json` | recorder heartbeat / health |
| `control.json` | pause / stop commands (removed once honoured) |
| `logs/` | dashboard and recorder logs |
| `exports/` | exported reports |

## Recording model

The dashboard starts the recorder automatically. When
`PCBlackBox.RecorderHost.exe` sits beside the dashboard (as it does in the
published builds) the recorder runs as a **separate, hidden process**, so a UI
crash never stops recording and the recorder keeps running after the dashboard
closes. If the recorder executable is missing, the dashboard runs the same
recorder **in-process** as a background task. Both modes talk through the same
versioned `status.json` / `control.json` files, so behaviour is identical.

The single-instance lock (`recorder.lock`) prevents two recorders from writing at
once, and a stale lock left by a crashed recorder is reclaimed safely.

## Build from source

Requirements: Windows x64, the .NET SDK 10.0.4xx or newer. Inno Setup 6 is
optional and only needed to produce the installer.

```powershell
# build and test
dotnet build PCBlackBox.slnx -c Release
dotnet test  tests\PCBlackBox.Tests\PCBlackBox.Tests.csproj -c Release

# produce a full release (tests + portable ZIP + installer) in .\dist
powershell -ExecutionPolicy Bypass -File .\scripts\build-release.ps1
```

If Inno Setup is missing, install it with
`winget install JRSoftware.InnoSetup` and re-run the script; otherwise the script
still produces the portable ZIP and prints a warning.

## Repository layout

```
src/
  PCBlackBox.Core        models, classification, IPC, reporting, settings, sanitization
  PCBlackBox.Storage     SQLite store (schema v1 = legacy, v2 = C# additions)
  PCBlackBox.Windows     WMI / event-log / registry / crash-dump probes
  PCBlackBox.Recorder    the recording loop (UI-independent)
  PCBlackBox.RecorderHost console host for the recorder
  PCBlackBox.App         WPF dashboard
tests/PCBlackBox.Tests   xUnit tests (synthetic fixtures only)
installer/               Inno Setup script + portable README
scripts/                 release build script
legacy-python/           the original Python application, kept for reference
docs/                    privacy, limitations, migration notes
```

## Documentation

- [docs/PRIVACY.md](docs/PRIVACY.md) — exactly what is and is not recorded.
- [docs/KNOWN-LIMITATIONS.md](docs/KNOWN-LIMITATIONS.md) — the honest fine print.
- [docs/MIGRATION.md](docs/MIGRATION.md) — Python → C# migration notes.
- [CHANGELOG.md](CHANGELOG.md) — what changed in 2.0.

## Licence

See [LICENSE](LICENSE). Provided as-is, with no warranty; it does not replace a
professional diagnosis.
