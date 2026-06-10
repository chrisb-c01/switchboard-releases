# Changelog

All notable changes to Switchboard are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
The version source of truth is `src-tauri/tauri.conf.json`.

## [Unreleased]

## [1.0.0-beta.10] - 2026-06-10

### Changed

- `Cmd+W` closes the focused session instead of the whole window. The
  shortcut is remappable in Preferences like the others.
- Closing a session no longer asks for confirmation — the shell and
  anything running in it terminate immediately.

## [1.0.0-beta.9] - 2026-06-10

### Changed

- The sidebar's "New group" and "New folder" buttons are replaced by a
  single "Add" button that opens a small menu to choose between the two.
- Folder collapse/expand is now a clear right/down chevron with a larger
  click target instead of the tiny disclosure glyph.
- "+ Session" is renamed to "Add Session" — button labels no longer use
  a `+` prefix.

### Fixed

- Clicking a session while another app has focus now focuses that session
  in one click: macOS swallowed the click that activated the window, so
  the pane underneath never saw it (the window now accepts the first
  mouse event, like iTerm).
- Dragging files from Finder onto a session works — the feature shipped
  in beta.8 but its drop events never reached the app (they target the
  window, not the webview, and carry logical coordinates mislabeled as
  physical).

## [1.0.0-beta.8] - 2026-06-10

### Added

- Sidebar folders: organize groups into collapsible folders, mixed freely
  with loose groups. Collapsing is purely visual (sessions keep running; the
  collapsed header shows the worst status across its groups); deleting a
  folder just moves its groups back out in place.
- Drag to reorder groups and folders in the sidebar. Order is fully manual
  now — the old alphabetical auto-sort is gone, and renaming a group keeps
  its position. `Cmd+Option+1..9` and `Cmd+.`/`,` follow the visible order.
- Resizable sidebar: drag its right edge (double-click the edge to reset);
  the width persists across launches.
- `Cmd+F` finds text in the focused terminal: find-as-you-type with all
  matches highlighted, Enter/Shift+Enter to cycle, Esc to return to the
  terminal. Remappable in Preferences like the other shortcuts.
- Drag files from Finder onto a pane to type their shell-quoted paths into
  that session at the cursor (no auto-submit); the pane under the cursor
  highlights during the drag and takes focus on drop.

### Fixed

- Clicking a pane while Switchboard was in the background reactivated the
  window but left keystrokes going to the previously-focused terminal; the
  clicked pane now takes focus once the window is active.

## [1.0.0-beta.7] - 2026-06-07

### Added

- Cmd+click a URL in a terminal to open it in the browser (iTerm-style);
  plain clicks still select/focus.

## [1.0.0-beta.6] - 2026-06-07

### Fixed

- Typing `exit` (or any shell exit) left a dead pane behind a broken "shell
  exited" overlay. The session now closes and drops its pane, like a
  terminal tab; close/restart/quit are still distinguished from a real
  exit, so a restart never auto-closes the session it just respawned.

## [1.0.0-beta.5] - 2026-06-07

### Fixed

- Window dragging from the seamless title bar (sidebar strip + group
  header) was denied by the capability ACL, so the window couldn't be
  moved; dragging and double-click-to-zoom work again.

## [1.0.0-beta.4] - 2026-06-07

### Fixed

- Pane titles now follow `cd`: the periodic cwd refresh never told the
  frontend about changes, so the folder name in a pane title stayed stale.
  It now updates within 5 seconds of a directory change.

## [1.0.0-beta.3] - 2026-06-07

### Added

- Releases also ship a versionless `Switchboard.dmg`, making
  `releases/latest/download/Switchboard.dmg` a permanent install link.

## [1.0.0-beta.2] - 2026-06-07

No user-facing changes (release-process documentation only).

## [1.0.0-beta.1] - 2026-06-07

### Added

- Release channels: opt into beta versions via Preferences → Updates. The
  beta channel also receives every production release; production installs
  never see betas. Beta builds report crashes under the `beta` environment.
- Updates section in Preferences: current version, beta-channel toggle, and
  a manual "Check for updates" with an explicit up-to-date / failed answer.
- The updater now re-checks every 4 hours while the app runs, instead of
  only once at startup.
- Crash reporting via Sentry for all builds: Rust panics and forwarded
  webview errors, scrubbed to error type/message/stack/release/OS — no
  breadcrumbs, no PII, no hostnames; terminal content never leaves the app.
  Builds are separated by environment (production / dev-for-testers /
  development) and every event carries `switchboard@<version>+<git sha>`.
- Seamless window: hidden title bar with overlay traffic lights; drag via
  the sidebar top strip or the group header.
- Shift+Enter inserts a newline in Claude prompts instead of submitting.

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

- Long group names truncate with an ellipsis instead of overflowing the
  sidebar; the full name shows on hover.
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

[1.0.0-beta.10]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.10
[1.0.0-beta.9]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.9
[1.0.0-beta.8]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.8
[1.0.0-beta.7]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.7
[1.0.0-beta.6]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.6
[1.0.0-beta.5]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.5
[1.0.0-beta.4]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.4
[1.0.0-beta.3]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.3
[1.0.0-beta.2]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.2
[1.0.0-beta.1]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v1.0.0-beta.1
[0.3.0]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v0.3.0
[0.2.0]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v0.2.0
[0.1.0]: https://github.com/chrisb-c01/switchboard-releases/releases/tag/v0.1.0
