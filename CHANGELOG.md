# Changelog

All notable changes to Switchboard are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
The version source of truth is `src-tauri/tauri.conf.json`.

## [Unreleased]

### Added

- Release channels: opt into beta versions via Preferences → Updates. The
  beta channel also receives every production release; production installs
  never see betas. Beta builds report crashes under the `beta` environment.
- Updates section in Preferences: current version, beta-channel toggle, and
  a manual "Check for updates" with an explicit up-to-date / failed answer.
- The updater now re-checks every 4 hours while the app runs, instead of
  only once at startup.
- Releases also ship a versionless `Switchboard.dmg`, making
  `releases/latest/download/Switchboard.dmg` a permanent install link.
- Crash reporting via Sentry for all builds: Rust panics and forwarded
  webview errors, scrubbed to error type/message/stack/release/OS — no
  breadcrumbs, no PII, no hostnames; terminal content never leaves the app.
  Builds are separated by environment (production / dev-for-testers /
  development) and every event carries `switchboard@<version>+<git sha>`.
- Seamless window: hidden title bar with overlay traffic lights; drag via
  the sidebar top strip or the group header.
- Shift+Enter inserts a newline in Claude prompts instead of submitting.
- Groups sort alphabetically.

### Changed

- Rebranded to C01: bundle identifier is `nl.c01.switchboard` (state
  migrates from the legacy directory automatically; stale hook entries are
  pruned on the next hooks Enable).
- Default shortcuts: `Cmd+]`/`[` cycle sessions, `Cmd+.`/`,` switch groups
  (the native Settings… accelerator was dropped to free `Cmd+,`).
- Sidebar: redundant attention-count pill removed; collapse toggle fused
  with the right border; pane-header status dots no longer pulse (the
  sidebar keeps the animation); scrollbar slimmed to a 6px macOS-like
  overlay.
- Expanding a pane now gives it all remaining space; squeezed neighbors
  render header-only (their PTY keeps its size).

### Fixed

- Resume: claude only persists a conversation after its first prompt —
  never-prompted sessions now restart a fresh claude instead of erroring
  "No conversation found"; SessionStart records the session id at launch;
  Ctrl+C clears a stale "working" status (no Stop hook fires on abort).
- Status truthfulness: permission/question prompts show orange even for
  the focused pane (only the idle "waiting for input" nag is suppressed
  there); answering a live claude's prompt flips to working immediately;
  Pre/PostToolUse keep "working" alive through a whole turn.
- Group switches hand keyboard focus to the first pane (typing works
  without clicking), and the giant-glyph flash on zoom/expand is gone
  (canvases hide for the frames until the refit lands).
- "Working" status precision: Esc aborts a turn without a Stop hook and now
  clears the status like Ctrl+C; focusing a needs-input session no longer
  flips it to "working" (claude is still waiting); and a watchdog demotes a
  "working" session whose terminal has been silent for 15s — a working
  claude repaints its spinner continuously, so silence means the Stop event
  was lost (claude killed, crash, dropped hook message).

## [0.3.0] - 2026-06-06

### Added

- Native macOS app menu with **Settings…** (`Cmd+,`) opening a sectioned
  Preferences surface (Shortcuts first; more sections to come).
- Pane titles show the session's jump shortcut as a `⌘N` chip (chip only for
  default-named sessions, chip + name after a rename); the path in the title
  bar is reduced to the current folder's name.

### Changed

- Sidebar footer decluttered: version centered on its own bottom line, the
  collapse toggle is now a `<` / `>` handle vertically centered on the
  sidebar's right edge, and the Shortcuts button moved into Preferences.

### Fixed

- Stray reverse-video `%` at the top of restored sessions: zsh's partial-line
  marker self-erase pads to the spawn-time terminal width, which never matched
  the real pane. Shells now spawn lazily on first attach at the pane's actual
  size, so the marker erases cleanly. Side effect: sessions in open but
  inactive groups start their shell on first visit.
