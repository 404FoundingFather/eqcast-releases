# How eqcast works

This is a tour of what eqcast is made of and how the pieces fit together. It's a tour, not a specification, so some detail is left out on purpose. If you want to set it up and use it, the [user's guide](USER-GUIDE.md) is the better place to start.

## The problem

If you play several EverQuest characters at once, you know the routine. You press a key in one window and you want the others to press it too: assist, target, heal, hold the follow key. You also want that without the other windows jumping to the front, and without losing your own keyboard while it happens.

EverQuest makes this harder than it sounds. We measured three things on real clients, and they shaped the whole design:

1. **It ignores posted window messages.** Post a keyboard message such as `WM_KEYDOWN` to a client and nothing happens, whether the client is in the foreground or the background.
2. **It only listens to the keyboard when it thinks it's in front.** It reads the keyboard through DirectInput, and every frame it asks Windows `GetForegroundWindow()`. If the answer isn't its own window, it stops looking. Shift, Ctrl and Alt are a separate case: it reads those once per frame through `GetAsyncKeyState`.
3. **Real input only works on the real foreground window.** The brute-force answer is to bring each client to the front, send the keys, and switch back. That works, but it costs about 0.6 s per client, which is too slow for combat, and your own typing gets in the way while it runs.

So eqcast doesn't try to push keys at the game from outside. Instead, it attaches a small piece of code to each client. That code answers "are you in front?" with "yes", and it hands the client its keys at the spot where the game already looks for them.

## The three programs

| Program | What it is | What it does |
|---|---|---|
| **Launcher** `eqcast-launcher.exe` | The window you open. | Holds your clients, accounts, character groups, keymaps and layouts. Starts the clients, puts them where they belong, logs them in, and looks after the controller. |
| **Controller** `eqcast.exe` | A hidden child of the launcher. | Watches your keyboard and mouse, decides what to repeat, and passes it on to every follower. Also draws the status lights, and swaps windows when you click a small one. |
| **Shim** `eqcast_shim.dll` (loaded by `shimhost.exe`) | A DLL that lives inside each `eqgame.exe`. | Lets a background client accept the keys and mouse that eqcast sends it, and keeps its window borderless, scaled and in its slot. |

A "shim" is just a thin layer that sits between two things and adjusts what passes through. This one sits between EverQuest and a handful of Windows calls.

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

## The launcher: getting six clients onto one screen

Most of what the launcher does is preparation. It never sits in the path of a keystroke. Here's what happens when you press Launch:

1. **Plan.** From your group (which characters, on which clients, with which accounts) and your layout (a grid of slots on a monitor), the launcher works out what to run for each character and which rectangle of the screen it should end up in.
2. **Start the clients, one at a time.** For each character, in slot order, the launcher patches the client folder's `eqclient.ini` so EverQuest creates its window in that slot. It then starts `eqgame.exe` (with `/login:<account>` if the character has an account), waits for the window, and loads the shim into it. Doing this one client at a time matters: characters that share an EverQuest folder also share one `eqclient.ini`, so the launcher has to be sure a client has read its own slot before the next one rewrites the file. Window placement and settling happen in the background, which brings a six-client launch down to about half a minute instead of a minute.
3. **Window placement.** Each client learns its slot through its environment. The shim then makes the window borderless and puts it in the slot from the inside. It keeps it there through the login screen and character select, where EverQuest likes to rebuild its window.
4. **Followers are scaled, not small.** Every client renders at the size of the main slot, and the shim shows a follower scaled down in its own slot, the way a video player shows a video in a small window. It's still the real, live window, so you can click in it, and every client has the same UI geometry.
5. **Login.** At the login screen, the launcher types the account's password. It reads the window's own content to know when the screen is up, so another window sitting on top doesn't throw it off. When Auto is on, or when you use the *Login all* or *Quick connect all* buttons, it then presses LOGIN or QUICK CONNECT with real mouse input. It stops there. Choosing a character and entering the world is up to you.
6. **The keymap.** Before starting any client, the launcher writes the chosen group's keymap to `active-keymap.json`. The controller checks that file about once a second, so keymaps switch without a restart.

The launcher never closes or kills a client, never launches through Inner Space (the platform ISBoxer runs on), and never touches a client it didn't start.

## The controller: deciding what to repeat

The controller runs as a child of the launcher. It starts when the launcher opens and exits when the launcher closes. If the launcher crashes, the controller notices, releases every key it was holding, and exits on its own. It doesn't load the shim itself, and it only serves clients whose shim has reported in.

### Watching your input

Two low-level Windows hooks, one for the keyboard and one for the mouse, run on dedicated high-priority threads. They mostly just observe: the real key always reaches the client you're using. The only things they swallow are the toggle hotkeys and the click that swaps windows. Everything else goes onto a queue for a single worker thread, which owns all the state.

### Main and followers

The **main** is whichever EverQuest client currently owns the real foreground window. Every other client is a **follower**. Click a different window and that one becomes the main: the controller releases whatever the followers were holding, and the keys now come from the new main. A client whose slot you switched OFF in the launcher is left out entirely: the launcher writes its process id to a small file that the controller reads once a second, and from then on nothing is relayed to it until you switch it back on.

