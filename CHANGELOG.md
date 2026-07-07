# Changelog

User-facing changes in the IBM 5250 MCP Server distribution, most recent first.

> 日本語版は [`CHANGELOG_ja.md`](CHANGELOG_ja.md) を参照してください。 (Japanese version: see [`CHANGELOG_ja.md`](CHANGELOG_ja.md).)

Time-limited beta release. **Windows (x64) only. Valid until 2026-07-31.** The source code is not
published. Verified in a Japanese environment only (primary language 2962, QCCSID 65535).

---

## 2026-07-07 (v0.1.2-beta)

### Fixed
- **A background grid-line box no longer lingers on a full-screen Help panel** — On DBCS screens that draw a ruled box with grid lines (for example the AICHAT assistant screen), opening Help used to leave the background grid-line rectangle drawn on top of the Help panel. The grid lines are now cleared while Help is shown and restored when you exit Help, matching ACS.

---

## 2026-07-06 (v0.1.1-beta)

### Fixed
- **A leftover option code no longer appears beside a popup window's left border** — When you type an option code (for example `5`) on a list and open a detail popup window, the typed character and its input underline used to remain just to the left of the window frame. They are now hidden, matching ACS.

---

## 2026-07-04 (v0.1.0-beta)

### Changed
- **Version numbering unified to `0.1.0-beta`** — The exe's Windows version info, the in-app Help (ℹ️), and the GitHub tag now show the same version. Previously the exe reported the bundled runtime (Bun) version, which did not match the actual product version. The version is bumped each release from now on
- **GitHub tag is now version-based** — The misleading date-style tag (formerly `v2026-07-31`) is replaced by a version tag such as `v0.1.0-beta`. The expiry date is shown in the release title/body instead

### Distribution
- **The distribution zip is now a self-contained package** — In addition to the exe it bundles the README, CHANGELOG, EULA, third-party notices, and the full manuals (docs/manuals: guide HTML and images). After extraction the in-app Help (ℹ️) works offline

### Fixed
- **Line editing in mixed Japanese (DBCS) text now fully preserves cell positions (ACS-compliant)** — In overwrite mode, characters after the edited column must never move; this fundamental 5250 behavior was broken in DBCS (Japanese) mixed lines
  - Deleting the only DBCS character in a run no longer removes the shift characters (an empty SO/SI pair now remains, matching ACS)
  - Overwriting a shift-out (SO) cell no longer destroys following characters or shifts the rest of the line; only the affected cells change
  - The shift-in (SI) cell is now treated as protected: pressing Delete on it shows the same "Cursor is in a protected area of the display." error as ACS
  - The cursor no longer jumps to unrelated positions after delete/overwrite operations
- **Japanese input in multi-row entry fields is no longer split by double blanks when sent to IBM i** — Text spanning rows could arrive fragmented (e.g. "棒グラ␣␣フ"); it is now sent as one continuous string, matching ACS
- **Underlines (e.g. the command-entry line) no longer disappear when the input-field highlight is turned OFF**
- **Fixed adjacent characters being deleted when typing Japanese in insert mode** — Confirming an IME conversion in insert mode could overwrite/erase the character at the cursor and jump the cursor to an unintended position
- **Fixed a silent hang on dropped connections** — Enabled TCP keepalive so a connection whose network path has died is detected and shown as disconnected (COMM659). This prevents the silent hang where even System Request stopped working on an unresponsive connection

### Documentation
- **New "Updating" section in the README** — Documents that the exe must be downloaded from the release Assets (the "Source code" zip does NOT contain the exe), that zips must be extracted before use, that IDEs must be closed before replacing the exe under the same name, and that `config.json` must be kept as-is

### Distribution
- **Releases now ship a single zip package** — Only `ibm5250-mcp-windows-x64.zip` (exe + `config.example.json` + README) is distributed; the raw .exe (which browsers/corporate environments often block) is no longer offered. Extract before use. On update the bundle ships `config.example.json`, so it never overwrites your existing `config.json`

## 2026-07-03

### Added
- **Online help (ℹ️)** — a help button on the second toolbar row. It shows the version and build
  date, the End-User License Agreement (EULA), the third-party notices, and the User Guide and
  Admin Guide (HTML with images; contents and full-text search in the left pane) in a normal
  window of the default browser
- **`open_viewer` tool (MCP)** — opens the unconnected connection form, or reopens the window of a
  session that was closed by mistake