- Claude-resume hardening: hook events are ignored while the app is shutting
  down, so the `SessionEnd` fired by claude dying with its PTY can no longer
  clear the resume flag at quit. (The resume pipeline itself was verified
  end-to-end.)

### Removed

- Worktree-based session creation — moved back to the backlog pending a
  better UX.

## [0.2.0] - 2026-06-06

### Added

- Auto-updater: the app checks the public releases repo on startup and shows
  an in-app popup with "Restart & Update" and a link to what's new.
- Grid expand: hover a pane header in a 2- or 4-session grid for ⇔/⇕ buttons
  that temporarily widen/heighten the pane (7:3), shrinking its neighbors —
  lighter-weight than full zoom.
- Collapsible sidebar: ☰ (or `Cmd+\`) collapses the 180px sidebar to a 56px
  rail of Slack-style group tiles (initials + one aggregated worst-wins status
  dot, tooltip with the per-status breakdown).
- App version shown at the bottom of the sidebar.
- Frontend (vitest) and Rust (cargo test) unit test suites, run in CI on every
  push and pull request.

### Changed

- New minimal default shortcuts: `Cmd+N` new session, `Cmd+Shift+N` worktree
  session, `Cmd+J`/`Cmd+Shift+J` cycle sessions (frees `Option+Tab` for the
  shell), `Cmd+]`/`Cmd+[` switch groups. `Cmd+K`, `Cmd+Enter` and the digit
  shortcuts are unchanged; everything remains remappable.
- Redundant "Switchboard" sidebar title removed.
- Terminal scrollbar restyled at the xterm 6 overlay-slider layer (the old
  webkit CSS never applied) and scrolling smoothed (`smoothScrollDuration`).

### Fixed

- False orange "needs input" flash when Claude finishes starting: attention
  signals for the session you are actively viewing are muted, and bell noise
  within 5s of shell spawn is ignored.

## [0.1.0] - 2026-06-06

Initial release.

### Added

- Groups of Claude Code terminal sessions: a Tauri 2 macOS app with a Rust
  backend (portable-pty sessions, persistence, Claude Code hook listener,
  attention state machine, macOS notifications) and a React/xterm.js frontend
  (group sidebar, auto-tiling terminal grid, WebGL-accelerated rendering).
- Visible rename/close/delete buttons on group rows, backed by native dialogs.
- Notification click focuses the app and jumps to the session that triggered it.
- Keyboard shortcuts: Cmd+1-9 sessions, Cmd+Opt+1-9 groups, Opt+Tab to cycle
  sessions, Cmd+Shift+[/] to cycle groups. Shortcuts are configurable from the
  sidebar and persisted in `state.json`.
- Resume Claude conversations on app restart: the hook script forwards the
  Claude `session_id` (persisted per pane), and restored sessions with an
  active conversation run `claude --resume` automatically. A `SessionEnd` hook
  clears the resume flag when Claude exits.
- Worktree sessions: "+ Worktree" / Cmd+Shift+T creates `<repo>-wtN` next to
  the group's first-session repo and opens a session there.
- Cmd+K quick switcher across all open groups, surfacing needs-input sessions
  first.
- Pane headers showing the session name, a status dot, and a status label,
  with a right-click context menu.
- Switchboard app icon (dock, notifications).
- Slim dark terminal scrollbars.

### Fixed

- Hook socket now binds inside the Tokio runtime instead of panicking, and
  attention signals for unknown session ids are ignored.
- `window.confirm` (a silent no-op in the Tauri webview that broke group
  close/delete) replaced with native dialogs.
- Working-directory tracking on macOS: replaced the Linux-only `pidcwd` path
  with a direct `proc_pidinfo(PROC_PIDVNODEPATHINFO)` FFI call, so session
  cwds refresh correctly and restore no longer reverts to the spawn default.

[0.3.0]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v0.3.0
[0.2.0]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v0.2.0
[0.1.0]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v0.1.0
