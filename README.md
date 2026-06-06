# Switchboard Releases

This repository hosts the public release artifacts and changelog for
**Switchboard**, a macOS app for managing groups of Claude Code terminal
sessions. The application source lives in a separate private repository.

## What's here

- **Releases** — signed macOS bundles (`.app`, `.dmg`) and the auto-updater
  artifacts (`.app.tar.gz` + `.sig`) for each version, published under
  [Releases](https://github.com/chrisb-c01/switchboard-releases/releases).
- **`latest.json`** — the [Tauri v2 updater](https://v2.tauri.app/plugin/updater/)
  manifest. The app checks
  `https://github.com/chrisb-c01/switchboard-releases/releases/latest/download/latest.json`
  to discover and download updates.
- **`CHANGELOG.md`** — the [Keep a Changelog](https://keepachangelog.com/)
  history, mirrored here from the app repo so it can be synced to GitBook.

## Installing

Download the latest `.dmg` from the
[Releases page](https://github.com/chrisb-c01/switchboard-releases/releases/latest)
and drag Switchboard to your Applications folder. The app updates itself from
then on.
