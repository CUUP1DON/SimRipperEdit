# TS4 SimRipper Startup splash + progress + ETA

## Startup splash + progress + ETA
- Added/expanded the startup splash form to show:
  - A progress bar
  - Current stage text (e.g., “Scanning mod packages”)
  - The current package filename being processed
  - Estimated time remaining (ETA) once enough samples exist
- Wired real progress updates into startup package scanning so the bar reflects actual work being done during load.

## Splash layout polish
- Reworked the splash layout to avoid overlapping UI (separated the banner/image area from the progress/status area).
- Kept the splash responsive during long scans.

## Package scanning robustness + diagnostics
- Improved “Unable to open packages” diagnostics to include more actionable information (exception details / likely cause).
- Reworked package resource indexing to avoid calling s4pi `FindAll(...)` during startup indexing.
  - Instead, iterates `package.GetResourceList` once per package and filters resource types.
  - Goal: reduce failures on some large/odd packages (e.g., HQ mod packages) and speed indexing.

## Build fix
- Fixed a compilation error introduced during the indexing refactor (local function captured `resourceTypesToIndex` before it was declared) by moving the `resourceTypesToIndex` declaration above the local function.

## Files touched (high level)
- TS4 SimRipper/src/Form1.cs
- TS4 SimRipper/src/GameFileHandler.cs
- TS4 SimRipper/src/StartMessage.cs
- TS4 SimRipper/src/StartMessage.Designer.cs
- TS4 SimRipper/src/Program.cs (investigated; may have been touched earlier in session)
