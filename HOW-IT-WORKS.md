# How eqcast works

A general overview of what eqcast is made of and how the pieces cooperate. It is deliberately not exhaustive. For setup and day-to-day use see the [user's guide](USER-GUIDE.md).

## The problem

You want to play one EverQuest client and have the same keys (and sometimes the mouse) applied to several others, at the same time, without those other clients ever coming to the front and without locking you out of your own keyboard.

Three facts about EverQuest shape the whole design (all measured on real clients):

1. **It ignores posted window messages.** Sending `WM_KEYDOWN` and friends to a client does nothing, in the foreground or the background.
2. **It reads the keyboard through DirectInput, and only polls while it believes it is the foreground window.** Each frame it asks Windows `GetForegroundWindow()`; if the answer is not its own window, it stops looking at the keyboard. It reads Shift, Ctrl and Alt separately, once per frame, through `GetAsyncKeyState`.
3. **Real input only works for the real foreground window.** The "honest" approach of bringing each client to the front, sending the keys and switching back works, but costs about 0.6 s per client, which is too slow for combat and blocks your own input while it runs.

So eqcast does not fight these facts from outside. It puts a small piece of code *inside* each client that makes it believe it is in the foreground and feeds it the keys directly where it already looks for them.

## The three programs

| Program | Runs as | Its job |
|---|---|---|
| **Launcher** `eqcast-launcher.exe` | The window you open. | Defines clients, accounts, character groups, keymaps and layouts. Starts the clients, places them, logs them in, and owns the controller's lifetime. |
| **Controller** `eqcast.exe` | A hidden child of the launcher. | Watches your keyboard and mouse, decides what to repeat, and hands it to every follower. Also draws the lights, and swaps windows when you click a small one. |
| **Shim** `eqcast_shim.dll` (+ the injector `shimhost.exe`) | A DLL living inside each `eqgame.exe`. | Makes a background client accept the injected keys and mouse, and keeps its window borderless, scaled and where it belongs. |

```
 your keyboard / mouse
        |  global hooks (observe; swallow only the toggle keys and the swap click)
        v
  CONTROLLER ---- decides which keys/mouse events to repeat, and when
        |
        |  one shared-memory block per client:  "held keys", "key press counters",
        |  mouse position / buttons / wheel, window commands, a heartbeat
        v
  SHIM inside each follower's eqgame.exe ---> EverQuest reads them as if typed
        ^
        |  loaded by shimhost.exe, started by
  LAUNCHER ---- starts clients, places windows, types passwords,
                starts / stops the controller, tells it the active keymap
```

## The launcher: getting six clients on the screen

The launcher's work is mostly *preparation*; it is not in the path of a single keystroke.

1. **Plan.** From your group (which characters, on which clients, with which accounts) and your layout (a grid of slots on a monitor) it works out, for each character, what to run and which rectangle of the screen it should occupy.
2. **Start one client at a time.** For each character, in slot order: patch the client folder's `eqclient.ini` so EverQuest creates its window in that slot, start `eqgame.exe` (with `/login:<account>` if the character has an account), wait for its window and load the shim into it. Clients are started strictly one after another because characters sharing one EverQuest folder share one `eqclient.ini`; the launcher must be sure each client has read its own slot before the next one patches the file. Window placement and settling run in the background, so a six-client launch takes about half a minute rather than a minute.
3. **Windows.** Each client is told its slot through its environment. The shim then forces the window borderless and into the slot from the inside, and keeps it there through the login screen and character select, where EverQuest tends to rebuild its window.
4. **Followers are scaled, not small.** Every client renders at the size of the main slot; the shim shows a follower scaled down in its own slot, the way a video player shows a video in a small window. It is the real, live window, so clicking in it works, and every client has the same UI geometry.
5. **Login.** At the login screen the launcher types the account's password (reading the window's own content to see when the screen is up, so another window on top does not matter) and, with Auto or the *Login all* / *Quick connect all* buttons, presses LOGIN or QUICK CONNECT with real mouse input. It stops there: choosing a character and entering the world is yours.
6. **The keymap.** Before starting any client, the launcher writes the chosen group's keymap to `active-keymap.json`. The controller polls that file about once a second and switches keymaps without a restart.

The launcher never closes or kills a client, never launches through Inner Space, and never touches clients it did not start.

## The controller: deciding what to repeat

The controller runs in a mode where the launcher is its parent: it starts when the launcher opens, exits when the launcher closes (or notices a launcher that crashed and exits by itself, releasing every key it was holding). It does not load the shim itself; it only serves clients whose shim has reported in.

### Seeing your input

Two Windows low-level hooks (keyboard and mouse) run on dedicated high-priority threads. They only *observe*: the real key always reaches the client you are using. The hooks swallow exactly two things: the toggle hotkeys and the click that swaps windows. Everything else is queued to a single worker thread that owns the state.

