# eqcast user's guide

How to install eqcast, set it up tab by tab, and use it day to day. This guide matches eqcast 1.6.x. For what is going on underneath, see [HOW-IT-WORKS.md](HOW-IT-WORKS.md).

**What eqcast does, in one paragraph.** You start several EverQuest clients from one window, each placed in its own spot on your screen and logged in. You then play in one client (the *main*), and the keys you press (and, optionally, your mouse) are repeated in all the others (the *followers*) at the same time, while the followers stay in the Windows background and your own keyboard is never locked out.

## Contents

1. [Before you start](#1-before-you-start)
2. [Install](#2-install)
3. [Setup, tab by tab](#3-setup-tab-by-tab)
4. [Launching and playing](#4-launching-and-playing)
5. [Layouts](#5-layouts)
6. [Where eqcast keeps things](#6-where-eqcast-keeps-things)
7. [When something goes wrong](#7-when-something-goes-wrong)

---

## 1. Before you start

- **Windows 10 or 11, 64-bit.** Nothing else to install: the .NET runtime is included.
- **Plain EverQuest folders.** eqcast starts `eqgame.exe` itself. It is not meant to be used with clients that ISBoxer / Inner Space manages (titles `is1` to `is6`); it ignores those windows. If you have both installed, define eqcast clients only for your plain EverQuest folders.
- **Same privilege level.** If you run EverQuest as administrator, run eqcast as administrator too.
- **Your EverQuest accounts.** Optional, but needed for automatic login: one name and password per account.
- **Use at your own risk.** eqcast is not made or endorsed by Daybreak. Sending input to background clients is what ISBoxer does too, but whether it is acceptable for your account is your call.

## 2. Install

You get one of two files:

| File | What it is |
|---|---|
| `eqcast-setup-<version>.exe` | The installer. No administrator rights needed; installs for your Windows user into `%LOCALAPPDATA%\Programs\eqcast`, adds "eqcast launcher" to the Start menu and offers a desktop shortcut. Run a newer installer to upgrade (close your EverQuest clients first; it refuses to continue while eqcast-started clients are running). |
| `eqcast-<version>-win-x64.zip` | The portable version. Unzip anywhere you can write and run `eqcast-launcher.exe`. |

**Windows may warn you.** eqcast is not code-signed yet, so SmartScreen may say "Windows protected your PC": choose **More info**, then **Run anyway**. Some antivirus programs also complain, because eqcast loads a small library into the game client to feed it keys; that is what it is for.

Start **eqcast launcher**. Your settings are kept in `%USERPROFILE%\.eqcast` (see [section 6](#6-where-eqcast-keeps-things)); upgrading keeps them, and uninstalling asks whether to remove them.

## 3. Setup, tab by tab

The launcher window has six tabs: **Launch, Groups, Accounts, Keymaps, Clients, Settings**. You use Launch almost every time, but the order to *set things up* is different:

> **Clients → Accounts → Groups → (Keymaps) → (Settings) → Launch**

The tabs in brackets are optional on the first run. Changes on the Accounts, Keymaps and Settings tabs are held as a draft until you press **Save** on that tab (the launcher also saves when you close it and before every launch); **Revert** throws the draft away.

### 3.1 Clients: tell eqcast where EverQuest is

A *client* is one EverQuest folder, that is, one `eqgame.exe`. Several characters can share one client; you only need a second client if you have a second EverQuest installation.

1. Press **Detect...** and pick a folder such as `C:\Games`. eqcast lists every `eqgame.exe` it finds one folder level below it and offers to add them. (Or press **Add**, then **Browse...** next to **Exe** and pick `eqgame.exe` yourself.)
2. Select the client and check its fields:

| Field | What it does |
|---|---|
| **Name** | A label you choose; groups refer to it. |
| **Exe** | Full path of `eqgame.exe`. Its folder is the working folder. |
| **Arguments** | Arguments given to every character started with this client. Default `patchme`, which is what the EverQuest launcher passes. |
| **Login argument** | Default `/login:{account}`. Added only for a character that has an account; `{account}` is replaced by the account name. This makes the client skip the splash screens and land on the login screen with the account name filled in. |
| **Apply layout to eqclient.ini** | On by default. Before each start, eqcast writes the window position and size of that character's slot into the client's `eqclient.ini` so EverQuest creates its window in the right place. The first time the file changes, a backup is made as `eqclient.ini.eqcast.bak` next to it. Leave this on. |
| **Borderless window (no title bar)** | On by default. Removes the title bar and frame so each window is exactly its slot and the tiles do not overlap. |

**Remove** and **Duplicate** do what they say. A problem (missing exe, duplicate name) is listed under the fields.

### 3.2 Accounts: names and passwords for automatic login

Skip this tab if you are happy to type each password yourself.

1. Press **Add**. Give the account its EverQuest **login name** (no spaces or double quotes; names are unique, ignoring case).
2. Type its **password**. The box is masked; **Show password** reveals it for a sanity check. The tab also tells you which characters use the account.
3. Press **Save** (the status next to it says `Saved` or `Unsaved changes`).

> **The passwords are stored as plain text** in `accounts.json` in `%USERPROFILE%\.eqcast`, on your own computer. Anyone who can read your user files can read them. The launcher's own `launcher.json` holds no passwords.

Renaming an account renames it for every character that uses it. Removing one that characters use asks first.

### 3.3 Groups: which characters, on which clients, in which order

A *group* is the set of characters you launch together: 1 to 6 of them.

1. Press **Add** under the group list and give the group a **Group name**.
2. Press **Add** under the character list for each character, then fill in:

| Field | What it does |
|---|---|
| **Name** | A label for the character. eqcast does not choose the character on the character-select screen; you do that in the game. |
| **Client** | Which client (EverQuest folder) starts this character. |
| **Account** | The account to log in with: `(no login)` or one of your accounts. Save the Accounts tab first so the new account shows in the list. A character with no account still starts; you type everything yourself. |
| **Extra arguments** | Optional, appended last. `{character}` and `{account}` are replaced. |

3. **Move up / Move down** set the order. **The first character is the main** and takes slot 1, the big window of a layout. The others become the followers, in slot order.
4. Pick the group's **Keymap** from the drop-down under the group name (see 3.4). Without a choice it uses the default keymap.

One EverQuest account normally cannot be logged in twice at once, so give each character its own account (the Launch tab warns you if two slots share one).

### 3.4 Keymaps: which keys are repeated in the followers

A *keymap* is a named list of keys. In **keymap mode** (the normal one) only these keys are repeated in the followers; everything else you type stays in the main. That keeps chat, movement and window dragging local.

The built-in keymap is called `default`. It covers the number keys `0`, `4`, `5`, `9`, the keypad (`Num0`..`Num9`, `NumEnter`, `Num+`, `NumDot`) and the chords `Alt+S`, `Alt+D`, `Shift+F`, `Shift+G`, `Ctrl+1`, `Ctrl+2`. Change it to match the hotbar buttons and keys you actually use.

- **Add / Duplicate / Remove / Make default** manage the list. Give a keymap a name and a description.
- **Add a key** by typing its name and pressing Enter, or press **Capture keys** and press the keys themselves: hold Ctrl, Alt or Shift to capture a chord such as `Alt+S`. Esc ends the capture.
- A plain key is repeated only while **no** modifier is held; a chord only while **exactly** those modifiers are held. So a plain `S` typed in chat is not repeated, but `Alt+S` is.
- A modifier on its own (`LShift`) cannot be an entry; use it inside a chord. The two toggle hotkeys (see 3.5) cannot be entries either, and the tab says why when you try.
- Keypad keys need **NumLock on**.
- **Import from ISBoxer...** copies the keys one of your ISBoxer key maps broadcasts into a new keymap (only the keys and chords; menus, repeaters, window actions and remapped keys are left out and counted). It tells you if ISBoxer is not installed.
- Press **Save**. The chosen group's keymap is handed to the controller each time you launch; the controller picks it up without a restart.

### 3.5 Settings: hotkeys, lights, behaviour

Everything here can be left alone on the first run. **Save** applies the change (the controller restarts; mirroring pauses for a second or two). **Revert** and **Restore defaults** do what they say; problems are listed under the controls and block Save until fixed.

| Section | Setting | Default | What it does |
|---|---|---|---|
| **Hotkeys** | Keymap on/off | `ScrollLock` | Turns keymap mode on and off. |
| | Broadcast on/off | `Alt+Shift+R` | Turns broadcast mode (every key and the mouse) on and off. Use **Change...** to capture a new key or chord, **Reset** to go back. |
| **Appearance** | Theme | Dark | Dark, Light or Follow Windows; applies after a restart of the launcher. |
| **Monitor** | Game monitor | each layout's own | Shown only with two or more monitors. Picks the monitor the clients launch on and where the lights appear. |
| **Indicator** | Show the KEYMAP and BROADCAST lights | on | Two small always-on-top lights. |
| | Click the lights to switch the modes | on | Clicking a light does what its hotkey does. |
| | Position | Top right | Which corner of the game monitor. |
| **Behaviour** | Keymap on when the controller starts | on | Keymap mode is live as soon as the clients are found. Broadcast always starts off. |
| | Swap windows by clicking a small one | on | See [section 4](#4-launching-and-playing). |
| | Mirror the mouse | on | Lets broadcast mode repeat mouse position, clicks, wheel and mouselook. |
| | Show the green rings on the followers | on | A small ring in each follower where its mirrored cursor is. |
| **Login** | Log in automatically after launching | on | Types each account's password at the login screen. |

### 3.6 Launch: the tab you use every time

This is the tab the window opens on.

- **Group** and **Layout** drop-downs choose what to start (layouts: [section 5](#5-layouts)). A drawing of the layout shows each character's name in its slot; the plan and any warnings update as you change either one.
- The table beside it lists the slots with an **Account** drop-down per character. You can pick a different account for a launch without editing the group; the choice is not saved.
- **Launch** starts the group. **Cancel** stops after the current step (clients already started keep running).
- **Login all** presses LOGIN, and **Quick connect all** presses QUICK CONNECT, on every client whose password was typed. **Auto** (ticked on every start) does a Quick connect for you when the launch finishes.
- The log underneath narrates the launch. The status line at the bottom shows the controller's state.

---

## 4. Launching and playing

### A normal session

1. Open **eqcast launcher**. It starts the controller by itself, hidden; closing the launcher stops it.
2. On **Launch**, choose the group and layout and press **Launch**.
3. eqcast starts the clients one after another in slot order. It places each window, types each password at the login screen and (with **Auto**) presses QUICK CONNECT. A six-client launch takes about half a minute. **Keep your hands off the keyboard and mouse while it types and clicks.**
4. In each client, pick your character and enter the world yourself.

Starting a group twice starts a second set of clients; Launch does not check for ones that are already running.

### Playing

- **Keymap mode (ScrollLock):** the keys in your keymap are repeated in all followers. The KEYMAP light shows when it is on.
- **Broadcast mode (Alt+Shift+R):** *every* key and the mouse are repeated: position, clicks, the wheel and right-button mouselook. **This includes chat typing**, so switch it off before you type to a friend. The BROADCAST light shows when it is on. While it is on, each follower shows a green ring where its cursor is.
- Both can be on at once; broadcast then wins and the keymap carries on when it goes off.
- **Which client is the main?** The one you are using. Click another window and it becomes the main; the keys then come from that one.
- **Swap by clicking.** With the layout's big main window and small follower windows, a left click on a small window swaps it into the main slot, and the main moves into the slot it left. The click itself is not passed to the game. Everyone else stays in their own slot.
- **Relaunch one client.** If a client closes or crashes, press the small circular-arrow button in its slot on the Launch tab. It starts just that character again, at the login screen with the account name filled in. You type the password and log in yourself.
- **Lights.** Click a light to switch its mode, exactly as the hotkey does.

Things that look odd but are normal: the followers run at the same size as the main and are shown scaled down in their slot, so every client has the same UI layout, and a small window is the real, live client. Mouse mirroring works from relative position in the window, so the EverQuest UI layout of every client should match.

---

## 5. Layouts

A layout is a grid of slots on a monitor. Slot 1 belongs to the first character of the group (the main). eqcast ships eight:

| Layout | Shape |
|---|---|
| `main-2x2-and-five` (default) | 3x3 grid: main in the top-left 2x2 block, five followers around it |
| `six-equal-3x2`, `four-equal-2x2` | equal tiles |
| `wide-1` .. `wide-5` | a wide main across the top and 1 to 5 small followers along the bottom row |

There is no layout editor. `layouts.json` (next to `launcher.json`, see below) can be edited by hand: a layout names its monitor (`"primary"`, an index, or an exact rectangle), a `grid` (columns, rows) and its `slots` (column, row, and optional `colSpan` / `rowSpan`). After editing, use **File > Open layouts file** and **File > Reload layouts**. A group of N characters needs a layout with at least N slots.

---

## 6. Where eqcast keeps things

Everything is in `%USERPROFILE%\.eqcast` unless noted.

| File | What it holds |
|---|---|
| `launcher.json` | clients, groups, remembered group and layout, timings. Written by the app; close the launcher before hand-editing it. |
| `accounts.json` | account names and **plain-text passwords** (Accounts tab). |
| `keymaps.json` | the keymaps (Keymaps tab). |
| `layouts.json` | the layouts (edited by hand). |
| `controller-settings.json` | the Settings tab's hotkeys, lights and behaviour. |
| `active-keymap.json` | the keymap of the last launched group, read by the controller. |
| `controller.log` | what the controller is doing; first place to look for problems. |
| `eqclient.ini.eqcast.bak` | one-time backup next to each EverQuest `eqclient.ini` that eqcast changed. |

To undo eqcast's changes to a client's `eqclient.ini`, close the client and copy the `.bak` file over `eqclient.ini`. The backup is from before eqcast's first edit, so it also reverts any EverQuest options you changed since.

The **File** menu has Save, Open/Reload layouts file, Open/Reload keymaps file, **Restart controller** and Exit. The version is under **Help > About**.

---

## 7. When something goes wrong

Look at the Launch tab's log, the status line at the bottom of the window and `controller.log`.

| Symptom | Likely cause and fix |
|---|---|
| Nothing is mirrored | Keymap mode is off (KEYMAP light muted: press ScrollLock), or the key is not in the group's keymap, or no EverQuest window has focus. The client must have been started by the launcher: a client started by hand gets no mirroring. |
| A chord does not repeat | Chords need exactly their modifiers. Windows may use Alt+Shift to switch the input language; pick another broadcast hotkey in Settings if that bothers you. |
| Keypad keys are silent | NumLock must be on. |
| Mouse and mouselook do nothing | They work only in broadcast mode, and mouselook only while the right button is held. |
| Status line: `Controller: not started (... shim files were not found ...)` | The files next to the launcher are incomplete: reinstall or unzip again. |
| `password NOT typed (no password stored for account ...)` | The account has no password on the Accounts tab; the client still started, type it yourself. |
| Client shows the EULA instead of the login screen | The character has no account, so no `/login:` was passed. |
| QUICK CONNECT lands on server select | That account has no remembered server; pick the server by hand once. |
| `started, not placed (...)` | The window did not settle in its slot. Try Launch again; leave **Apply layout to eqclient.ini** on. |
| A follower keeps pressing a key | A key-up was missed (clicking away while holding a key, a UAC prompt, Win+L). Tap that key in that client. |
| Launch is disabled | `layouts.json` is broken: fix it, then **File > Reload layouts**. |
| Launcher refuses to start with a message about `launcher.json` | It contains an unknown or misspelt setting name. Unknown names are errors, not ignored; the message names the place. |

After upgrading eqcast, restart your EverQuest clients so they pick up the new shim file.
