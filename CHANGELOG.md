# Changelog

User-facing changes in the IBM 5250 MCP Server distribution, most recent first.

> 日本語版は [`CHANGELOG_ja.md`](CHANGELOG_ja.md) を参照してください。 (Japanese version: see [`CHANGELOG_ja.md`](CHANGELOG_ja.md).)

Time-limited beta release. **Windows (x64) only. Valid until 2027-01-31.** The source code is not
published. Verified in a Japanese environment only (primary language 2962, QCCSID 65535).

---

## 2026-11-26 (v0.1.19-beta)

### Added
- **Colors of the 5250 screen** — The window opened from the "Colors" icon on the toolbar sets the colors of the same elements as the "Colors" of IBM i Access Client Solutions (the 7 field colors, the 4 OIA indicators and the OIA background, the screen background, the active field, the crosshair ruler and the column separator). The cursor inverts the colors under it, as in IBM i Access Client Solutions. IBM i Access Client Solutions color settings (*.col) can be imported and exported from the same window. "Save" records them in the configuration.
- **Aspect ratio of the 5250 screen** — Drag the handle at the bottom-right of the 5250 screen to change the aspect ratio; double-click it to restore the default. The number of columns and rows and the column positions of characters do not change (only the column width and row height change). "Save" records it in the configuration.
- **Crosshair ruler** — A toolbar button shows or hides a horizontal line on the cursor row and a vertical line on the cursor column (hidden by default). The line color can be set in the "Colors" window, and the crosshair ruler color in *.col files is read and written.
- **Cursor blink and shape** — The cursor icon to the right of the mouse wheel button on the toolbar opens a window to choose whether the cursor blinks (on/off), the blink speed (a slider) and its shape (block/underline). The default is a blinking block. It is saved the same way as the shift character display.
- **User-defined character list** — Copy is now an icon only, and the explanatory text was removed. The list is shown in the display language only (Japanese or English). Hovering over a glyph enlarges it.
- **Display of user-defined characters (UDC)** — "Get user-defined characters" on the configuration screen gets the UDC glyphs (QIGC3232 and QIGC2424) of the IBM i over FTP, and UDCs on the 5250 screen are drawn with those glyphs (32×32 first; a UDC without a glyph is drawn as a dotted box with a diagonal line). "Show user-defined characters" opens a list (EBCDIC, PC (Shift-JIS) and Unicode codes and glyphs) in a new tab, and the copy icon lets you paste a UDC into the 5250 screen. UDCs are handled as Unicode private-use characters (X'6941' = U+E000 onward). See Administrator Guide section 2.1.

### Fixed
- **The empty box below the 5250 screen is no longer shown** — It showed the communication phase for a moment only, so usually only an empty box was visible. The 5250 screen gets that space.
- **Pressing Help after a host error (such as "Function key not allowed") did not show the message help** — As in IBM i Access Client Solutions, the IBM i "Additional Message Information" is now shown (Help is sent as Help in the error state, as the 5494 specifies). Reset after returning from the help restores the original last line.
- **The glyphs of GUI elements were changed to be close to those of ACS** — Window borders, scroll bars, selection indicators and so on. The menu bar separator is a double line, and the top corners of a pull-down have the same shape as in IBM i Access Client Solutions.
- **In an English (037 etc.) session, programs that read the screen (QsnRtvScr etc.) failed with CPFA306** — The query reply now reports the ideographic (kanji) capability only for the Japanese terminal (5555).
- **A paste that entered no characters (the field did not accept the first character) still marked the field as modified** — The field content did not change, but the field was sent to the host as modified with the next AID key, and a following Erase Input cleared the value the host had displayed. As when a typed key is rejected, the field is no longer marked modified (5494: pasting is the same as typing the keys).
- **Copying a multi-row selection copied nothing when a later row started on the right half of a double-byte character** — As in IBM i Access Client Solutions, nothing is copied only when the first row starts on a right half; on later rows that character is dropped and the rest is copied.
- **Backspace right after an SI (shift-in) stopped on the SI column** — As in IBM i Access Client Solutions, the cursor skips the SI and moves to the start of the preceding double-byte character.
- **Paste while input-inhibited (X SYSTEM) was written to the screen immediately** — As in IBM i Access Client Solutions, it is discarded (typed characters are still kept as type-ahead).
- **Arrow, Delete, Backspace and End (Erase EOF) keys pressed while input-inhibited had no effect after the unlock** — Like typed characters, they are now kept as type-ahead and applied in order after the unlock (as in IBM i Access Client Solutions).
- **A right click (also Shift + right click and Shift + middle click) moved the cursor to the clicked position** — As in IBM i Access Client Solutions, it does nothing unless the host defines the mouse button.
- **A middle click only moved the cursor** — As in IBM i Access Client Solutions, it moves the cursor and then sends Roll Up (next page), unless the host defines the button.
- **In an English (037 etc.) session, confirming kanji with the IME was rejected ("Field data must be alphanumeric")** — As in IBM i Access Client Solutions, characters not in the code page are written as "?" one column each (same as paste).
- **The slider position and size of short scroll bars differed from IBM i Access Client Solutions** — Now drawn the same way as the 27 cases measured in IBM i Access Client Solutions.
- **Scroll bar clicks sent different data from IBM i Access Client Solutions** — The cursor is no longer moved (only when the definition requests it), the scroll bar column is sent correctly, an arrow sends nothing when the slider is already at that end, and a click in the shaft sends the scroll bar size as the increment (5494 specifies 0; this follows IBM i Access Client Solutions as an exception).
- **Opening the viewer started in shared input mode instead of exclusive** — As described in User Guide section 2.4, exclusive is the default at start-up (the first exclusive request did not identify the browser).
- **Message when the file of a FILE:// URL hotspot does not exist** — As in IBM i Access Client Solutions, it is now "Path not found: FILE://…" (the URL as shown on the screen; previously "File not found: \\…").
- **Window borders had no shadow** — As in IBM i Access Client Solutions, the right and bottom borders have a shadow.
- **The second and later push buttons placed side by side in one field were shifted one column to the left** — They now use the same spacing as IBM i Access Client Solutions (button frame width plus the gap between buttons).

