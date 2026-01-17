# TS4 SimRipper Friendly Errors + Non-Sticky “Working…” UI - README 8

## Overview
This README documents the changes made to TS4 SimRipper to:

- Replaced the generic WinForms “Continue / Quit” crash-style dialogs with **user-friendly messages**.
- Prevent the UI from getting stuck showing **“Working…”** (and/or a wait cursor) after invalid actions or exceptions.
- Add consistent **precondition checks** so common “invalid clicks” (ex: exporting without a save/sim/model loaded) are handled cleanly.
- Log technical details to a predictable location for troubleshooting.

Previously, those invalid actions would throw exceptions and show a generic .NET dialog that *looks like a crash*, and in some cases the UI would remain in a “Working…”/busy state even after choosing “Continue”.

## High-Level Pattern (“Guard + Scope + Report”)
Most updated handlers now follow this consistent structure:

1) **Guard**: Validate required state before doing work.
2) **Scope**: Wrap the operation in a disposable busy UI scope so cleanup always happens.
3) **Report**: Show a friendly message to the user and write details to a log.

## Changes Made

### 1) Centralized error reporting + logging
**New file**: [`ErrorReporter.cs`](../TS4%20SimRipper/src/ErrorReporter.cs)

Adds a small utility that:
- Shows **friendly** MessageBox text (warnings vs errors).
- Writes exception details to log files under:
  - `%LocalAppData%\TS4SimRipper\logs\TS4SimRipper_yyyyMMdd_HHmmss.log`

Key entry points:
- `ErrorReporter.ShowWarning(...)`
- `ErrorReporter.ShowError(...)`
- `ErrorReporter.ShowUnhandled(...)`

### 2) Guaranteed busy UI cleanup (prevents “Working…” sticking)
**New file**: [`BusyScope.cs`](../TS4%20SimRipper/src/BusyScope.cs)

Adds a disposable scope that temporarily:
- Enables a wait cursor
- Shows `Working_label` (and restores its previous text/state on dispose)

This ensures that even if an exception occurs, the UI is returned to a normal state.

### 3) Global exception handlers (avoid “Continue / Quit” dialogs)
**File**: [`Program.cs`](../TS4%20SimRipper/src/Program.cs)

Registers global handlers so unhandled exceptions are caught and shown using `ErrorReporter`, rather than letting WinForms display the generic crash dialog:

- `Application.ThreadException`
- `AppDomain.CurrentDomain.UnhandledException`
- `TaskScheduler.UnobservedTaskException`

### 4) Main-form precondition helpers and safer selection-change behavior
**File**: [`Form1.cs`](../TS4%20SimRipper/src/Form1.cs)

Added helper methods (used across multiple buttons/handlers):
- `EnsureSaveLoaded(actionTitle)`
- `EnsureSimSelected(actionTitle)`
- `HasAnyModelLoaded()` / `EnsureModelLoaded(actionTitle)`

Updated multiple selection/option-change handlers to:
- Guard null/invalid selections
- Wrap work in `BusyScope`
- Catch exceptions and call `ErrorReporter` so users get a clear message and the UI doesn’t remain stuck.

Also updated several “write file” helper paths (GEOM/OBJ/MS3D/SIMO) to report friendly errors rather than stack-trace-heavy dialogs.

### 5) Export/texture buttons hardened + Troubleshoot ZIP fixed
**File**: [`PreviewControl.cs`](../TS4%20SimRipper/src/PreviewControl.cs)

Key improvements:
- `SaveModelMorph(...)` now validates “model loaded” and uses `BusyScope` + friendly error reporting.
- Texture save handlers now check required state first and show warnings instead of throwing.
- `SimTrouble_button_Click` was repaired (brace/structure fix) and now:
  - Requires a sim selection
  - Runs inside `BusyScope`
  - Produces friendly errors via `ErrorReporter` if zip creation fails

## What Users Will Notice
- Clicking export actions in the wrong state (no save/sim/model) shows a **friendly warning** dialog.
- The **“Working…”** indicator and wait cursor should no longer get stuck after errors.
- If something genuinely breaks, users get a clear message and a log file for debugging.

## Where Logs Go
If an error occurs and you need technical details:

- `%LocalAppData%\TS4SimRipper\logs\`

Each error log file is timestamped.

## Notes / Known Remaining Work
- There are still other parts of the codebase that display stack traces in MessageBoxes (not yet migrated), notably some paths in `GameFileHandler.cs`.
