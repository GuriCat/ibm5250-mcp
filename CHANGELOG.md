# Changelog

User-facing changes in the IBM 5250 MCP Server distribution, most recent first.

> 日本語版は [`CHANGELOG_ja.md`](CHANGELOG_ja.md) を参照してください。 (Japanese version: see [`CHANGELOG_ja.md`](CHANGELOG_ja.md).)

Time-limited beta release. **Windows (x64) only. Valid until 2026-07-31.** The source code is not
published. Verified in a Japanese environment only (primary language 2962, QCCSID 65535).

---

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
