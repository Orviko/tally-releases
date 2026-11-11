# Tally

Tally tracks AI usage and plan limits for Claude Code, Codex and more, from the menu bar or system tray. This repository holds its installers for macOS, Windows and Linux. The source code is private.

**[Download the latest release](https://github.com/Orviko/tally-releases/releases/latest)** · [All releases](https://github.com/Orviko/tally-releases/releases)

## Install

- **macOS 11 or later:** `.dmg`, for Apple silicon and Intel. Signed and notarized, so it opens without a warning.
- **Windows 10 or later:** `-setup.exe`, 64-bit. Not signed, so Windows warns before opening it: choose **More info**, then **Run anyway**.
- **Linux:** `.deb`, 64-bit, for Debian 12, Ubuntu 22.04 or later. Install it with `sudo apt install ./Tally_*.deb`.

Tally doesn't update itself. To update, install the newest release over the old one. Settings and the model policy carry over.

## Release notes

### 1.1.0

Install it over 1.0.0. Builds that send usage to Orviko Tasks ask each person to sign in once.

- **Sign in with Orviko Tasks.** Tally asks to sign in the first time it runs, and shows nothing else until then. It opens Orviko Tasks in the browser with a short code, the person approves the PC, and its usage counts as theirs from then on. Nobody can send usage as someone else. **Settings** shows who the PC is signed in as, with **Sign out**, and the admin can sign a PC out from Orviko Tasks.
- **Who used each account's week**, on the new Accounts page in Orviko Tasks: each account's weekly limit split between team members and their PCs, in the account's own window, with past weeks kept for good.
- **Works with claude-swap.** With `cswap` installed, Tally reads every account it manages through it and never asks Claude for those itself, so the two don't use up an account's allowance between them. Switch account from the popover, where the account Claude Code is on shows as **Active**, or from **Switch Claude Code To** in the tray menu, both with `cswap switch`. Tally asks first, and won't switch to an account whose limit has nothing left.
- **Gentler on accounts.** Tally asks Claude for limits at most once every 3 minutes, and never renews a login itself.
- **Shows what's left.** The menu bar says how much of the limit is left, such as `18% left · 11.8h`, and its bar drains like a battery: green, yellow under 40%, orange-red under 20%, and red when it's gone. The dashboard's meters and limit bars work the same way.
- **A refreshed dashboard and popover.** The cards follow the range picked: total, average per hour or day with the busiest one, and the top model. With all accounts, the popover lists each account's limits and when they reset. Choosing one shows its tightest limit, then its usage. The account menu, switches and buttons are new too.
- **Live usage.** The popover, dashboard and menu bar show new usage within about 2 seconds of Claude Code, Cowork or Codex writing it, instead of every 20 seconds.
- **Sync now always reads limits.** A Sync now from Orviko Tasks, or new usage, that comes within 3 minutes of the last reading now waits for those 3 minutes to end, rather than being skipped until the next 5-minute reading.
- **Says why the model policy can't be fetched,** such as a broken key, instead of only that Orviko Tasks couldn't be reached. A copied key's stray spaces or line breaks no longer break it.
- **Finds the Claude desktop app however it was installed.** Tally now reads the Microsoft Store version's profile, in `%LOCALAPPDATA%\Claude-Data`, and Flatpak and Snap installs on Linux, so their account, its limits and its Code tab and Cowork usage show up. It also reads Claude Code configs in `$XDG_CONFIG_HOME/claude` and `~/.claude-*` folders used with `CLAUDE_CONFIG_DIR`, including one that only holds a login, so signing a desktop-only account in to Claude Code once in its own folder shows its email, plan and live limits. A test no longer writes into the real `%APPDATA%\Claude` on Windows.
- **The Mac's model policy helper keeps a valid signature.** It used to be a copy of Tally's executable on its own, whose signature only holds inside the app, so macOS logged it as invalid and ran it only because it doesn't yet enforce that check on background services. Tally now installs a copy of the whole app, checks its signature before replacing the helper, and removes the old copy. Each Mac asks for an administrator's password once to switch over.
- **The model policy can't be faked on Windows.** Anyone could add files to the folder where the policy task notes each run, `C:\ProgramData\Tally`, so a person without administrator rights could make a PC look set up. The task now locks that folder to administrators and replaces one someone else made, and Tally only trusts a note the task wrote itself.
- **Max and Ultra.** Claude's Max 5x and Max 20x plans show their new names, Max and Ultra.

## Commands Tally runs

Tally does not run `sudo` or other administrator-level commands without your permission, and *Orviko Tasks* cannot trigger arbitrary commands. All commands *Tally* executes are explicitly defined in its source code with fixed arguments. These restrictions are enforced only through application code, not by the operating system.

I reviewed the source code to verify the current implementation and made no changes.

### The admin-level job: model policy only

- **Mac:** Tally asks for an admin password once, in macOS's normal prompt, to install its policy helper. It doesn't use `sudo`.
- **Windows and Linux:** the installer sets the job up, as a scheduled task on Windows and a systemd timer on Linux.
- **What the job does:** every minute it fetches the blocked-model list from Orviko Tasks and writes only `deniedModels` and `availableModels` to Claude Code's policy file.
- **Why the server can't misuse it:**
  - Model names are filtered to letters, numbers and `-._:@/[]`, so Orviko Tasks can't slip in hooks or commands.
  - On a Mac, the helper is a root-owned copy of the app, so someone who can change Tally.app can't use it to gain root.

### Commands Tally runs as the normal user

| Command | Why |
|---|---|
| `claude auth status` | Renews the Claude login so limits can be read |
| `security find-generic-password` (Mac) | Reads the Claude Code and Cursor logins from Keychain; macOS asks the person first |
| `cswap list` / `cswap switch` | Only if claude-swap is installed; switching only after the person confirms |
| `scutil` / `hostname` | The PC's name |
| `open` / `rundll32` / `xdg-open` | Opens the sign-in page in the browser |
| `reg`, `schtasks`, `icacls` (Windows) | Start at login, check the policy task, lock down the policy folder |
| Install/uninstall scripts | Create or remove the policy task or timer (Windows uses PowerShell for the battery setting) |
