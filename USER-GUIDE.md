# eqcast user's guide

eqcast lets you play one EverQuest character and have the keys you press (and, when you want, your mouse) repeated in up to five other clients at the same time. The other windows stay in the background, your keyboard is never locked out, and one window starts, places and logs in the whole team.

This guide matches eqcast 1.6.x. If you want to know what's going on under the hood, read [HOW-IT-WORKS.md](HOW-IT-WORKS.md).

## Contents

1. [Before you start](#1-before-you-start)
2. [Install](#2-install)
3. [Your first launch](#3-your-first-launch)
4. [The tabs](#4-the-tabs)
5. [Playing](#5-playing)
6. [Layouts](#6-layouts)
7. [Where eqcast keeps things](#7-where-eqcast-keeps-things)
8. [When something goes wrong](#8-when-something-goes-wrong)

---

## 1. Before you start

- **Windows 10 or 11, 64-bit.** There's nothing else to install; the .NET runtime is built in.
- **Plain EverQuest folders.** eqcast starts `eqgame.exe` itself. It's not meant for clients that ISBoxer or Inner Space manages (their windows are titled `is1` to `is6`), and it ignores those windows. If you have both, point eqcast only at your plain EverQuest folders.
- **Same privilege level.** If you run EverQuest as administrator, run eqcast as administrator too. Otherwise eqcast can't reach the client.
- **Your account names and passwords**, if you want eqcast to log in for you. You can skip this and type them yourself.
- **Use at your own risk.** eqcast isn't made or endorsed by Daybreak. Sending input to background clients is what ISBoxer does too, but whether it's acceptable for your account is your call.

## 2. Install

You get one of two files from the [Releases](../../releases/latest) page:

| File | What it is |
|---|---|
| `eqcast-setup-<version>.exe` | The installer. It doesn't need administrator rights. It installs for your Windows user into `%LOCALAPPDATA%\Programs\eqcast`, adds **eqcast launcher** to the Start menu, and offers a desktop shortcut. |
| `eqcast-<version>-win-x64.zip` | The portable version. Unzip it anywhere you can write to and run `eqcast-launcher.exe`. |

**Windows may warn you.** eqcast isn't code-signed yet, so SmartScreen may say "Windows protected your PC". Choose **More info**, then **Run anyway**. Some antivirus programs also object, because eqcast loads a small library into the game client to hand it keys. That's the whole point of the program, so you may need to add an exception.

**Upgrading.** Run the newer installer over the old one. Close your EverQuest clients first: while a client that eqcast started is running, the installer stops with "eqcast's files are in use" and asks you to try again. Your settings are kept. After an upgrade, restart your EverQuest clients so they pick up the new version.

**Uninstalling** asks whether to delete your settings and saved passwords too. The default is No.

Your settings live in `%USERPROFILE%\.eqcast` (see [section 7](#7-where-eqcast-keeps-things)).

## 3. Your first launch

The launcher has six tabs: **Launch, Groups, Accounts, Keymaps, Clients, Settings**. You'll live on Launch once you're set up, but the first time through you need four of them, in this order:

**1. Clients tab: tell eqcast where EverQuest is.**
Press **Detect...** and pick the folder that holds your EverQuest folders (for example `C:\Games`). eqcast lists every `eqgame.exe` one level below it; tick the ones you want and press **Add checked**. One client is one EverQuest folder, and several characters can share it. Leave the other fields alone for now.

**2. Accounts tab: add your accounts.** (Skip this if you'd rather type passwords yourself.)
Press **Add**, type the **Account name** exactly as you log in with it, type the **Password**, then press **Save**. Repeat for each account. Passwords are stored as plain text on your own computer; see [4.3](#43-accounts) before you decide.

**3. Groups tab: say who's playing.**
Press **Add** under the group list and give the group a name. Then press **Add** under the character list for each character and set its **Character** name, **Client** and **Account**. The character at the top of the list is your **main**, the one you'll play. Use **Move up** and **Move down** to order the rest.

**4. Launch tab: go.**
Pick your group, leave the layout at `main-2x2-and-five`, and press **Launch**. Then **keep your hands off the keyboard and mouse** while eqcast starts the clients one by one, places them, types the passwords and presses QUICK CONNECT. A six-client launch takes about half a minute. Pick each character and enter the world yourself.

That's it. Press **ScrollLock** in your main window and the keys in the default keymap (number keys, keypad, and a few Alt/Shift/Ctrl chords) start repeating in the others. You'll want to tailor that list on the Keymaps tab soon, but it works out of the box.

## 4. The tabs

### Saving

The Groups and Clients tabs save by themselves when you launch or close the launcher. The Accounts and Keymaps tabs hold your edits as a draft until you press **Save**, though a launch and closing the launcher save them too. The Settings tab is different: only its **Save** button writes anything, and closing with unsaved settings asks you first. **Revert** on any of those tabs throws the draft away.

### 4.1 Launch

The tab the window opens on.

- **Group** and **Layout** pick what to start. The drawing shows each character's name in its slot, and the plan box beside it updates with any warnings as you change either one.
- The slot table lists each character with an **Account** drop-down and a small dot. You can pick a different account for one launch without editing the group; the choice isn't saved and is forgotten when you switch groups or restart the launcher. The dot is green while the client eqcast started for that slot is still running, grey otherwise (hover it for the detail).
- **Launch** starts the group, or rather the part of it that isn't running: slots whose dot is green are skipped, so after a crash you can simply press Launch again and only the missing client starts, password typed and all. If everything is running, Launch does nothing and the log says so. It never starts a second copy of a client. **Cancel** stops after the current step; clients already started keep running.
- **Login all** presses LOGIN and **Quick connect all** presses QUICK CONNECT, on every client whose password eqcast typed. It uses real mouse clicks, one client at a time, so keep your hands off the mouse. A client is never clicked twice; if one didn't take, press the button again and only that one is tried.
- **Auto** is ticked at every start. With it on, eqcast presses QUICK CONNECT for you as soon as the launch finishes.
- **Switch a character OFF.** Click a character's slot in the drawing (anywhere but the little arrow button) and eqcast stops relaying anything to that client: no keymap keys, no broadcast, no mouse. The slot goes dim and says `(OFF)`. Use it when one character has to be somewhere else while the others fight. Click the slot again to turn it back on. The switch sticks to that slot until you click it, even across a relaunch or a restart of the launcher. If you click into an OFF client and play it yourself, its keys still go to the others.
- **Relaunch one client.** Each slot in the drawing has a small circular-arrow button in its top-right corner. If a client crashed or you closed it, press that button. It starts just that character again, in its own slot, at the login screen with the account name filled in. You type the password and log in. While that slot's dot is green the button does nothing (the log says the client is still running): close the client first.
- The log underneath narrates the launch. The status line at the bottom of the window shows what the controller is doing.

eqcast remembers the clients it started, even across a restart of the launcher: open it again while they run and their dots are green right away, and Launch fills in only what is missing. It only knows clients it started itself, though. One you started by hand is invisible to it, and Launch would start another for that slot.

### 4.2 Groups

A group is the set of characters you launch together: one to six of them.

| Field | What it does |
|---|---|
| **Group name** | Whatever you like. |
| **Keymap** | Which keymap this group uses in keymap mode. `(default: ...)` uses whichever keymap is marked default on the Keymaps tab. |
| **Character** | A label for you. eqcast never picks a character on the character-select screen. |
| **Client** | Which EverQuest folder starts this character. |
| **Account** | `(no login)` or one of your saved accounts. Only accounts you've saved on the Accounts tab show up here. A character with no account still starts; you type everything yourself. |
| **Extra arguments** | Optional, added to the command line last. `{character}` and `{account}` are replaced. |

The first character in the list is marked `1 (main)` and takes slot 1, the big window of a layout. The rest are followers, in list order. One EverQuest account can't normally be logged in twice at once, so give each character its own account; the Launch tab warns you if two slots share one.

### 4.3 Accounts

Press **Add**, type the **Account name** (no spaces or quotes; it goes on the command line as `/login:<name>`) and the **Password**. The password box is masked; **Show password** reveals it so you can check your typing. The tab tells you which characters use each account. Press **Save**.

> **Passwords are stored as plain text** in `accounts.json` in `%USERPROFILE%\.eqcast`. Anyone who can read your user files can read them. If that's not acceptable to you, leave the Accounts tab empty and type your passwords at the login screen; everything else still works.

Renaming an account renames it for every character that uses it. Removing one that characters use asks first; those characters then start without a login.

### 4.4 Keymaps

A keymap is a named list of keys. In **keymap mode** (the everyday one), only those keys are repeated in the followers. Everything else you type stays in your main window, so chat, movement and dragging windows around are safe.

The built-in keymap is called `default`. It has the number keys `0`, `4`, `5` and `9`, the whole keypad (`Num0` to `Num9`, `NumEnter`, `Num+`, `NumDot`), and the chords `Alt+S`, `Alt+D`, `Shift+F`, `Shift+G`, `Ctrl+1` and `Ctrl+2`. Change it to match the hotbar keys you actually use.

- **Add**, **Duplicate**, **Remove** and **Make default** manage the list. Each keymap has a name and a description.
- **Adding keys.** Type a key's name in the **Key** box and press Enter, or press **Capture keys** and press the keys themselves. Hold Ctrl, Alt or Shift while you press to capture a chord such as `Alt+S`. Esc ends the capture (to map Esc itself, type `Escape`).
- **How chords work.** A plain key is repeated only while **no** modifier is held. A chord is repeated only while **exactly** those modifiers are held. So a plain `S` typed into chat is never repeated, but `Alt+S` is.
- A modifier on its own (`LShift`) can't be an entry; put it in a chord. The two hotkeys on the Settings tab can't be entries either.
- **Keypad keys need NumLock on.**
- **Import from ISBoxer...** copies the keys one of your ISBoxer key maps broadcasts into a new keymap. Only plain key broadcasts come over; menus, repeaters, window actions and remapped keys are left out, and the dialog says how many. If ISBoxer isn't installed, it tells you and offers to let you find the profile yourself.
- Press **Save**. The group's keymap is handed to the controller at every launch, and again whenever you save this tab or change the launched group's keymap on the Groups tab, so an edit is live in the running clients within about a second. No restart, no relaunch.

### 4.5 Clients

A client is one EverQuest folder. You only need a second client if you have a second EverQuest installation; characters that share a folder share a client.

**Detect...** scans one level below the folder you pick and offers to add each `eqgame.exe` it finds. **Add** then **Browse...** lets you pick one by hand. **Remove** and **Duplicate** do what they say.

| Field | What it does |
|---|---|
| **Name** | A label you choose. Groups refer to it. |
| **Exe** | The full path of `eqgame.exe`. Its folder is the working folder. |
| **Arguments** | Given to every character started with this client. Default `patchme`, which is what the EverQuest launcher passes. |
| **Login argument** | Default `/login:{account}`. Used only for characters that have an account. It makes the client skip the splash screens and land on the login screen with the account name filled in. |
| **Apply layout to eqclient.ini** | On by default. Before each start, eqcast writes the slot's window position and size into the client's `eqclient.ini` so EverQuest opens its window in the right place. The first time it changes the file it saves a backup, `eqclient.ini.eqcast.bak`, next to it. Leave this on. |
| **Borderless window (no title bar)** | On by default. Removes the title bar and frame so each window is exactly its slot and the tiles don't overlap. |

### 4.6 Settings

You can leave all of this alone on the first run. Saving the hotkeys, the game monitor or the indicator and behaviour options restarts the controller, so mirroring pauses for a second or two. **Restore defaults** asks first and still needs a Save.

| Section | Setting | Default | What it does |
|---|---|---|---|
| **Hotkeys** | Keymap on/off | `ScrollLock` | Turns keymap mode on and off. |
| | Broadcast on/off | `Alt+Shift+R` | Turns broadcast mode (every key, plus the mouse) on and off. Press **Change...** and then the new key or chord; **Reset** goes back. A hotkey can't be a key one of your keymaps already uses on its own. |
| **Appearance** | Theme | Dark | Dark, Light, or Follow Windows. Takes effect after you restart the launcher. |
| **Monitor** | Game monitor | each layout's own | Only shown with two or more monitors. Picks where the clients launch and where the lights appear. |
| **Indicator** | Show the KEYMAP and BROADCAST lights | on | Two small always-on-top lights. |
| | Click the lights to switch the modes | on | Clicking a light does what its hotkey does. |
| | Position | Top right | Top left, Top middle, Top right, Bottom left or Bottom right of the game monitor. |
| **Behaviour** | Keymap on when the controller starts | on | Keymap mode is live as soon as the clients are found. Broadcast always starts off. |
| | Swap windows by clicking a small one | on | See [Click to swap](#click-to-swap). |
| | Mirror the mouse | on | Lets broadcast mode repeat the mouse: position, clicks, wheel and mouselook. |
| | Show the green rings on the followers | on | A small ring in each follower where its mirrored cursor is. |
| **Login** | Log in automatically after launching | on | Types each account's password at the login screen. |

---

## 5. Playing

### A normal session

1. Open **eqcast launcher**. It starts the controller by itself, hidden. Closing the launcher stops it.
2. On **Launch**, pick the group and layout and press **Launch**.
3. Hands off the keyboard and mouse until the log says the launch is done.
4. In each client, pick your character and enter the world.
5. Play in the main window. Press **ScrollLock** if the KEYMAP light isn't already on.

### The two modes

| Mode | Hotkey | What's repeated in the followers |
|---|---|---|
| **Keymap** | ScrollLock | Only the keys in the group's keymap. Chat and everything else stay local. This is the mode you'll use most. |
| **Broadcast** | Alt+Shift+R | *Every* key and the mouse: position, clicks, wheel and right-button mouselook. **Chat typing included**, so switch it off before you talk to someone. While it's on, each follower shows a green ring where its cursor is. |

The two are independent. If both are on, broadcast wins, and keymap mode carries on when broadcast goes off. The KEYMAP and BROADCAST lights show which is on, and you can click a light instead of pressing its hotkey.

One thing to know: the **Pause** key is an older hotkey that also toggles broadcast. It still works, so if every key suddenly mirrors and you didn't press Alt+Shift+R, that's probably why.

### Main and followers

The main is simply the client you're using. Click into a different window and that one becomes the main; the keys now come from it and go to the rest.

### Click to swap

With the big-main layouts, a left click on a small follower window swaps it into the main slot, and the old main moves into the slot it left. Everyone else stays put. The click itself never reaches the game. If the character in the main slot logs out, the slot is simply empty: click the main character's small window to bring it back, or another follower to put that one in. This only works on layouts whose slots differ in size; on the equal-tile layouts there's nothing to swap into.

### Things that look odd but are normal

- **Followers are scaled, not small.** Every client renders at the size of the main slot, and a follower is shown scaled down in its own slot, like a video in a small player window. It's the real, live client: you can click in it, and it has exactly the same UI layout as the main.
- **Your EverQuest UI should match across clients.** The mouse is mirrored by relative position in the window, so a hotbar button in the same place in every client gets the same click.
- **A chord lands a fraction later than a plain key.** EverQuest reads Shift, Ctrl and Alt once per frame, so eqcast sends the modifiers first and the key about 40 to 50 ms after. You won't notice in play.

---

## 6. Layouts

A layout is a grid of slots on a monitor. Slot 1 belongs to the first character of the group. eqcast ships eight:

| Layout | Shape |
|---|---|
| `main-2x2-and-five` (default) | 3x3 grid. The main fills the top-left 2x2 block; five followers wrap around it. |
| `six-equal-3x2`, `four-equal-2x2` | Equal tiles. No click-to-swap on these. |
| `wide-1` to `wide-5` | A wide main across the top and one to five small followers along the bottom row. |

A group of N characters needs a layout with at least N slots.

There's no layout editor yet, but `layouts.json` (next to `launcher.json`) is easy to edit by hand. A layout names its monitor (`"primary"`, a monitor number, or an exact rectangle), a `grid` of columns and rows, and its `slots` (each a column and row, with optional `colSpan` and `rowSpan`). After editing, use **File > Open layouts file** and **File > Reload layouts**.

Known quirk: layouts use the whole monitor, so a bottom row can sit under the Windows taskbar. Auto-hiding the taskbar or using a monitor without one avoids it.

---

## 7. Where eqcast keeps things

Everything is in `%USERPROFILE%\.eqcast` unless noted.

| File | What it holds |
|---|---|
| `launcher.json` | Clients, groups, the remembered group and layout, timings. The launcher rewrites it when it saves, so close the launcher before editing it by hand or your edit will vanish. |
| `accounts.json` | Account names and **plain-text passwords**. |
| `keymaps.json` | Your keymaps. |
| `layouts.json` | Your layouts (edited by hand). |
| `controller-settings.json` | The Settings tab's hotkeys, lights and behaviour. |
| `active-keymap.json` | The keymap of the last launched group, read by the controller. Rewritten when you save the Keymaps tab or change that group's keymap. |
| `clients.json` | The clients eqcast started (which slot, which process) and the slots you switched OFF, so the Launch tab is right after a restart. No passwords. |
| `muted-clients.json` | The clients that are switched OFF, for the controller. Rewritten whenever you click a slot. |
| `controller.log` | What the controller is doing. The first place to look when mirroring misbehaves. |
| `eqclient.ini.eqcast.bak` | A one-time backup next to each EverQuest `eqclient.ini` that eqcast changed. |

To undo eqcast's changes to a client's `eqclient.ini`, close that client and copy the `.bak` file over `eqclient.ini`. The backup is from before eqcast's first edit, so it also reverts any EverQuest options you've changed since then.

The **File** menu has Save, Open and Reload for the layouts and keymaps files, **Restart controller**, and Exit. The version is under **Help > About**.

---

## 8. When something goes wrong

Look at the Launch tab's log, the status line at the bottom of the window, and `controller.log`.

### Mirroring

| Symptom | Likely cause and fix |
|---|---|
| Nothing is mirrored | Keymap mode is off (the KEYMAP light is dim: press ScrollLock), the key isn't in the group's keymap, or no EverQuest window has focus. The clients must have been started by the launcher; one you start by hand gets nothing. |
| Everything is mirrored, chat included | Broadcast mode is on. Press Alt+Shift+R, or click the BROADCAST light. The Pause key also toggles it. |
| A chord doesn't repeat | Chords need exactly their modifiers. Windows may grab Alt+Shift to switch the input language; pick another broadcast hotkey in Settings if that bites you. |
| Keypad keys are silent | NumLock must be on. |
| Mouse and mouselook do nothing | They only work in broadcast mode, and mouselook only while the right button is held. |
| A follower keeps pressing a key | A key-up was missed (you clicked away while holding a key, a UAC prompt, Win+L). Tap that key in that client. |
| Clicking a small window doesn't swap | Both **Swap windows by clicking a small one** and **Mirror the mouse** must be on in Settings, and the layout needs a big main (not the equal-tile layouts). |
| A follower stops responding after a fast double-click or two buttons at once | A known defect when two mouse buttons land within about a third of a second. Click into another window or toggle the mode off and on. |
| Nothing changed after upgrading | Restart your EverQuest clients. A client keeps the old version of eqcast's library until it exits. |

### Launching and logging in

| Symptom | Likely cause and fix |
|---|---|
| `password NOT typed (no password stored for account ...)` | That account has no password on the Accounts tab. The client still started; type it yourself. |
| `password NOT typed (the login screen did not appear within 60 s)` | The client was slow to reach the login screen. Type the password yourself, or relaunch that slot. |
| `Passwords are not typed: ...` | `accounts.json` couldn't be read; the message says why. The Accounts tab goes read-only until you fix the file by hand and restart the launcher. |
| The client shows the EULA instead of the login screen | The character has no account, so no `/login:` was passed. Normal; log in by hand. |
| QUICK CONNECT lands on server select | That account has no remembered server. Pick the server by hand once. |
| `started, not placed (...)` | The window didn't settle in its slot. Press Launch again, and keep **Apply layout to eqclient.ini** on. |
| Launch is greyed out | `layouts.json` is broken. Fix it, then **File > Reload layouts**. |
| The Keymaps tab is read-only | `keymaps.json` is broken. Fix it with **File > Open keymaps file**, then **File > Reload keymaps**. |
| The launcher won't start and mentions `launcher.json` | It contains a setting name eqcast doesn't know (a typo, usually). The message names the spot. Unknown names are errors, not ignored. |

### The controller

| Status line says | What to do |
|---|---|
| `Controller: not started (... shim files were not found ...)` | The files next to the launcher are incomplete. Reinstall, or unzip again. |
| `Controller: stopped (exit code N; see controller.log)` | Read the log, then **File > Restart controller**. |
| `Controller: running (started by something else; left alone)` | Another copy of the controller is running. Close it, then **File > Restart controller**. |
