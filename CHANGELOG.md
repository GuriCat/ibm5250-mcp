# Changelog

User-facing changes in the IBM 5250 MCP Server distribution, most recent first.

> 日本語版は [`CHANGELOG_ja.md`](CHANGELOG_ja.md) を参照してください。 (Japanese version: see [`CHANGELOG_ja.md`](CHANGELOG_ja.md).)

Time-limited beta release. **Windows (x64) only. Valid until 2027-01-31.** The source code is not
published. Verified in a Japanese environment only (primary language 2962, QCCSID 65535).

---

## 2026-10-30 (v0.1.15-beta)

### Changed (includes incompatible changes)
- **Expiry extended to 2027-01-31** — two months beyond the previous 2026-11-30. From 2027-02-01, startup and connection are automatically refused.
- **AUTOTYP data lines are now typed at the cursor position, as on the keyboard (incompatible change)** — As in the original AUTOTYP specification, data is entered at the cursor position (overwrite; the rest of the field is not erased). When the last position of a field has been typed, typing continues in the next input-capable field, and typed characters appear on the screen immediately. Previously the data was buffered by field number "from the first field" and sent together with the AID key, so it could go into a different field from the one where the host placed the cursor. Write `^FLDEXIT` / `^ERASEEOF` to erase the rest of a field. After typing into a right-adjust field, end it with a `^FLDEXIT` key before sending an AID key, as on a real terminal.
- **AUTOTYP field keys now behave the same as MCP `send_key`** — `^FLDEXIT` keys (erase from the cursor and move to the next input-capable field; numeric fields are right-adjusted), `^DEL`, `^DUP`, and `^ERASEEOF` / `^ERASEINPUT` act on the screen (previously they only logged, or only advanced the field number). `^INSERT` toggles insert mode, `^RESET` is Error Reset, and `^ESC` is New Line.
- **AUTOTYP end codes now follow the numbering of the original AUTOTYP (incompatible change)** — e.g., jump target label not found = 19 (previously 20), terminal-side input check error = 24 (previously 26), overall timeout = 29 (previously 12). A first data other than `^ATP` = 14, a last data other than `^END` = 12, and an unknown keyword = 15 stop before execution (previously an unknown keyword was typed as text). `^DSP` format errors stop with 20–23. See the manual for the list.
- **AUTOTYP `^ERM` and `^CSR` are implemented with their original meaning and are on by default** — `^ERM`: stops with 25 when a host error message (such as an input error) is displayed (the previous `^ERMON` behavior of resending the AID is removed). `^CSR`: the next data is not processed while the host has not sent a screen yet. Write `^ERMOFF` / `^CSROFF` to turn them off.
- **The state right after connecting, before the host screen arrives, now conforms to Free Key mode as specified in the IBM manual "5494 Remote Control Unit Functions Reference" (SC30-3533-04), pages 15-1, 16-3 and A-11** — the whole screen is one input field with the cursor at row 1, column 2. Pressing a key that needs the host (such as Enter) during this time shows 0099 on row 1 and sends nothing to the host.
- **`send_key HOME` now conforms to the same manual, page 16-10 (Home key) and page 16-5 (record backspace)** — it moves the cursor to the home position (the cursor position the host specified for the screen, or else the first input-capable field). Only when the cursor is already at the home position does it send Record Backspace to the host (previously it always sent it; the viewer's Home key already behaved correctly). `NEWLINE` (New Line) was added to `send_key`.
- **Numeric fields (right-adjust, signed numeric, numeric only) no longer advance to the next input-capable field when the last position has been typed** — Conforming to the same manual, page 15-20 (numeric only and signed numeric fields are exited with Field Exit / Field+ / Field−) and to measurements on ACS, the cursor stays at the last position (keyboard input, `send_text`, and AUTOTYP). Use a Field Exit key to leave the field. In AUTOTYP, characters that do not fit end with code 24.
- **Automatic reconnection when the host closes the connection** — As described in the help of the ACS (IBM i Access Client Solutions) 5250 emulator (OIA), COMM659 is shown in the OIA when the connection to the host is lost. From 1 second later, reconnection is attempted and COMM657 (establishing the connection) is shown for each attempt. The session reconnects in the same window and session, returning to the sign-on screen. By default it tries every 5 seconds up to 10 times; change this with `reconnect` in the configuration file (config.json) (`enabled` / `attempts` / `interval_sec`).
- **AUTOTYP `^DSP` syntax restored to the original AUTOTYP specification (incompatible change)** — In `^DSPxxyy<text> [label`, the row and column are **both 2 digits, counted from 0** (e.g., row 18, column 2 of the screen = `^DSP1701`). For column 100 and beyond, write A–D as the tens digit of the column (`A1` = column 101). Scripts written in the previous "2-digit row + 3-digit column, from 1" form must be rewritten. The check is whether the text from the given position to the end of the row starts with the comparison text (SO/SI are ignored).
- **Input is no longer processed while input is inhibited (except SysReq, Attn, RESET and PRINT)** — As on a real 5250 terminal, after an AID key the next input is not processed until the host accepts input.
  - AUTOTYP: before data, key, and `^DSP` lines, it waits up to 30 seconds for the release. Previously it waited for only one screen update, so it could check `^DSP` or pick the input field while the host was still sending the screen, and a command right after sign-on could be dropped. As in the original AUTOTYP, `^INH` monitoring is **on by default** (use `^INHOFF` to stop it). `^INH [label` jumps to the label whenever input is inhibited, regardless of `^INHON` / `^INHOFF`.
  - MCP `send_text` / `paste_text` / `send_key` and viewer input also wait up to 30 seconds while input is inhibited. If it is not released, the input is discarded and `INPUT_INHIBITED` is returned. SysReq, Attn, RESET, and PRINT do not wait; pressing SysReq, Attn, or RESET discards the waiting input.

### Fixed
- **On rare occasions, the host screen did not appear after connecting** — When the random value the host sends at connection time contained a certain byte (about 3% of connections), the end of the telnet control data was misread and the first screen was never processed.
- **The keyboard could stay input-inhibited even after the host unlocked it** — the unlock indicated by the screen write was not reflected.
- **Field Exit right after DBCS characters in an open (O) field removed the SI** — Data with an SO but no SI was sent to the host and the write failed (CPF5035). The SI is now kept, and when pressed in the middle of DBCS data, an SI is placed at that position (reported by another project).
- **Signed numeric fields were not right-adjusted by Field Exit / Field+ / Field−** — They are now right-adjusted and padded with blanks on the left (the sign position is kept).
- **The Enter key was rejected with 0020 just because the cursor was in an untouched right-adjust field holding a host-written value** — Only fields the operator has typed into are checked.
- **Pressing Enter on the system request line could be stopped by input-field checks (such as mandatory entry)** — Enter on the system request line does not send the screen's input fields, so they are not checked.
- **In AUTOTYP, a data line after `^SYSRQ` went into a screen input field instead of the system request line** — `^SYSRQ` → `90` → `^ENTER` now signs off.
- **SysReq always took 5 seconds to respond** (AUTOTYP `^SYSRQ` and viewer SysReq).
- **The viewer recorder wrote `^WAITX` in 100 ms units** — Playback waited 5 times longer. It now writes 500 ms units.

### Added
- **AUTOTYP `^WAIT1`–`^WAIT9`** — The original AUTOTYP form (the digit × about 550 ms). `^WAITX nn` (500 ms units) still works.

---

## 2026-09-15 (v0.1.14-beta)

### Changed
- **Expiry extended to 2026-11-30** — two months beyond the previous 2026-09-30. From 2026-12-01, startup and connection are automatically refused. No functional changes (identical features to v0.1.13-beta).

---

## 2026-08-11 (v0.1.13-beta)

### Added
- **Terminal-side input checks (`CHECK(MF)` / `CHECK(ME)` / right-adjust Enter gate) are now enforced** — the input checks that a real 5250 terminal performs (mandatory fill / mandatory enter / pending right-adjust) were not executed, so input that a real terminal would reject was sent to the host as-is (reported by another project). The server now validates before sending Enter / F1-F24; on a violation it does not send the AID, returns an error (operator error code 0020 / 0014 / 0007), and shows a white message on the bottom line of the screen using the same wording as the real emulator (verified against ACS on real hardware; auto-restored by the next valid key). When a right-adjust field (`CHECK(RZ)` / `CHECK(RB)` / signed numeric) is partially typed, exit it with a Field Exit key (which right-adjusts the value) before sending — the error message says so. CLEAR / HELP / PRINT / Roll / PA keys are exempt, matching real terminals. Applied to every input path: `send_text` / `send_key` / `paste_text` / viewer keyboard / AUTOTYP / macros. Note: `CHECK(FE)` (field-exit required) does not block the Enter key even on a real terminal (verified on ACS — it only rejects further typing while the cursor is held on the last position), so it is not part of the AID-time checks; rejecting extra typing was already implemented.
- **Viewer notification of check violations** — violations are also pushed to the viewer and appear on the bottom error line.
- **Terminal-side check-digit validation (`CHECK(M10F)` / `CHECK(M11F)`)** — type-time modulus 10 / 11 check digits are now validated before sending Enter / F1-F24; on failure the AID is not sent and an error is returned (operator error code 0015, error line 「検査数字エラー。」, same wording as ACS). `CHECK(M10)` / `CHECK(M11)` (without `F`) remain host-validated as before (the host sends no field control word for them).
- **Character-set validation by keyboard shift** — input is now rejected before writing according to the field's keyboard shift: digits-only fields (signed numeric / shift `D`) accept 0-9 only (operator error code 0010, same wording as ACS), numeric-only fields (shift `Y`) accept digits plus comma/period/sign/blank (0009), and alphabetic-only fields (shift `X`) accept letters plus comma/period/dash/blank (0008). Also enforced while typing in the viewer. Use `FIELD_EXIT_MINUS` to enter negative values; numeric format (decimal positions etc.) is still validated by the host as before.
- **DUP-key rejection wording now matches the real emulator** — pressing DUP in a field without DUP permission was already rejected; the error wording is now identical to ACS (operator error code 0019).
- **CAnn keys no longer send field data** — DDS CAnn (cancel) keys respond without field data on a real terminal (controlled by the function-key switches in the screen header, SOH bytes 5-7). The server previously always sent modified fields with any key, differing from real terminals; it now sends only the cursor position and AID code for masked keys (byte-identical to a real terminal, verified on real hardware). CFnn / Enter behavior is unchanged.
- **Paste now applies field character-set checks** — a real terminal performs normal field checks (numeric only, alpha only, ...) while pasting; a paste containing violating characters now stops before the offending field and reports the dropped count.
- **The local error line is now cleared by cursor movement keys** — matching the real emulator; the host-written error line behavior is unchanged.
- **Cursor progression (DDS FLDCSRPRG) support** — the field control word X'88nn' is now honored: TAB / Field Exit / DUP / auto-advance move the cursor to the designated entry field instead of the physically next one (Shift+TAB reverses via back-reference lookup). The entry numbering (bypass fields counted; continued / selection fields count as one) was verified byte-for-byte against a real IBM i. Field info now exposes `cursorProgression`. The same rule applies to viewer keyboard navigation.
- **Field-exit checks added** — leaving a field with a Field Exit key now performs the real terminal's checks: bypassing an empty `CHECK(ME)` field is an error (operator error code 0007), and a `CHECK(M10F)` / `CHECK(M11F)` check-digit mismatch is an error (0015, the value is left unchanged). Field Exit from the first position (clearing the field) passes, as on real terminals. TAB still moves without checks. All cases verified on IBM i 7.5.
- **FIELD_MARK key added** — `send_key` now accepts `FIELD_MARK`: writes the Field Mark character (X'1E') at the cursor and advances one position (processed like a data character key, unlike DUP's fill-to-end). Rejected with 0019 in fields without DUP permission. Host round-trip (send / accept / echo back) verified on real hardware.
- **Unified cursor advance after FIELD_EXIT / DUP** — `send_key` FIELD_EXIT / DUP now advance through the same path as TAB (selection-group representative, continued-field chain skip, window confinement, and cursor progression all apply consistently).

### Fixed (additional)
- **Viewer character-set checks for numeric-only / alpha-only fields did not work** — a syntax error in the check regular expression prevented the viewer's interactive script from loading at all (development-only issue in v0.1.13-beta; no released version was affected).
- **Viewer cursor mispositioned after Field Exit (numpad Enter) / DUP** — the advance target was off by one row and one column (a leftover from the old 0-based coordinate convention); now unified with TAB / auto-advance.
- **Edit-mask (DDS EDTMSK) edit characters were not displayed** — with EDTMSK, the host overwrites the field attribute positions with the edit characters (such as the "/" of a date), verified byte-for-byte on IBM i 7.5. The screen text previously blanked attribute positions unconditionally, showing "12 34 56"; it now shows "12/34/56" like a real terminal (`get_screen` / `get_field_text` as well). EDTMSK input/transmission already worked via the continued-entry-field mechanism.

### Fixed
- **AUTOTYP and session logs no longer record values typed into non-display fields (passwords)** — automating sign-on left the password in plain text in the execution and session logs (reported by another project). When the target field's display attribute is non-display, the logs record `<非表示欄・値は記録しない>` (non-display field, value not recorded) and the pre-send screen capture omits the value as well.

### Documentation
- **User Guide (Japanese / English): added "Field Input Rules and Pre-send Checks"** — Section 4.1 now lists the check messages with their meaning and remedies, and Section 6.3 covers the "confirm key does not send" symptom (both the PDF and the HTML served by the help viewer).
- **AI-GUIDE.md: added the input-check error responses** — Section 7 now lists the `error_code` / operator error codes returned on check violations and how to proceed (e.g. complete a right-adjust field with `FIELD_EXIT` before sending the AID).

### Notes
- **Why `CHECK(M10)` / `CHECK(M11)` never appear in field info** — without the `F` suffix, `CHECK(M10)` / `CHECK(M11)` are validated by IBM i data management when Enter is pressed, so the host does not send the terminal a field control word and no flag appears in the field info (working as designed). Only `CHECK(M10F)` / `CHECK(M11F)` (validated while typing) cause the FCW to be sent, in which case `selfCheckMod10` / `selfCheckMod11` appear (verified on real hardware).

---

## 2026-08-07 (v0.1.12-beta)

### Fixed
- **The bundled `config.example.json` now uses the correct viewer key** — the sample file shipped in the zip wrote the viewer port as `viewer.port`, a key the server does not read, so changing the port based on the sample had no effect (the viewer silently stayed on the default 5250). The correct key is `viewer.port_default`, which the README and manuals already documented; only the sample file was wrong. If you created your `config.json` from an earlier sample and changed the viewer port, rename the key `port` to `port_default`.

---

## 2026-07-28 (v0.1.11-beta)

### Fixed
- **`?` and a few other characters no longer reach the host corrupted** — typing a half-width `?` showed `?` on the 5250 screen, but storing that character in a database produced a control character (U+001A) instead. The code page tables filled positions that the host leaves unassigned with placeholder values, so the emulator sent an unassigned byte rather than the correct byte for `?`. IBM i substitutes a control character for any unassigned byte it receives, which is why the screen looked right while only the stored data was wrong. The same cause affected `?` on Host Latin (cp5035 / cp1399) and `!`, `i`, `ｰ`, `ﾊ` on Katakana Extended (cp5026_current). The tables now mark unassigned positions explicitly; sending never uses them, and displaying them shows a blank rather than a fabricated character that would hide the problem. DBCS characters such as full-width `？` were already correct and are unaffected.

---

## 2026-07-23 (v0.1.10-beta)

### Fixed
- **Full-screen programs that read all their input fields at once now receive typed values correctly** — some host programs (the full-screen FTP client, interactive REXX procedures that use `pull`, and similar) ask the emulator to return every input field's full contents in a single read. The server was replying in the format used for ordinary display-file screens (only the changed fields, with trailing blanks trimmed), so those programs misread the input; for example a full-screen FTP login was rejected (530) even with the correct password. The reply now uses the correct all-fields format (every input field returned at full length with blanks preserved), so full-screen FTP login succeeds and a REXX `pull` receives exactly the value entered. Ordinary display-file screens are unaffected.
- **The frame of the full-screen FTP exit-confirmation window is now drawn** — when quitting the full-screen FTP client (F3), the "End FTP client / 1=Yes 2=No" window appeared with no border (its frame characters were invisible). Windows that leave their frame characters to the terminal default now draw the standard thin-line box, matching ACS. Windows that specify their own frame characters are unchanged.

---

## 2026-07-20 (v0.1.9-beta)

### Fixed
- **Pasting into an input field now overwrites only at the cursor position, like typing** — previously a paste (Ctrl+V, or the paste tool) first erased the whole field and filled the cells to the left of the cursor with blanks, so pasting a character in the middle of a field could shift it one column to the right (showing " A") and wipe out text already in the field. Paste now writes the content starting at the cursor and leaves the cells before the cursor and after the pasted text unchanged, matching ACS and ordinary keyboard entry. DBCS shift-character (SO/SI) handling is unchanged.

---

## 2026-07-20 (v0.1.8-beta)

### Fixed
- **Input fields hidden behind a window can no longer be tabbed into or typed over** — when an application opened a window over a form (for example the "AWS resource" window reached with F9 on a billing-account edit screen), the underlying form's input fields that the window covered were still reachable with the field-tab key and still accepted keystrokes, so text could be typed into fields the operator could not even see, and characters aimed at a window field could be captured by a hidden background field. Fields overwritten by a window are now invalidated, so only the window's own fields are enterable and typing lands in the intended field (fields outside the window frame keep their attributes, matching ACS).
- **An input field's underline no longer bleeds past its end into empty space** — on ordinary form screens the underline of a field could continue in green past the field's right edge into the blank area below or beside it (for example the account-ID field on the billing-account edit screen). The underline now stops at the end of the field.

---

## 2026-07-16 (v0.1.7-beta)

### Fixed
- **Underlines of input fields continuing past the right edge of non-topmost windows no longer disappear** — the underlying-attribute continuation outside a window's right border (per the 5494 specification) was only applied for the most recent window. Found during a real-machine test with five stacked windows (an extension of the official IBM sample in *Application Display Programming*); the continuation is now applied for every stacked window (single-window screens are unaffected).
- **Closing stacked windows one by one (e.g. with F12) no longer turns all remaining window borders green or leaves background grid lines drawn over the windows** — IBM i closes windows using Save/Restore Screen, but the saved data did not include window, grid-line and colour state, so it was lost on restore. The save data now carries the GUI state, so border colours (WDWBORDER), background grid-line clipping, push-button frame hiding and field underlines stay correct no matter how many windows are stacked and closed (matching ACS).

### Changed
- **Expanded the AUTOTYP section of the User Guide** — added a figure of the AUTOTYP operation toolbar revealed by "▼ More" and a table describing each button (script selection / Rec / Run / Pause / Stop / Edit), in both Japanese and English.

---

## 2026-07-14 (v0.1.6-beta)

### Added
- **Multiple overlapping windows are now modeled** — when an application stacks two or more DDS windows (verified with the official IBM sample from *Application Display Programming*), background suppression now follows the real stacking order: grid-line boxes, selection indicators and input-field underlines of lower layers stay visible where they are not covered, and disappear exactly where a newer window overlaps.

### Fixed
- **Multi-row selection fields: choices outside the first row now reach the host** — selecting a radio/check choice on the second or later row of a selection field (SNGCHCFLD/MLTCHCFLD) was rejected by the host ("not selected"). The inbound selection-field address now follows the 5494 specification (field definition address + 1), so any choice registers.
- **Window overlay visuals** — a stray radio mark no longer appears on a window's top border, and radios on the row just below a window no longer degrade to a plain dot (one-row offset in the window exclusion band). Background grid-line boxes now remain visible around a window (previously all were hidden; only push-button frames are hidden, matching ACS) and are clipped by the window area including its side attribute columns.
- **Cursor no longer lands on a non-existent column after sign-on** — when the first input field's attribute byte sits at the end of a row, the cursor now wraps to the next row (e.g. 2/1 instead of the invalid 1/81), matching ACS. The same row-wrap correction is applied consistently to cursor movement, field addressing in the AID stream, field matching and local echo.

---

## 2026-07-14 (v0.1.5-beta)

### Changed
- **Expiry extended to 2026-09-30** — two months beyond the previous 2026-07-31. From 2026-10-01, startup and connection are automatically refused.

### Fixed
- **Push button (PSHBTNFLD) selection-cursor highlight no longer shows a white cell at its right edge** — when the cursor is on a push button, the highlight now uses one uniform colour (the host-specified selection-cursor attribute) across the whole button, matching ACS. Internally, button display attributes now follow the 5494 specification (leading attribute = choice colour, ending attribute X'20'). Button width and position are unchanged.

---

## 2026-07-08 (v0.1.4-beta)

### Added
- **Signed-numeric field entry via `send_text`** — new Field Exit keys (`FIELD_EXIT` / `FIELD_EXIT_PLUS` / `FIELD_EXIT_MINUS`) let you type a value and set its sign in one call, so **negative** signed-numeric values can now be entered (e.g. `key:"FIELD_EXIT_MINUS"` → `-1234567`).

### Fixed
- **Signed-numeric fields now behave like ACS / the host** — a signed-numeric field (e.g. `7S0`) accepts only its digit capacity (7 digits; the last position is the sign), the sign is transmitted as proper zoned decimal, and negative values are accepted by the host. Previously an extra digit was accepted and the value was rejected by the host as a decimal-data error. Cursor position and Field-Exit-Required (FER) behaviour now match ACS.
- **Single-character field fill** — filling a one-character field (e.g. an auto-enter field) now correctly triggers auto-enter / auto-skip (previously the field-full condition was missed for single-character input).

---

## 2026-07-08 (v0.1.3-beta)

### Changed
- **`send_text` cursor now matches keyboard typing** — after typing text into a field, the cursor is left at the end of the text (or, when the field fills exactly, it moves to the next input field), just like typing on a real keyboard. This is the cursor position sent to the host, so cursor-sensitive operations (F4 prompt, subfile selection, context help) behave correctly.

### Fixed
- **`send_text` with a key but no text now positions the cursor first** — key-only submissions with an explicit row/column (for example context help / an F4 prompt at a specific position) now move the cursor before sending the key.
- **The server no longer terminates when a connection attempt fails** — connecting to an unreachable or refused host now returns an error cleanly instead of crashing the MCP process.

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