### Main and followers

The **main** is whichever EverQuest client currently owns the real foreground window; every other client is a **follower**. Clicking a different window changes the main: the controller flushes the followers, and the keys then come from the new main.

### Two modes

| Mode | Repeats | Default hotkey |
|---|---|---|
| **Keymap** | only the keys and chords in the group's keymap, each with exactly its modifiers | ScrollLock |
| **Broadcast** | every key, plus the mouse (position, clicks, wheel, right-button mouselook) | Alt+Shift+R |

They are independent. If both are on, broadcast wins and the keymap waits underneath. A modifier is never sent on its own in keymap mode, only together with the key of the chord that needed it.

### Shaping the timeline

Delivering a keystroke to a background client is not just "write the key down". Each follower has its own timeline shaper that:

- sends a chord's modifiers first and the key a short **lead** later, because EverQuest samples Shift, Ctrl and Alt only once per frame. The lead is about 2.5 times that follower's own keyboard-poll interval, measured live;
- holds every press for a minimum time (120 ms by default) and spaces repeated taps, because a follower only looks at the keyboard at its own polls, and would otherwise merge two quick taps into one long hold;
- for the mouse, moves the follower's cursor to the right spot first, then sends the click.

The cursor position is sent as a fraction of the main window's client area, so followers click the same *relative* spot. That is why every client should have the same UI layout.

### Click to swap

Every client has a **home slot**. A left click on a small follower window brings that client into the main slot and moves the main into the slot the clicked client left; everyone else stays at home. The click is consumed so the game never sees it. Only genuine user clicks count, not clicks the launcher synthesises while pressing LOGIN.

### The lights and the rings

Two small always-on-top windows show KEYMAP and BROADCAST state (and can be clicked to toggle). While the mouse is being broadcast, a small click-through ring is drawn on each follower at the spot where its mirrored cursor is.

## The shim: making a background client listen

The shim is a DLL loaded into each `eqgame.exe` by the injector `shimhost.exe`. It binds the client's shared-memory block and hooks a handful of calls:

| What EverQuest does | What the shim answers |
|---|---|
| `GetForegroundWindow()`, many times a second | "You are." (only while the controller's heartbeat is fresh and a mode is on). The client now keeps polling the keyboard though it is in the background. |
| DirectInput keyboard read (`GetDeviceData`) | The real data, merged with the injected keys as ordinary press and release events. A device that is not acquired is reported as fine so the client keeps reading. |
| `GetAsyncKeyState` for Shift, Ctrl, Alt | The injected modifier state. |
| `GetCursorPos`, `SetCursorPos`, `WindowFromPoint`, DirectInput mouse | The injected cursor, buttons, wheel and relative motion. |
| Window style, move, size and maximise calls | Its own, so the window stays borderless, in its slot, and scaled. |

It also publishes back to the controller how often the client polls the keyboard (used for the lead above), the client's window state and its home slot (so a restarted controller knows where everybody belongs).

### The shared-memory block

The controller creates one named block per client, `Local\eqcast.keys.<client pid>`. Conceptually it holds: for each key, whether it is held and a press counter (so a very short tap cannot be lost between two polls of the client); the mouse fields; any window command; flags telling the shim which kinds of answers to give; and a heartbeat the controller refreshes constantly. The format is a documented contract between the two programs.

### Fail-safes

- The shim **fails open**: any fault passes the real call straight through.
- If the heartbeat is more than half a second old (the controller died or hung), the shim stops answering "foreground", releases every injected key and button, and the client behaves as if eqcast were not there.
- When a mode is switched off the controller releases everything through the timeline first, and keeps the flags up for a brief dwell (100 to 400 ms) so each follower can see the releases before it stops listening.

## Lifetime, in one picture

```
open launcher ---> controller starts (hidden), scans for eqgame every second
press Launch ---> keymap published; clients start one at a time; shim loads into each;
                  controller finds each client, makes its block, "shim attached", "client ready"
you play      ---> hooks -> worker -> timeline -> block -> shim -> EverQuest
close launcher --> controller asked to quit: keys released, flags cleared; ended after 3 s if it hangs
```

## What eqcast does *not* do

- It does not pick your character or enter the world.
- It does not mirror a left-button *drag*, or the horizontal wheel.
- It does not inject into, or control, clients started by other programs: a client started by hand gets no shim and no mirroring.
- It does not touch ISBoxer-managed clients: they are skipped on purpose.
- It cannot rescue a key-up that Windows never delivered to the hooks (a UAC prompt, Win+L): tap the key in that client.

## The one-sentence summary

The launcher prepares and starts the clients, the controller watches you and writes what to repeat into one shared-memory block per client, and a shim inside each client makes it believe it is in front and reads that block where it already looks for the keyboard and mouse.
