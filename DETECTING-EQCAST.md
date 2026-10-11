# Detecting eqcast

*For Daybreak's engineering and anti-cheat teams, from the author of eqcast.*

eqcast is a multiboxing tool for EverQuest on Windows. Daybreak's Truebox servers don't allow that kind of play, and I'd like to help you enforce that rule rather than make you work for it. This note lists every way eqcast can be recognised, from the inside of the game client and from the outside, in the order I'd rely on them. It matches eqcast 1.6.x; I'll keep it current.

Two things up front:

- **eqcast does not try to hide.** Every file, process, kernel object and window it creates has "eqcast" in its name. Nothing is packed, renamed, randomised or signed by someone else. If a future version changes any name in this document, the document changes with it.
- **It does not defeat Truebox's own mechanism.** eqcast runs several `eqgame.exe` processes on one computer, in plain sight. It does nothing to hardware identifiers, to the network traffic, or to anything that tells your servers how many clients a machine is running.

## What eqcast does to the client

So that the checks below make sense: eqcast loads a DLL, `eqcast_shim.dll`, into each `eqgame.exe` it starts. That DLL patches a handful of entries in the client's import table and four slots of the DirectInput8 device vtables, so that a background client is told it is the foreground window and reads keyboard and mouse events that eqcast relays to it through a shared-memory block. It also replaces the main window's window procedure to keep the window borderless and in place. It reads no game state and touches no game memory beyond those patches.

## Checks from inside the client

These are the ones I'd build on. Each is a few lines of code and none depends on a file name.

### 1. The shim's mutex

The DLL creates a named mutex in the session-local namespace as soon as it is loaded, to stop a second copy loading:

```
Local\eqcast.shim.<pid>        (<pid> = the client's own process id, decimal)
```

From the client: `OpenMutexW(SYNCHRONIZE, FALSE, L"Local\\eqcast.shim.<own pid>")` succeeds only when the shim is loaded. The mutex lives as long as the DLL does.

### 2. The shared-memory block

The controller creates one named file mapping per client, which the shim opens:

```
Local\eqcast.keys.<pid>
```

Its first sixteen bytes are a header: `uint32 magic = 0x4B435145` (the bytes `E Q C K`), `uint32 version = 1`, `uint32 clientPid` (the client's pid), `uint32 size`. A client can `OpenFileMappingW` its own name and check the magic. The block exists while eqcast's controller is running, even before the shim has loaded.

### 3. Environment variables

The launcher starts each client with these in its environment:

```
EQCAST_SHIM_WINDOW = "x,y,w,h"      (always, for a client eqcast started)
EQCAST_SHIM_SCALE  = "w,h"          (when the client is a scaled follower)
```

`GetEnvironmentVariableW(L"EQCAST_SHIM_WINDOW", ...)` returning anything means eqcast started this process. This check works before the DLL is even loaded.

### 4. Import-table integrity

The shim patches these import entries so they resolve into `eqcast_shim.dll` instead of `user32.dll`:

| Module | Patched imports |
|---|---|
| `eqgame.exe` | `GetForegroundWindow`, `GetKeyState`, `GetAsyncKeyState`, `GetCursorPos`, `SetCursorPos`, `WindowFromPoint`, `SetWindowLongPtrA`, `GetWindowLongPtrA`, `SetWindowPos`, `ShowWindow` |
| `eqmain.dll` (while mapped) | `SetWindowLongPtrA`, `GetWindowLongPtrA`, `SetWindowPos`, `ShowWindow`; plus `GetCursorPos`, `SetCursorPos`, `GetClientRect`, `GetWindowRect` for a scaled follower |

A generic check: for each of those entries, confirm the resolved address lies inside `user32.dll`'s image range. If it points anywhere else, something has patched the import. This catches eqcast under any file name.

### 5. DirectInput vtable integrity

On the keyboard and mouse `IDirectInputDevice8` objects (both the A and W interfaces), four vtable slots are replaced: `Release`, `Acquire`, `GetDeviceState` and `GetDeviceData`. Confirm those slots point inside `dinput8.dll`.

### 6. The window procedure

When the shim manages the window (whenever the client's "Borderless window" setting is on, which is the default), `GetWindowLongPtr(hwnd, GWLP_WNDPROC)` on the main game window points into `eqcast_shim.dll`, not into `eqgame.exe`.

### 7. The module itself

`eqcast_shim.dll` is in the process's module list. It exports one function, `EqcastHookProc`. It carries no version resource and is not code-signed. This is the easiest check and the easiest to evade by renaming a file, which is why I list it last; eqcast will not rename it, but a modified copy could.

### 8. Behaviour

Independent of any name: the patched `GetForegroundWindow` answers the client's own window while eqcast is active, so **several clients on one machine each believe they are the foreground window at the same time**. A direct comparison (for example `GetGUIThreadInfo` with thread id 0, whose `hwndActive`/`hwndFocus` come from a different path, or a check that the window has focus through `GetFocus` on the foreground thread) disagrees with the imported call. Likewise, a background client whose DirectInput keyboard keeps delivering events after it lost the foreground is being fed.

## Checks from outside the client

Useful for a launcher-side or machine-level check:

| What | Where |
|---|---|
| Processes | `eqcast-launcher.exe` (the window the user opens), `eqcast.exe` (the controller, hidden), `shimhost.exe` (runs briefly to load the DLL) |
| Controller single-instance mutex | `Local\eqcast.controller` |
| Controller quit event | `Local\eqcast.controller.quit.<launcher pid>` |
| On-screen lights | A top-level window of class `EqcastIndicator`, title `eqcast` (topmost, tool window) |
| Mouse rings | A hidden timer window titled `eqcast markers`, plus one small topmost layered window per follower while the mouse is mirrored |
| Install folder | `%LOCALAPPDATA%\Programs\eqcast` (installer) or wherever the zip was unpacked |
| Settings folder | `%USERPROFILE%\.eqcast` (`launcher.json`, `accounts.json`, `keymaps.json`, `layouts.json`, `controller-settings.json`, `active-keymap.json`, `controller.log`) |
| Diagnostics folder | `%LOCALAPPDATA%\eqcast` (`shim-<pid>.log`, `flip-ms.txt`, `holdtop.txt`; only present if the user turned logging on or tuned those) |
| Next to the game | `eqclient.ini.eqcast.bak`, a one-time backup made before eqcast first edited `eqclient.ini` (it writes only the window position and size keys) |
| Command line | eqcast passes `patchme` and `/login:<account>`; both are ordinary and not a signal on their own |

## How the DLL gets in

`shimhost.exe` opens the client with `PROCESS_QUERY_INFORMATION | PROCESS_VM_READ` (no write access), finds the client's main window and its thread, and installs a `WH_GETMESSAGE` hook on that thread with `eqcast_shim.dll` as the hook module. Windows loads the DLL on the client's next message. The DLL then pins itself and stays loaded until the client exits. There is no remote thread, no written process memory and no code cave.

## What I'd suggest

If you want one check, use **1 or 2** (the named mutex or the file mapping). They are cheap, they work from inside the client, and eqcast cannot function without them: the mutex is what stops the DLL loading twice, and the file mapping is how every key arrives. If you want a check that would also catch a modified or renamed build, use **4 and 5** (import-table and vtable integrity); they don't depend on anything eqcast chooses to call itself.

I'm happy to keep a stable, documented beacon in the client for you if a specific one would be easier to consume (a named event, a window property, an export), and to tell you ahead of a release if any name in this document is going to change.

*Contact: through the repository this file lives in.*
