[README.md](https://github.com/user-attachments/files/35235303/README.md)
# GameSpoof Studio

A small, local desktop app that makes the official Discord client think a game is running. It starts an idle placeholder process at the exact file path Discord's detectable-games catalog expects, and nothing else.

**It never asks for your Discord login, password, or token, and it never talks to your Discord account.** Discord detects the placeholder process on its own, the same way it detects a real game.

Built with Tauri v2 (Rust) + React + TypeScript + Tailwind CSS. Runs on Windows and macOS.

> ⚠️ **Read the [disclaimer](#disclaimer) before using this.**

---

## Features

- **Huge game library:** pulls Discord's official detectable-games catalog (10,000+ titles), cached locally for instant offline starts, with a manual **Sync Catalog** button.
- **Game artwork:** shows each game's logo, with fallbacks so most games get an image.
- **Home screen with four shelves:**
  - **Pinned:** your favorites, one click to launch
  - **Recently Opened:** the last games you started
  - **New Releases:** the newest additions to Discord's catalog
  - **Least Popular:** lesser-known titles *(an estimate, see [Limitations](#limitations))*
- **Fast search** by game title or executable name, across the whole catalog, with a virtualized list that stays smooth at 10,000+ entries.
- **Multiple games at once**, each with a live `HH:MM:SS` session timer in the Running Games panel.
- **Custom games:** add any executable name or relative path yourself.
- **All Platforms toggle:** see executables Discord lists for other operating systems (useful on macOS, where many games only list Windows `.exe` names).
- **Clean teardown:** stopping a game, or closing the app, kills every placeholder process and deletes its temporary files.

---

## How it works

Discord detects games by looking at running processes and matching their executable name and folder path against its catalog. GameSpoof Studio creates a harmless idle process at that path:

| OS | What the placeholder is |
|----|-------------------------|
| **Windows** | A copy of this app's own executable, saved at the game's exact relative path (for example `Binaries/Win64/Game.exe`) and started in idle mode. It opens a tiny window and uses ~0% CPU. |
| **macOS** | A copy of `/bin/sleep` saved at the exact path (for example `Game.app/Contents/MacOS/Game`) and run with `sleep 86400`. If macOS refuses it, the app falls back to its own idle binary. |

Placeholders live under the app's local data folder in `active/<application id>/…`. The app checks them every 2 seconds and removes them when they stop.

---

## Download

Grab the latest build from the **[Releases](../../releases)** page:

- **Windows:** `.exe` installer or `.msi`
- **macOS:** `.dmg` (universal: Apple Silicon and Intel)

### First launch warnings

The builds are **not code-signed**, so your OS may warn you:

- **Windows SmartScreen:** click **More info → Run anyway**.
- **macOS Gatekeeper:** right-click the app and choose **Open**, or run
  `xattr -cr "/Applications/GameSpoof Studio.app"`.
- **Antivirus:** because the Windows placeholder is a copy of the app that sits idle at a "game" path, some antivirus tools may flag it. If that happens, you can add an exclusion for the app's data folder (see [Where data is stored](#where-data-is-stored)). You can also read the source and build it yourself.

---

## Usage

1. Open the app and let the catalog sync (first run needs an internet connection).
2. Find a game on **Home** or search for it, then click **Start**. The card turns green and shows **LIVE**.
3. Open Discord (the **desktop app**, not the browser). Within a short time it should show you as playing that game.
4. Click **Stop** (or **Stop All** in the Running Games panel) when you're done.

### If Discord doesn't pick it up

- In Discord, go to **Settings → Activity Privacy** and make sure activity sharing is turned on.
- Go to **Settings → Registered Games**. If the placeholder appears in the **Add it!** list, add it once.
- Run Discord and this app at the same privilege level (don't run only one as administrator).
- Try quitting Discord fully (from the system tray) and reopening it with the placeholder already running.
- Detection depends on Discord's catalog data, so it isn't guaranteed for every title.

---

## Build from source

**Requirements:** [Node.js](https://nodejs.org) 20+, [Rust](https://rustup.rs) (stable), and the [Tauri v2 prerequisites](https://v2.tauri.app/start/prerequisites/) for your OS.

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
npm install
npx tauri icon app-icon.png     # generates src-tauri/icons (once)
npm run tauri dev               # run in development
npm run tauri build             # produce installers in src-tauri/target/release/bundle
```

### Automated builds (GitHub Actions)

`.github/workflows/build.yml` builds Windows and macOS (universal) packages.

- Run it manually from the **Actions** tab (**Run workflow**), or
- push a tag like `v1.0.0` to also publish the files as a GitHub Release.

For releases, set **Settings → Actions → General → Workflow permissions** to **Read and write**.

---

## Where data is stored

| What | Windows | macOS |
|------|---------|-------|
| Placeholder files, pins, recents, custom games | `%LOCALAPPDATA%\com.gamespoof.studio\` | `~/Library/Application Support/com.gamespoof.studio/` |
| Catalog and icon caches | `%APPDATA%\com.gamespoof.studio\` | `~/Library/Application Support/com.gamespoof.studio/` |

Uninstalling the app does not necessarily delete these folders; you can remove them by hand.

## Privacy

The app has no account system, no analytics, and no telemetry. It makes these network requests:

- `discord.com`: downloads the public detectable-games catalog, and looks up public icon info for games that lack one. No login or token is sent.
- `cdn.discordapp.com`: game icons.
- `cdn.cloudflare.steamstatic.com`: game artwork fallback for games linked to Steam.

Pins, recents, and custom games are stored only on your computer.

---

## Limitations

- **"New Releases" is approximate.** The date is derived from each game's Discord application ID, which records when the entry was created. That shows the newest additions to Discord's list, which can be later than the game's real release date.
- **"Least Popular" is an estimate.** Discord's catalog has no player counts. The app guesses from how few executables a game has registered, since big titles tend to register many.
- **macOS placeholder lifetime:** the `sleep` placeholder exits after 24 hours; the app then removes it from the running list.
- **Crashes:** if the app is force-killed, placeholders may keep running until you end them in Task Manager / Activity Monitor. A normal close always cleans up.
- **Detection isn't guaranteed.** It depends on Discord's catalog and on how Discord's client matches processes.

---

## Project structure

```
├── src/                    React + TypeScript + Tailwind UI
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── src-tauri/
│   ├── src/main.rs         idle-stub entry point
│   ├── src/lib.rs          catalog, process spawner, monitor, cleanup
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── capabilities/
├── app-icon.png            source icon (expanded by `tauri icon`)
└── .github/workflows/      CI build
```

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Disclaimer

GameSpoof Studio is an independent project. It is **not affiliated with, endorsed by, or sponsored by Discord Inc.** "Discord" and the names of all games are trademarks of their respective owners.

Making Discord display a game you are not actually playing may go against Discord's Terms of Service or Community Guidelines, and could be considered misleading to other users. **You use this software at your own risk and are responsible for how you use it.** The authors accept no liability for account actions, lost data, or any other consequences. Please do not use it to deceive others in ways that could harm them, or to circumvent rules of any game, event, or service.

## License

[MIT](LICENSE)
