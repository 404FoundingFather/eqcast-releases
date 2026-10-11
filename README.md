# eqcast

Play one EverQuest character and have your keys, and when you want it your mouse, repeated in up to five other clients at the same time. The other windows stay in the background, your keyboard is never locked out, and one window starts, places and logs in the whole team.

eqcast is a free, Windows-only alternative to ISBoxer for people who only multibox EverQuest. This repository holds the downloads and the guides. There's no source code here.

## Get it

Download the newest version from the [Releases](../../releases/latest) page. Each release comes as:

- **`eqcast-setup-<version>.exe`**, the installer. No administrator rights needed; it installs for your Windows user and keeps your settings when you upgrade.
- **`eqcast-<version>-win-x64.zip`**, the portable version. Unzip it anywhere and run `eqcast-launcher.exe`.

You need Windows 10 or 11, 64-bit. Nothing else to install; the .NET runtime is included. Every release lists what changed and a SHA-256 for each file.

**Windows will probably warn you.** eqcast isn't code-signed yet, so SmartScreen may say "Windows protected your PC". Choose **More info**, then **Run anyway**.

## Read these

- **[User's guide](USER-GUIDE.md)** — install, your first launch in four steps, every tab explained, and what to do when something goes wrong. Start here.
- **[How it works](HOW-IT-WORKS.md)** — a tour of the three programs that make up eqcast and how a keystroke travels from your keyboard to the other clients. For the curious, and for anyone who wants to know exactly what eqcast does to the game before running it.

## Good to know

- **It stays out of ISBoxer's way.** If you have ISBoxer installed, eqcast ignores the clients ISBoxer manages. It can also import the keys from one of your ISBoxer key maps to get you started.
- **It never picks your character or enters the world.** It gets each client to the login screen (or character select, with Quick Connect) and stops there.
- **Passwords, if you save them, are stored as plain text** in your user folder. Saving them is optional; everything else works without.
- **Your settings live in `%USERPROFILE%\.eqcast`.** Upgrading keeps them; uninstalling asks before removing them.

## The fine print

eqcast is free software under the [MIT license](https://opensource.org/licenses/MIT). It isn't made or endorsed by Daybreak. Sending input to background game clients is what multiboxing tools have always done, but whether that's acceptable for your account is between you and Daybreak's rules. Use it at your own risk.