## 2026-11-24 (v0.1.18-beta)

### Fixed
- **Viewer selection could not select a single column** — Mouse drag and Shift+Arrow can now select one column (including half of a double-byte character). Shift+Arrow moves the corner of the selection without moving the cursor (as in IBM i Access Client Solutions).
- **Copy included a whole double-byte character when only half of it was in the selection** — As in IBM i Access Client Solutions, that character is not copied. When the left edge of the selection is the right half of a double-byte character, nothing is copied. Shift-character columns become blanks.
- **Grid lines of the screen behind a window showed through inside the window** — As 5494 specifies (creating a window clears its area), grid lines inside the window area are cleared.
- **Double-byte line-drawing characters (such as ━) were drawn in a different font from other characters and did not join** — They are now drawn in the same font as the rest of the screen.
- **Clicking the right half of a double-byte character put the cursor in the middle of the character** — As in IBM i Access Client Solutions, it goes to the start of the character.
- **Paste put single-byte characters into J and G fields (double-byte only) and mixed single- and double-byte characters in E fields** — As 5494 specifies (paste is checked like typing), each character is checked and the paste stops at the first error. As in IBM i Access Client Solutions, no message is shown.
- **Paste in insert mode overwrote, and the cursor moved after a paste** — In insert mode the following characters are now shifted right. After a paste the cursor stays at the paste start in both modes (as in IBM i Access Client Solutions).
- **Paste stopped silently when there was no room to insert, did nothing at a protected position, and wrote nothing (not even "12") when "12A" was pasted into a numeric field** — It now matches measurements in IBM i Access Client Solutions. When there is no room to insert, it inserts what fits, shows "No room to insert data", and leaves the cursor where it stopped. At a protected position, it pastes into the next input field (the cursor does not move; 5494 specifies an operator error, so this is an exception that follows IBM i Access Client Solutions). In fields such as numeric-only fields, it writes the characters before the first one that is not accepted (no message).
- **Pasting kanji or half-width katakana in an English (037 or other single-byte) session garbled the screen** — they were written as double-byte characters. As in IBM i Access Client Solutions, characters not in the code page are now written as "?" in one column (the host receives the same characters).
- **Mixing half-width letters and half-width katakana in an E-type (DBCS-either) field was rejected** — In 5026 (katakana) sessions, the input check of MCP send_text and AUTOTYP counted half-width katakana as double-byte and rejected "Aｱ" with "Field data must be alphanumeric". It now uses the code page table, like the viewer.
- **In an error line inside a window, the error text overlapped the area left of the window when that area had half-width katakana** — half-width katakana were counted as two columns.
- **Half-width katakana confirmed with the IME was rejected with "Field data must be alphanumeric" in fields that do not accept double-byte characters** — In 5026 (katakana) sessions, characters confirmed with the IME are now classified as single- or double-byte by the code page table (typing one character at a time was fixed on 2026-11-10).
- **STRPCCMD passed different characters to the PC than IBM i Access Client Solutions** — With a Japanese host code page, a yen sign (for example X'B2') was passed as `¥` (U+00A5), so it did not work as a Windows path separator. It is now passed as `\` (U+005C), as IBM i Access Client Solutions does. After the character set switch (Ctrl+F3), the string is read with the switched table (the meanings of X'5B' and X'E0' are swapped). See "STRPCCMD characters" in the Administrator Guide.
- **When the same STRPCCMD string was issued twice in a row, the second one was neither run nor answered** — The second one is now run too (previously the IBM i program stopped, waiting for the reply).
- **In the viewer, the status line did not show X II while a host error message (for example, "Function key not allowed") was displayed** — like IBM i Access Client Solutions, a red X II is now shown. It clears with the next key.
- **After a host error message (for example, "Function key not allowed"), AUTOTYP waited 30 seconds before the Enter key and stopped with exit code 30, and typed characters did not clear the error line** — matching measurements in IBM i Access Client Solutions, a character typed after the error restores the error line and is entered, and Enter, F keys and Roll are sent without waiting. This is an exception that follows the behavior of IBM i Access Client Solutions, not the 5494 rule (after an error, keys other than Reset, Help and Attn are ignored).
- **In the viewer, a cursor position set by clicking right after Reset (left Ctrl) could jump back to the position before Reset a moment later** — It happened when responses were slow, and the next typed character or Dup went to the old position. Reset no longer moves the cursor (as on a real terminal).
- **AUTOTYP could not type text starting with "^" or "[", and conversion from ACS macros dropped the text after "#"** — Write a leading "^" or "[" in a data line as `\^` or `\[` (like `\#` for "#"). Operation recording (Rec) and macro conversion now write it this way.
- **The User Guide described the Home key incorrectly** — It moves to the home position (the cursor position the host set for the screen), and sends Record Backspace if the cursor is already there.
- **The viewer did not open when its port fell in a range reserved by Windows (an excluded port range)** — As for a port in use, the next port is now tried. On PCs where Hyper-V or similar reserves ports in the 5250 range, connecting could return no viewer_url.
- **In a word-wrap continued field that accepts double-byte characters (DDS type O), an English word after kanji was split at the end of a row (when a string was entered at once, for example with MCP `send_text`)** — As 5494 specifies (in type O fields word wrap applies to single-byte data) and as IBM i Access Client Solutions does, the word now moves to the next row. Typing one character at a time already behaved this way. In the same fields, Insert and Delete with kanji present also lost the blank between words and did not re-wrap the words; this is fixed too.
- **A new input field that the host defined before the last input field on the screen was treated as an input field** — As 5494 specifies and as IBM i Access Client Solutions does, it is no longer an input field (this does not happen with DDS screens, only with programs that write the data stream directly).
- **Grid lines (DDS GRDBOX, GRDLIN, etc.): the rules inside ruled boxes and repeated lines were not shown, and clearing an area also cleared lines outside it** — Grid lines are now held and drawn as a top line and a left line for each character position, as described in the IBM Personal Communications 5250 manual (chapter 11) and the 5494 reference. Rules inside ruled boxes (horizontal, vertical, both) and repeated lines (count and interval) are shown. Clearing grid lines (CLEAR GRID LINES, GRDCLR) clears only the lines inside the area (previously the area was read one byte off and any grid line touching the area was removed entirely). Color and line style are a single color and a solid line, as in IBM i Access Client Solutions. Because grid lines are a feature of DBCS display stations (5555) only, a session connected with a single-byte terminal type (3179, 3477) now answers the query that grid lines are not supported (previously it claimed support regardless of the terminal type).
- **MCP `send_text` and AUTOTYP did not perform mnemonic selection in selection fields such as menu bars** — A character typed in a selection field now selects the choice with that mnemonic (the underlined character), as the keyboard does (5494 p.15-84; matches measurements of IBM i Access Client Solutions). On an auto-enter selection field, Enter is sent. Previously the character was written as text and failed, so AUTOTYP recorded in the viewer could not be played back.
- **The AID-key path (ENTER, F keys) of MCP `send_text` could not write from the middle of a field** — As the keyboard does, it now overwrites from that position and then sends the AID key (the rest of the field is kept). Previously it was rejected with `NO_TARGET_FIELD` (the TAB path was fixed on 2026-11-17).
- **In a multi-row field, the second row of a run of double-byte characters (such as kanji) that continued past the end of a row was garbled** — For example, typing "漢字あい" from row 3, column 75 garbled row 4. As in IBM i Access Client Solutions, the next row is now also shown as double-byte characters. As in ACS, the character that straddles the row boundary is drawn in halves (the left half in the last cell of the row, the right half in the first cell of the next row). In the screen text (`get_screen`), these two cells are blanks.
- **In the viewer, a character typed after a signed numeric field was full overwrote the last digit** — For example, typing "12345678" into a 7-digit (7S 0) field produced "1234568". As in IBM i Access Client Solutions, characters typed after the field is full are now rejected with the field-exit-required error message. Field Exit, Field+, Field−, Reset or moving the cursor clears the state. MCP `send_text` now also rejects digits beyond the field with `FIELD_EXIT_REQUIRED` (previously they were truncated and processing continued).
- **In the viewer, characters typed after filling a field kept overwriting its last position** — The cursor should advance to the next field automatically, but the server's response moved it back. Typing now continues in the next input field, as 5250 specifies.
- **In the viewer, letters could be typed into DBCS-only fields (DDS type J)** — They are now rejected with the "DBCS input required" message, as for G fields (the MCP path already rejected them).
- **MCP `send_text` with `key=TAB` moved to a different field than the keyboard when the field became full** — The keyboard advances to the next field when the field fills and then tabs from there; `send_text` stopped one field short. Also, when an auto-enter (CHECK(ER)) field becomes full, `send_text` now tries Enter as the keyboard does, and stops on terminal checks such as mandatory enter (CHECK(ME)) (previously it tabbed without the auto Enter).
- **In the viewer, Reset (left Ctrl) did not work while a field exit was required (for example after a CHECK(FE) field was filled), and the next character was lost** — Reset now clears the state. Moving the cursor with the mouse also clears it.
- **In the viewer, characters typed while the keyboard was locked could still be entered after Reset, SysReq or Attn** — Characters waiting to be sent are now discarded too (as 5250 specifies, all unprocessed type-ahead is discarded).
- **Specifying code_page "cp5026" in connect_5250 garbled the screen** — The alias with the "cp" prefix was not converted to the proper name and the screen was read with a US code page. It is now treated as cp5026_current, like "5026". An unsupported value is now rejected with an error listing the valid values, instead of silently using another code page.
- **In the viewer, typing continuously into a numeric field with an edit mask (EDTMSK) did not move past the protected separators (parentheses, dashes, slashes, etc.) and the digits were scrambled** — For example, 11 digits typed into a phone-number-shaped field produced "8(901)000-0000". Typing now skips the separators and continues in the next part.
- **AUTOTYP did not perform the automatic Enter when an auto-enter (CHECK(ER)) field became exactly full** — It now tries Enter as the keyboard does, and stops with error code 24 on terminal checks such as mandatory enter (CHECK(ME)).
- **In the viewer, typing in insert mode into a full field discarded the character without an error, and the cursor could jump to another line** — As in IBM i Access Client Solutions, the "no room to insert data" message is now shown and the cursor stays where it is.
- **In the viewer, an input error message could disappear when a screen update arrived afterwards** — Only "X II" remained. The message now stays until the error is cleared.
- **Passwords entered with AUTOTYP `^PWD` appeared in screen text and logs** — Values entered with `^PWD` are now replaced with `*` in MCP responses (`final_screen` of `run_autotyp`, `get_screen`, `get_field_text`, etc.), in the screen data sent to the viewer, and in session logs (screen images and send traces; `**` in the hex of send traces). They are still sent to the host. Values written as plain strings in a script are displayed as before.
- **In the viewer, the mouse pointer is now an arrow over push buttons** — Previously it was the text caret (I-beam), as over other text. Like URL hotspots (hand pointer), the shape shows that the item can be clicked.
- **Characters typed while the keyboard was locked (waiting for an AID response) were still entered after Reset, SysReq or Attn** — As the 5494 specifies (the type-ahead buffer is cleared), characters held in the viewer are now discarded too.
- **Print, SysReq, Reset or Attn pressed right after fast typing could reach the server before the characters typed earlier** — When the keyboard is not locked, keys are now processed in the order pressed (while locked they are still sent immediately).
- **When several host screens arrived at once, the last one was shown late** — The per-screen wait was shortened from 0.1 to 0.03 seconds.
- **While someone kept typing quickly, other tabs (shared session) did not update** — The screen was not sent until typing stopped. The first character is now sent immediately, and the screen is sent every 0.05 seconds while typing continues.
- **Typing quickly, for example right after opening the viewer, could make characters overwrite the previous ones and get lost** — For example, typing "GURICAT" quickly on the sign-on screen produced "RAT". When a screen arrived from the host mid-typing, the cursor jumped back to that screen's older position. The viewer now keeps its own cursor until the server has processed the typed characters.
- **Delete, Backspace, Erase EOF and Dup were slow to appear in the viewer** — Like typed characters, they are now shown right away using the same processing as the server.
- **In the viewer, a character typed quickly right after kanji could land one position off and erase the previous character** — For example, typing "漢字テスト" followed quickly by " ABC" lost the space ("漢字テストABC"). The viewer decided the next position before it knew about the shift-in (SI) that ends the kanji. The viewer now applies what you type with the same processing as the server and sends to the same position.
- **In the viewer, typed characters could briefly disappear (flicker), and kanji and insert-mode input were slow to appear** — What you type is now shown right away, using the same processing as the server, without waiting for the server's confirmation. When the server's result arrives, the screen is redrawn from it (characters the server rejected disappear). Characters no longer vanish mid-way when typing fast, and kanji and insert-mode characters appear without waiting for the network round trip.
- **Typing half-width katakana and similar characters in the viewer's insert mode inserted an ideographic space** — Insert mode treated every non-ASCII character as double-byte, so single-byte characters of the code page (half-width katakana, etc.) became an ideographic space. They are now inserted as single-byte characters, the same as in overwrite mode.
- **In the viewer, the home position of the Home key could be stale** — after an AID key sent from MCP or another tab, or after the first screen, Home moved to an old position or did not work as Record Backspace at the home position. The viewer now uses the position set by the host (or the first input field).
- **In the viewer, Reset after moving the cursor with a click moved the cursor back to where you last typed** — Reset does not move the cursor.
- **In the viewer, the mouse pointer is now an arrow over check boxes, radio buttons, menus and scroll bars as well** — the same as usual Windows behavior (push buttons already did). The shape does not change while dragging to select a range.
- **In a continued field with word wrap (WRDWRAP), typing one character at a time split a word at the segment boundary** — word wrap was applied only when text was written at once (send_text, paste). Typing now also moves the word to the next segment.
- **In the viewer, Backspace right after a kanji moved back only one position and stopped on the right half of the kanji** — it now moves to the start of the kanji.
- **Paste (Ctrl+V, paste_text) could go to the start of the field instead of the cursor position** — when pasting on the second line of a multi-line field (such as the command line), or after blanks left by Erase EOF.
- **A kanji typed at an empty position (for example after Erase EOF) was moved to the start of the field** — it now goes to the cursor position.
- **MCP `send_text` after `send_key INSERT` overwrote instead of inserting** — in insert mode the text is now inserted, shifting the rest to the right. `send_key RESET` turns insert mode off.
- **MCP `send_text` could write the opposite kind of character into an either-type (E) field that already had data** — when writing in the middle of a field or in insert mode. It is now rejected, the same as in the viewer.
- **MCP `send_key` Erase EOF, Delete and Field Exit processed only the current segment of a continued field (such as a numeric field with an edit mask)** — they now process the whole field.
- **The status line showed "英数 半角" (alphanumeric/half-width) with single-byte code pages** — it is shown only with code pages that use double-byte characters.
- **Strings such as `**` could turn into kanji in the field list of session logs** — Strings without SO/SI were read as double-byte characters. They are now read with the same rule as the screen image (double-byte characters only between SO and SI).
- **Tab to a field whose attribute is in the last column of a row put the cursor off the screen (for example, column 81 of an 80-column screen)** — the cursor now goes to column 1 of the next row, the same as ACS.
- **Unselected choices in multiple-choice selection fields (check boxes) showed "." as the indicator** — they now show a blank (the 5494 default, the same as ACS). This is the indicator in the screen text (`get_screen` and so on); the □/☒ drawing in the viewer is unchanged.
- **Selecting a choice with SPACE or a click did not change the indicator ("/" or ".") in the screen text** — the indicator is now rewritten to match the selection.
- **A window title without an attribute got a blank column on each side** — the blanks are made only when the title has an attribute, the same as ACS (the border line touches the title).
- **The viewer's operation recording (Rec) did not keep typed characters or cursor positions** — only the AID keys were recorded, so playback did not type anything. Typed characters, editing keys (Erase EOF, Delete, Field Exit and so on), insert mode, clicks (choices and push buttons) and cursor positions (`^CURrrcc`) are now recorded. Characters typed into non-display fields (such as passwords) are not recorded; `^PWD` is written instead.
- **In word-wrap (WRDWRAP) continued fields, a word ending exactly at the end of a segment was not moved** — it now moves to the next segment (a word as long as the segment is not moved), the same as ACS.
- **Word-wrap continued fields were not reformatted after insert and Delete** — a word that no longer fits after an insert now moves to the next segment, and when Delete frees space in an earlier segment, the following words move back (as in the 5494 specification and ACS). Previously an insert could drop the blank between words. Field Exit no longer packs data into the earlier segment (which also dropped the blank between words); like Erase EOF, it does not reformat, the same as ACS.
- **On the viewer's System Request (SysReq) input line, Delete caused the "cursor in protected area" error** — you can now move the cursor with ← → and delete characters with Delete within the input line, the same as ACS. When the Backspace setting is "delete", Backspace deletes the previous character.
- **Leading and embedded empty positions (nulls) of input fields were sent to the host as X'00'** — they are now sent as blanks (X'40'), as in the 5494 specification and ACS (they stay nulls for READ MDT ALTERNATE).
- **Typing kanji one after another at a middle position of an empty input field moved them to the start of the field from the second character** — they now stay where they are typed, the same as ACS.
- **In continued fields (CNTFLD), kanji in segment 2 with segment 1 left empty reached the host shifted by 2 positions** — the empty part of segment 1 is no longer packed. Kanji started near the end of a segment continue from the start of the next segment, the same as ACS.
- **Reconnecting to the same host within 5 seconds after closing a viewer window could disconnect the new session** — the automatic disconnect for the closed window hit the new session that reused the same ID.

### Changed
- **The browser tab icon is now a terminal** — The 5250 screen tab shows a terminal icon with a gray frame and green text (previously the default globe icon).
- **The screen data sent to the viewer no longer contains raw screen-buffer bytes** — `debugBuffer` (raw bytes) in the screen-state API (`/api/sessions/<session>/screen`, etc.) was replaced by `cellKinds`, which carries only what rendering needs (attribute and control values as is; every character as X'41').
- **AUTOTYP `^SPACE` now works like the space bar** — on a choice in a selection field it selects the choice; in an input field it types a blank (previously it always did the selection operation).

### Added
- **AUTOTYP `^CURrrcc`** (an MCP extension) — puts the cursor at row rr, column cc (0-based, counted the same way as `^DSPrrcc` in the original AUTOTYP). It is used to convert ACS macro inputs that specify a row and column.
- **The viewer can run ACS macros (.mac)** — .mac files in the `scripts` folder appear in the AUTOTYP list. Running one converts it to AUTOTYP and runs it (the same happens when `script_file` of the MCP `run_autotyp` is a .mac). Unless specified, the overall time limit is the macro's per-screen wait limit times the number of screens.
- **AUTOTYP `^CLICKrrcc`** (an MCP extension) — clicks the mouse at row rr, column cc (0-based). It is handled the same as a click in the viewer (a URL hotspot opens; GUI parts such as push buttons and choices are sent to the host; elsewhere the cursor moves). Without a viewer, only the server's mouse handling (GUI parts and the cursor) is done.
- **AUTOTYP `^WAITSTR row col end-row end-col [!]text`** (an MCP extension) — waits until the text appears in that area of the screen (1-based), or with `!` until it disappears. If it does not come, the exit code is 31. It is used to convert the screen recognition of ACS macros.
- **AUTOTYP `^ALTVIEW`** (an MCP extension) — the character set switch (Ctrl+F3 in ACS). Like the viewer's switch button, it switches the displayed single-byte characters (katakana / lowercase letters).

### Fixed (ACS macro → AUTOTYP conversion)
- **The row and column of inputs that specify a position were dropped** — they are now converted with `^CURrrcc`.
- **Leading and trailing blanks of inputs were dropped** — they are now converted to `^SPACE` lines.
- **Waits (pause, commwait) were written as `^WAITX` in 100 ms units** — they are now written in 500 ms units (previously the wait was five times too long).
- **Screen recognition (`<string>`) was dropped and the script went on** — it ran ahead after screens where the host takes time. It is now converted to `^WAITSTR`, which waits until that screen comes. Screens are converted as a single path in the order they are written, so branches with several candidate next screens are not converted.
- **Mouse clicks (mouseclick) were dropped** — they are now converted to `^CLICKrrcc`. ACS records a double-click as a single click, so screens that act on a double-click (for example, running an item by double-clicking it) do not act on playback (the same happens in ACS's own playback).
- **Encrypted inputs (recorded passwords) were typed as the encrypted string** — they are now converted to `^PWD` (the password is asked when the script runs).
- **A converted script stopped with exit code 25 on a host error message (for example, "no matching key" in a search)** — ACS macros do not stop on error messages, so `^ERMOFF` is added at the start of the converted script.
- **Keywords such as `[altview]` and `[left]` were dropped** — `[altview]` (character set switch) is converted to `^ALTVIEW`, `[left]`, `[right]`, `[up]` and `[down]` to cursor moves, and `[pageup]` and `[pagedn]` to page up and page down.

---

## 2026-11-10 (v0.1.17-beta)

### Fixed
- **Background push button frames remained over windows drawn with characters (such as help)** — the "remove all GUI constructs" command (REMOVE ALL GUI CONSTRUCTS) sent by IBM i was not processed. Following the 5494 manual (pp. 15-43 and 15-87), windows, selection fields, scroll bars and push button frames are now removed. The buttons of the original screen come back when the window closes. The commands that remove a single window, selection field or scroll bar (REMOVE GUI WINDOW / SELECTION FIELD / SCROLL BAR FIELD) and the command that lifts the window cursor restriction (UNRESTRICTED WINDOW CURSOR MOVEMENT) are now processed too (they were skipped before).
- **Background push button frames remained inside windows** — frames of background or removed buttons inside a window (CREATE WINDOW) are no longer drawn. The window's own button frames are unchanged.
- **Selection fields (radio buttons, check boxes, push buttons, etc.) were sent to IBM i in a form different from the IBM manual** — following "5494 Remote Control Unit Functions Reference" (pp. 15-67/15-68), (1) only selection fields the user selected (changed) are sent, (2) check boxes are sent as one byte per choice (X'F1' selected / X'00' not selected), and (3) selection and input fields are ordered by screen position. Examples fixed: a search returned "no search word entered" even with a word typed; checking a box and pressing Enter cleared the box and showed the same screen again.
- **Clicking a push button did not tell IBM i which button was pressed** — the chosen button's data is now sent. Example fixed: clicking a push button in a window did not start the process.
- **Check boxes in windows were shown as "." / "/" instead of □/☒**.
- **Text with display attributes 2A, 2B and 2E (red, blink) did not blink** — it now blinks as the IBM manual defines. The light bulb toolbar button turns blinking off (IBM i Access Client Solutions does not blink by default).
- **On screens showing a window, input fields outside the window were sent to IBM i again** — fields cleared by SOH (screen rebuild) are no longer sent.
- **Choosing a menu bar or pull-down choice with a click or the space bar did not reach IBM i** — for selection fields defined to return immediately (auto-enter), the Enter key and the chosen choice are now sent at the moment of selection. Example: clicking a menu bar item shows its pull-down, and choosing a pull-down item passes its number.
- **Double-clicking did not send the Enter key** — a double click that the screen does not assign is now the Enter key, as in the IBM manual "5494 Remote Control Unit Functions Reference" (except on buttons, choices and scroll bars). A second click or a double click at the position the cursor had just moved to was sometimes taken by the hidden input field used for Japanese input and had no effect; this is fixed.
- **After switching the language, the tab title kept the previous language for up to 3 seconds** — it now changes immediately.
- **After closing a screen shown over another one (such as help), the input fields and push buttons of the original screen no longer worked** — when IBM i restored the screen (RESTORE SCREEN), the next write to the display cleared the field list (according to the IBM manual "5494 Remote Control Unit Functions Reference", a write to the display does not clear it). Push buttons, choices and scroll bars of the restored screen also became plain input fields. Example: after opening help with F1 and closing it, clicking "+" (pull-down) or the function key buttons did nothing.
- **The ATTN key ended the session (CPF5140 on IBM i)** — the reply to the QUERY STATION STATE command that IBM i sends after ATTN was the reply to a different query. It now follows the format in the IBM manual "5494 Remote Control Unit Functions Reference" (SC30-3533-04).
- **Half-width katakana could not be typed in the viewer with 5026 Katakana** — whether a typed character is single-byte or double-byte is now decided from the session's code page table.
- **Fields with attributes 34, 35 and 36 (turquoise / yellow, column separator + underscore) showed no underline and no left column separator** — the column separator drawing overwrote the underline. As in IBM i Access Client Solutions, the left and right separators and the underline are now drawn. Example: one-position option fields in a list.
- **The separator line below a menu bar was not drawn** — the separator specification sent by IBM i ("5494 Remote Control Unit Functions Reference" Table 15-20) was skipped. As in the manual, the separator is written as characters on the screen, so it disappears when the screen changes. It is a solid line (IBM i Access Client Solutions draws a double line).
- **A selected menu bar or pull-down choice was not shown in white** — following the defaults in the "5494 Remote Control Unit Functions Reference" (p.15-81), a selected choice in a selection field without indicators is now white, and the selection cursor on it is reverse white (for example, the menu bar item while its pull-down is open, and the pull-down item at the moment Enter selects it). The mnemonic underlines also disappeared while a pull-down was open.
- **X SYSTEM (input inhibited) hardly ever appeared in the OIA after a mouse click (push button etc.)** — with a fast host response it ended before being shown. It now also appears in other tabs of a shared session.
- **Unavailable choices were not displayed, and the numbers of later choices were shifted** — choices that IBM i marks as unavailable ("5494 Remote Control Unit Functions Reference" Table 15-19) were neither drawn nor counted (the manual, p.15-68, counts them). Selecting a later choice could send the wrong choice number to IBM i. They are now displayed in the specified color (Table 15-21, default blue), and the "unavailable" and "cannot accept a cursor" settings are honored. Example: opening a pull-down from a menu bar did not change the color of that menu bar item (IBM i sends the opened item as "unavailable" with a display color), and the cursor could not move to the other menu bar items afterwards.
- **Tab on a menu bar and similar fields skipped the choices and went to the next field** — in a selection field where IBM i specifies that Field Advance works like Character Advance ("5494 Remote Control Unit Functions Reference" Table 15-18, flag byte 2 bit 4), Tab now moves choice by choice and goes to the next field after the last choice. Shift+Tab goes in reverse.
- **When the next window appeared, the mnemonic underline disappeared from the pull-down text left on the screen** — as in the manual (p.15-53), the underline is written as part of the screen characters and remains after the selection field is gone.
- **The underline of reverse + underscore characters was invisible** — 5250 is character based, so reversing a character reverses its underline too. With green and other reverse colors the underline was drawn in the background color (for example attributes 25, 2D, 35 and 3D, and the mnemonic underline under the selection cursor). It is now drawn in the reverse color (black).
- **When IBM i redefined a selection field at the same position, its choices were duplicated** — as in the manual (p.15-67), the new definition replaces the old one and leftover choices are erased.
- **The arrow keys could not leave a selection field (menu bar etc.) across the screen edge** — for example, ← from the leftmost choice of a menu bar, or ↑ from the menu bar. The cursor now wraps to the opposite edge, as outside selection fields.
- **The radio button / check box symbols (characters used instead of ○◉□☒) and their color specified by IBM i were ignored** — "5494 Remote Control Unit Functions Reference" Table 15-21 (indicator color) and Table 15-22 (indicator characters) were skipped. They are now used when specified (unchanged otherwise).
- **Menu bar and pull-down choices showed a radio button circle on the left** — the indicator was drawn even when IBM i specified "no indicators" for the selection field (IBM "5494 Remote Control Unit Functions Reference" Table 15-18).
- **Keys and text sent right after connecting, before IBM i's first screen arrived, were discarded** — input now waits up to 10 seconds for the first screen (viewer and MCP send_key / send_text / paste_text).
- **The ENTER button on the bottom toolbar sent nothing** — the keyboard Enter key worked. Example: you could not leave the SEU display screen with the ENTER button.
- **STRPCCMD commands containing Japanese (DBCS) text were garbled** — e.g. the file in `START "\\…\IBM i 機能拡張詳細_1.2.ppt"` could not be opened. Japanese text between shift characters (SO/SI) is now read with the same tables as the screen. Whether single-byte letters are lowercase or half-width katakana follows the toolbar character set toggle (A / ｱ, Ctrl+F3).
- **With 5026 Katakana (cp5026_legacy = old 290), lowercase letters sent by the host were shown as blanks** — the display now uses the current 290 table, which has lowercase letters (as other IBM emulators do). Input is unchanged: lowercase letters are still sent as uppercase.

### Changed
- **Connection presets are now one file per preset** — `presets\<name>.i5250` in the exe folder (one `key=value` per line, in the same spirit as the session configuration files of IBM i Access Client Solutions and other IBM emulators). Presets written in `config.json` by an earlier version are moved to the `presets` folder automatically at startup and removed from `config.json` (other settings in `config.json` are kept). The bundled template is `presets\MYSYS.i5250.example`.
- **Ctrl+F3 is now the character switch** — it toggles the display between half-width katakana and lowercase letters (same as A / ｱ on the toolbar and the same key as in IBM i Access Client Solutions). Previously Ctrl was ignored and F3 was sent.
- **Toolbar layout** — the buttons on the right are grouped by kind. First row = switches (language, column separator lines/dots, character set A/ｱ, window mode, input mode exclusive/shared, input field highlight, shift character display, blink display); second row = keyboard input target and others (mouse wheel settings, microphone, hide toolbars, help). The blink display toggle (light bulb icon) is a new button. Colors now follow the groups (switches = fixed blue-gray, with the three on/off icons dimmed while off; keyboard input target = teal for the 5250 screen / dark red for the toolbar; others = gray with light lines). The "on" color changed from olive to teal, and the buttons share one size (22 px high, 30 px wide). The column separator icon (three vertical lines / dashed line) now uses thin lines. The character set toggle (A / ｱ) uses a regular weight, the shift character display shows a vertical "0E 0F", and the microphone, input mode (one / two people) and mouse wheel settings icons are clearer line drawings.

### Added
- **Mnemonic selection in selection fields** (IBM "5494 Remote Control Unit Functions Reference" p.15-84) — in a menu bar, pull-down or other selection field, pressing the key of a choice's underlined character selects that choice (case-insensitive). In selection fields that run as soon as a choice is selected, the Enter key is sent as well.
- **URL hotspots** (as in IBM i Access Client Solutions) — URLs shown on the screen (http / https / www / mailto / ftp / file / gopher) are underlined, and a click opens them in a new window. URLs inside input fields are excluded. Browsers cannot open `file://` URLs (for example `FILE://SERVER/SHARE/A.JPG`) directly, so the MCP server reads the file from the PC through UNC (`\\SERVER\SHARE\A.JPG`) and returns it. This works only for URLs shown on the current screen.

## 2026-11-09 (v0.1.16-beta)

### Fixed
- **In the viewer, Field Exit in a signed numeric field did not pass the value to the host correctly** — e.g., typing "1" into a field of length 6 (5 digits + sign position) and pressing Field Exit moved the "1" into the sign position (6th position), so no digit reached the host. The value is now right-adjusted into the digit positions, excluding the sign position, as described for Field Exit in the IBM manual "5494 Remote Control Unit Functions Reference" (SC30-3533-04) (FIELD_EXIT of MCP `send_key` / `send_text` already behaved correctly).
- **Lowercase letters became blanks on a code page without lowercase (cp5026_legacy = old 290 Katakana)** — viewer typing, `send_text`, paste and AUTOTYP now all send them as uppercase. The viewer shows the uppercase letter from the moment you type it.
- **With the same session open in several tabs, typing in one tab did not move the cursor in the others** — the typed characters appeared, but the cursor stayed where it was. Tabs that did not type now show the host's cursor position.

### Added
- **AUTOTYP: added `^STEPON` / `^STEPONn` / `^STEPOFF`** — waits n × 0.5 seconds (n = 1–9; 0.5 seconds when omitted) after each following line (typing text, moving the cursor or a field key, and AID keys) so that a person can follow the screen. `^STEPOFF` stops it. The default is no wait.

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