### Two modes

| Mode | Repeats | Default hotkey |
|---|---|---|
| **Keymap** | Only the keys and chords in the group's keymap, each with exactly its own modifiers. | ScrollLock |
| **Broadcast** | Every key, plus the mouse (position, clicks, wheel, right-button mouselook). | Alt+Shift+R |

The two modes switch on and off independently. If both are on, broadcast wins and the keymap waits underneath. In keymap mode, a modifier is never sent on its own, only along with the key of the chord that needed it.

### Getting the timing right

Handing a keystroke to a background client takes more care than just writing the key down. Each follower has its own timeline shaper, which does three things:

- **Modifiers first.** It sends a chord's modifiers first, then the key a short *lead* time later, because EverQuest only samples Shift, Ctrl and Alt once per frame. The lead is about 2.5 times that follower's own keyboard-poll interval, measured live.
- **Presses that last.** It holds every press for a minimum time (120 ms by default) and spaces out repeated taps. A follower only looks at the keyboard at its own polls; without this, it would merge two quick taps into one long hold.
- **Mouse in the right place.** For the mouse, it moves the follower's cursor to the right spot first and sends the click afterwards.

The cursor position travels as a fraction of the main window's client area, so every follower clicks the same *relative* spot. That's why all your clients should share the same UI layout.

### Click to swap

Every client has a **home slot**. Left-click a small follower window and that client moves into the main slot, while the old main goes to the slot the clicked client just left. Everyone else stays home. The click is consumed so the game never sees it, and only genuine user clicks count, not the ones the launcher makes while pressing LOGIN.

### The lights and the rings

Two small always-on-top windows show whether KEYMAP and BROADCAST are on, and you can click either one to toggle it. While the mouse is being broadcast, a click-through ring appears on each follower at the spot where its mirrored cursor is.

## The shim: letting a background client listen

`shimhost.exe` loads the shim DLL into each `eqgame.exe`. Once it's there, the shim opens the client's shared-memory block and answers a handful of the Windows calls the client makes:

| What EverQuest asks | What the shim says |
|---|---|
| `GetForegroundWindow()`, many times a second | "You are." This only happens while the controller's heartbeat is fresh and a mode is on. The client keeps polling the keyboard even though it's in the background. |
| DirectInput keyboard read (`GetDeviceData`) | The real data, merged with the relayed keys as ordinary press and release events. A device that isn't acquired is reported as fine, so the client keeps reading. |
| `GetAsyncKeyState` for Shift, Ctrl, Alt | The modifier state eqcast is relaying. |
| `GetCursorPos`, `SetCursorPos`, `WindowFromPoint`, DirectInput mouse | The relayed cursor, buttons, wheel and relative motion. |
| Window style, move, size and maximize calls | The shim's own, so the window stays borderless, in its slot, and scaled. |

The shim also reports back to the controller how often the client polls the keyboard (which is where the lead above comes from), the state of the client's window, and the client's home slot, so a restarted controller knows where everybody belongs.

### The shared-memory block

The controller creates one named block per client, `Local\eqcast.keys.<client pid>`. For each key it holds whether the key is down, plus a press counter, so a very short tap can't slip between two of the client's polls. It also holds the mouse fields, any pending window command, flags that tell the shim which kinds of answers to give, and a heartbeat that the controller keeps refreshing. The layout is a documented contract between the two programs.

### When something goes wrong

eqcast is built to get out of the way:

- **The shim fails open.** If anything goes wrong inside it, the real call goes straight through, untouched.
- **No heartbeat, no help.** If the heartbeat is more than half a second old, because the controller died or hung, the shim stops saying "you're in front" and releases every key and button it was holding. The client then carries on as though eqcast had never been there.
- **Clean goodbyes.** When you switch a mode off, the controller releases everything through the timeline first. It then keeps the flags up for a short dwell (100 to 400 ms) so each follower sees the releases before it stops listening.

## A session, start to finish

Here is a whole session at a glance:

```
open launcher ---> controller starts (hidden), scans for eqgame every second
press Launch ---> keymap published; clients start one at a time; shim loads into each;
                  controller finds each client, makes its block, "shim attached", "client ready"
you play      ---> hooks -> worker -> timeline -> block -> shim -> EverQuest
close launcher --> controller asked to quit: keys released, flags cleared; ended after 3 s if it hangs
```

## What eqcast doesn't do

- It doesn't choose your character or enter the world. That's yours.
- It doesn't mirror a left-button *drag*, or the horizontal wheel.
- It doesn't attach to or control clients started by other programs. A client you start by hand gets no shim and no mirroring.
- It leaves ISBoxer-managed clients alone. They're skipped on purpose.
- It can't rescue a key-up that Windows never delivered to the hooks (a UAC prompt, Win+L). If a key seems stuck, tap it in that client.

## In one breath

The launcher gets the clients started and in place. The controller watches what you do and writes what should be repeated into one shared-memory block per client. A shim inside each client tells the game it's in front, and reads that block at the spot where the game already looks for the keyboard and mouse.