### Changed
- **Windows now open automatically only when connecting** — when started by an MCP client (such as
  Claude), no window opens at startup any more (previously the connection form always opened and
  doubled up with the per-connection window). Only when the product is launched directly does the
  connection form open at startup

### Fixed
- **Removed a spurious underline on window borders** — the left border of an on-screen window could
  show an underline carried over from an input field behind the window

### Documentation
- **Screenshots retaken with the current toolbar** — the User Guide and Admin Guide screen examples
  now show the current UI (including the window-mode toggle and the help button)

## 2026-07-01

### Added
- **A separate viewer window / port per session** — each connection gets its own port and browser
  window. This is equivalent to ACS "Run with the same configuration" and lets you open several
  connections with the same configuration side by side
- **Auto-open the viewer on connect** — controlled by `viewer.auto_open` in `config.json`
  (`off` / normal browser window / app window; the default is an app window)
- **Window-mode toggle button** (toolbar) — switch between an app window and a normal browser window.
  It reopens the current window in the other mode and also changes the startup default

### Changed
- **Disconnect and window close are now linked in both directions** — disconnecting from the MCP
  closes that viewer window, and closing a viewer window auto-disconnects the session after a few seconds
- **`session_id` is generated automatically** on connect (explicit values are no longer accepted)

## 2026-06-28

### Changed
- **The English-UI default for new connections is no longer Japanese** — when the UI is English, a new
  connection configuration defaults to host code page cp037 and font Consolas (the Japanese UI keeps
  cp1399 and ＭＳ Gothic). Switching the UI language also switches this default
- **Viewer port setting consolidated** into `viewer.port_default` (the per-preset viewer_port was removed)

### Added
- **English manuals now show English 5250 screens** — sign-on, main menu, WRKACTJOB, etc. screen
  examples were replaced with English captures

## 2026-06-21

### Fixed
- **Input-field underline offset** — fixed a case where the input-field underline was drawn two columns
  too far right on certain screens (a DBCS run continuing to the end of a line)

### Docs
- **Admin guide** — added the procedure and caveats for using the viewer together with Playwright

## 2026-06-15

### Fixed
- **No more dropped keystrokes on fast input** — viewer input is now serialized, eliminating dropped or
  reordered characters when typing quickly

## 2026-06-14 — Baseline features

The main features of the distribution as of this date. Later changes are listed above, newest first.

### Terminal emulation
- TN5250E protocol with EBCDIC SBCS / DBCS support (cp1399, cp5026_current, cp5026_legacy, cp037, cp5035, etc.)
- 24×80 (DS3) and 27×132 (DS4) screen sizes; the device type is auto-selected from the screen size + code page
- Ideographic (DBCS) field types J / E / G / O, with SO/SI handling and IME (Japanese) input
- Field handling: numeric / signed-numeric fields, field exit (including negative sign), monocase,
  mandatory / auto-enter, and segmented CNTFLD chains with word wrap

### Web screen viewer (`http://localhost:5250`)
- Color attributes, cursor, and the OIA (operator information area) with the `X II` keyboard-lock and
  `COMM659` (disconnected) indicators
- Configurable display font / size / bold / italic, with per-language defaults
  (Japanese: ＭＳ Gothic, English: Consolas)
- Function-key toolbar (F1–F24), PRINT, SysReq, Reset
- Paste with undo (Ctrl+Z), rectangular paste overflow (flows into the rows below), scroll-bar clicks
- Voice input (the recognition language follows the UI language)
- Toolbar hide, and input-ownership (exclusive / shared) mode for multi-client viewing
- UI language toggle (Japanese / English)

### Operation
- Connection presets in `config.json`; AUTOTYP scripting for sign-on automation
- Headless password via an environment variable; session logging
- A single self-contained executable (runtime bundled, no Node.js install required) that reads
  `config.json`, presets, and logs from the folder where the executable resides

### Documentation
- English and Japanese **User Guides** and **Admin Guides** (PDF) under `docs/manuals/`
- `AI-GUIDE.md` for driving the server from an AI agent

### Main improvements in this build
- Manual corrections (showing / hiding shift characters, voice-input auto-off, the SysReq action for
  `X SYSTEM`, and five SBCS code tables)
- Voice-input recognition language now follows the UI language (Japanese / English)
- Toolbar icons replaced with images of the actual buttons; cursor display and OIA insert-mark
  descriptions corrected to match the implementation
