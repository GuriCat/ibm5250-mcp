# IBM 5250 MCP Server — Guide for AI Agents

This document is written for the **AI agent (LLM)** that drives this MCP server. The bundled PDF
manuals are for humans; this file gives the dense, structured information an agent needs to operate
IBM i 5250 sessions reliably. Read this once before your first `connect_5250` call.

> 本書は本 MCP サーバーを操作する **AI エージェント (LLM) 向け**の密な手引きです。PDF マニュアルは
> 人間向けで、エージェントの操作判断には情報が冗長かつ不足するため、本書を併用してください。

---

## 0. The tools are self-describing — you do NOT need the source code

This product is distributed as a binary **without source code**, but you never need the source to
operate it. The MCP protocol exposes the **full schema and description of every tool** — parameters,
enums, defaults, return shapes, and caveats — embedded in the executable.

**Before operating, call the MCP "list tools" operation and read each tool's `inputSchema` and
description.** They are detailed (e.g. `send_text` explains that `TAB` moves the cursor without sending
an AID and that `row`/`col` is the data position = `field.col + 1`; `get_screen` documents the full
json shape; `send_key` lists every AID and edit key). This guide complements those per-tool schemas
with the cross-cutting concepts, flows, and pitfalls that individual schemas do not convey.

> 開発時に AI がソースを参照していたのは実装調査のためで、通常運用には不要です。配布 exe には
> 各ツールの完全な schema/description が埋め込まれており、MCP の list-tools で取得できます
> (ソース非公開でも操作に必要な情報は揃います)。本書はそれを概念・手順・注意点で補完します。

## 1. What this server does

Connects to **IBM i (AS/400)** over **TN5250E (Telnet, port 23)**, maintains the live screen buffer,
and exposes it through MCP tools. Supports EBCDIC **SBCS/DBCS (Japanese)**, screen sizes **24x80 (DS3)**
and **27x132 (DS4)**, AID keys, edit keys, AUTOTYP scripts, and HAScript macros. A web viewer is served
at `http://localhost:5250/` for humans to watch/operate the same session.

## 2. Tools

| Tool | Purpose | Key arguments |
|---|---|---|
| `connect_5250` | Open a session | `preset` (recommended) or `host`/`port`/`code_page`/`screen_size`; `session_id` (optional, auto-generated). Returns `session_id` used by all later calls. |
| `disconnect_5250` | Close a session | `session_id` |
| `get_screen` | Read the current screen | `session_id`, `format` = `text` \| `json`. **Use `json`** for `fields`, `cursor`, `inputReady`, OIA flags. |
| `get_field_text` | Read text at a position | `session_id`, `row`, `col`, `length` (1-based) |
| `send_text` | Type into a field + send a key | `session_id`, `row`, `col` (1-based data position), `text`, `key` (default `ENTER`; `TAB` = move only, no AID) |
| `send_key` | Send an AID / nav / edit key | `session_id`, `key` (e.g. `ENTER`, `F1`-`F24`, `CLEAR`, `PRINT`, `RESET`, `PAGEDOWN`, `PA1`-`PA3`, `TAB`, `ERASE_EOF`, `FIELD_EXIT`, `DUP`, `DEL`, `INSERT`) |
| `paste_text` | Multi-field / multi-line paste | `session_id`, text with `\t` (TAB) / `\n` (field break) |
| `run_autotyp` | Run a deterministic `.autotyp` script | `session_id`, script name (in `scripts/`) |
| `run_macro` | Run an HCL ZIEWeb/HOD HAScript XML macro | `session_id`, macro |
| `convert_macro` | Convert a macro format | macro input |
| `list_sessions` | List active sessions | — |
| `list_connection_presets` | List presets from `config.json` | — |
| `list_autotyp_scripts` | List available `.autotyp` scripts | — |
| `save_connection_preset` | Persist a connection preset | preset fields |
| `get_session_log` | Retrieve session log entries | `session_id` |

## 3. 5250 concepts you must understand

- **OIA (Operator Information Area)** — the bottom status line. `MA*` = connected; `X II` = keyboard
  locked (server busy, e.g. after PRINT); `X SYSTEM` = waiting for the host; `COMM659` = disconnected;
  `nn/nnn` = cursor row/col.
- **`inputReady`** (in `get_screen` json) — `false` means the keyboard is locked (`X II`/`X SYSTEM`).
  **Do not send input while `false`**; wait, or send `RESET` to clear `X II`.
- **DS3 (24x80) / DS4 (27x132)** — the host switches size on demand (e.g. WRKACTJOB stays DS3, some
  panels go DS4). `get_screen` json reports actual `rows`/`cols`.
- **Fields** — `get_screen` json `fields[]` gives each input field. **`row`/`col` is the *attribute
  byte* position; the data starts at `col + 1`.** So when calling `send_text`, pass `row = field.row`
  and `col = field.col + 1`. Fields also carry DDS metadata (`protected`, `numeric`, `monocase`,
  `mandatoryEnter`, `dbcsType`, `continued`, etc.).
- **AID keys** — only AID keys (ENTER, F1-F24, CLEAR, PA1-3, PAGEUP/DOWN, etc.) are sent to the host.
  Navigation/edit keys (TAB, arrows, ERASE_EOF, FIELD_EXIT, DEL, DUP, INSERT) act on the client-side
  pending buffer before an AID is sent.
- **DBCS / SO-SI** — Japanese double-byte text is bracketed by Shift-Out/Shift-In. In `text` output,
  SO/SI are shown as `{` / `}` to avoid column drift. Pick the right `code_page` at connect
  (`cp1399` JIS2004 DBCS is the common Japanese default).

## 4. Typical operation flow

1. `connect_5250 { "preset": "<name>" }` → note the returned `session_id`.
2. Wait briefly, then `get_screen { session_id, format: "json" }`. Check `inputReady`, read `fields`
   and `cursor`, and read `text` to understand the panel.
3. To type: `send_text { session_id, row, col, text, key }` where `row`/`col` is the **data** position
   (`field.col + 1`). Use `key: "TAB"` to fill one field and move without submitting; use
   `key: "ENTER"` (or `F3` etc.) to submit.
4. `get_screen` again to confirm the result (check the new `text` and `inputReady`).

## 5. Sign-on (example)

A sign-on panel typically has user at row 6 and password at row 7. Fill both, then ENTER:

```
send_text { session_id, row: 6, col: 53, text: "<user>", key: "TAB" }   // type user, move only
send_text { session_id, row: 7, col: 53, text: "<password>", key: "ENTER" }  // type password + submit
```

If a "display program messages" panel appears, send `send_key { key: "ENTER" }` to continue.
(Exact row/col vary by panel — confirm with `get_screen` json `fields` first.)

## 6. Common patterns

- **Run a CL command**: on a command line, `send_text { text: "WRKACTJOB", key: "ENTER" }`
  (row/col default to the cursor when omitted).
- **Function keys**: `send_key { key: "F3" }` (exit), `F4` (prompt), `F12` (cancel).
- **Paging**: `send_key { key: "PAGEDOWN" }` / `"PAGEUP"`.
- **PRINT**: `send_key { key: "PRINT" }` locks the keyboard (`X II`); clear with `send_key { key: "RESET" }`.
- **Multi-field paste**: `paste_text` with `\t` between fields and `\n` for the next input row.
- **Deterministic flows**: prefer `run_autotyp` for repeatable sequences (e.g. sign-on).

## 7. Pitfalls (read before operating)

- **Never send input while `inputReady` is `false`** (`X II`/`X SYSTEM`). Wait or `RESET`.
- **`field.row`/`field.col` is the attribute byte**; the data position is `col + 1`. Off-by-one here is
  the most common mistake.
- **`TAB` does not submit** — it only moves the cursor client-side. Use `ENTER`/`Fnn` to send an AID.
- **Confirm before assuming** — after any AID, call `get_screen` and verify the new panel; do not assume
  success from the absence of an error.
- **Sign off before disconnecting** an interactive session: return to a command line and run `SIGNOFF`
  (or the menu sign-off) **before** `disconnect_5250`. A bare TCP disconnect can orphan an interactive
  job (`QPADEVxxxx`) on the host. (If the session is hung, `send_key SYSRQ` then `2` to cancel, then sign off.)
- **DBCS panels**: choose the correct `code_page` at connect; otherwise Japanese text is garbled.
- **Keep Playwright to observation; route 5250 keystrokes through MCP or the user's keyboard.**
  Playwright's keystroke functions (`browser_type` / `browser_press_key` / `browser_fill_form`) are
  **not forbidden as such**, but mixing them with a concurrent MCP AID **simultaneously and at high
  volume / high speed** can drop or reorder input — the MCP AID runs server-side and bypasses the
  viewer's client-side input serialization. Use Playwright for **observation**
  (`browser_navigate` / `browser_snapshot` / `browser_take_screenshot` / `browser_evaluate`), and enter
  5250 keystrokes / AID via the user's keyboard or this server's `send_key` / `send_text` /
  `run_autotyp` / `paste_text`. Also do **not** call `browser_resize` (it breaks viewport following).

> Playwright MCP を併用する場合、Playwright は**観測のみ** (`browser_navigate` / `browser_snapshot` /
> `browser_take_screenshot` / `browser_evaluate`) に使い、5250 へのキー入力・AID 送信は、ユーザーによる
> キーボード入力、あるいは本サーバーの `send_key` / `send_text` / `run_autotyp` / `paste_text` で行なう事を
> 推奨します。Playwright のキー入力 (`browser_type` / `browser_press_key` / `browser_fill_form`) の使用自体は
> 禁止ではありませんが、MCP の AID と**同時に、かつ大量・高速に**混在させると入力の取りこぼし・順序入替が
> 起きえます (= MCP の AID はサーバー側で動き viewer の直列化を経由しないため)。`browser_resize` も呼ばないこと。

### Division of roles — MCP tools / Playwright / user's normal browser

◯ = available, △ = only indirectly / partially, × = not available; conflict caveats are noted in the cell.

| Purpose / function | MCP tools | Playwright (dedicated Chromium) | User's normal browser (manual) |
|---|---|---|---|
| **Primary use** | input + logical-screen retrieval + automation | visual / DOM observation only | interactive use + final visual check |
| **Keystroke / AID input** | ◯ at the protocol layer (`send_key` / `send_text`) | ◯ via `browser_type` etc. (note: conflicts with the MCP AID when concurrent / high-volume / fast — route input through MCP) | ◯ user types on the keyboard |
| **Multi-line paste** | ◯ `paste_text` | ◯ text entered via `browser_type` (note: same conflict when concurrent / fast) | ◯ Ctrl+V |
| **Deterministic scripts (AUTOTYP / macro)** | ◯ `run_autotyp` / `run_macro` | × no means to run | × manual operation only |
| **Mouse / text selection** | × no mouse (keys / AID only) | ◯ `browser_click` / `browser_drag` | ◯ with the mouse |
| **Logical screen (text / attributes)** | ◯ `get_screen` | △ indirectly via the DOM | △ by eye / DevTools |
| **Screenshot (rendered image) / appearance** | △ text form via `get_screen`; no rendered pixel image | ◯ `browser_take_screenshot` | ◯ by eye / OS screenshot |
| **DOM / CSS / layout inspection** | × no live DOM access | ◯ `browser_evaluate` | ◯ DevTools (F12) |
| **Browser console / network** (DevTools: JS console / traffic log) | △ server-inbound requests only (`get_session_log`); browser console/network not visible | ◯ `browser_console_messages` / network | ◯ DevTools (F12) Console / Network tabs |
| **Connection management** | ◯ `connect_5250` / `disconnect_5250` | △ no API, but can click the connect/disconnect button via `browser_click` | ◯ viewer connect form / preset + disconnect toggle |

> 役割分担: 入力 (キー / AID / 貼り付け) と論理画面取得・自動化は **MCP ツール**、描画 / DOM / console 等の
> 観測は **Playwright** (= AI が起動する専用 Chromium、観測専用)、日常の対話操作と最終的な見た目確認は
> **ユーザーの通常ブラウザー** (= Playwright からは制御・観測できない別インスタンス) が担当します。

## 8. Voice input (optional)

The viewer has a microphone (🎙) button for voice input. It works **only** when the viewer is used
with the **Claude extension (Claude Code) in Visual Studio Code**, and requires a **hook** on the
Claude side. Flow: 🎙 button → MCP server `POST /api/voice` → `tmp/pending-voice.txt` → the hook
(`scripts/voice-hook.cjs`, registered on `SessionStart` and `Stop`) reads the file and injects the
transcript as user input. As the AI agent you receive the transcript as ordinary user input via the
hook's `additionalContext`; no special tool call is needed. After the transcript is delivered the mic
indicator returns to idle automatically. See the Administrator Guide (Section 1.6) for the exact hook
configuration.

> 音声入力 (🎙) は VSCode + Claude 拡張機能との併用時のみ動作し、Claude 側に hook の追加が必要です。
> 🎙 → MCP `/api/voice` → `tmp/pending-voice.txt` → hook (`voice-hook.cjs`) が transcript を入力として注入します。
> AI エージェントは hook の additionalContext として transcript を受け取ります (専用ツール呼出は不要)。設定は管理ガイド 1.6 節参照。

## 9. Showing the live 5250 screen to a human (viewer visibility)

Each connection opens its **own** viewer window on its **own** port — like ACS "run same configuration" → a new session window. The web viewer renders the live screen so a human can watch/operate the session you drive.

> ⚠️ **Common mistakes (read first):**
> - **Never hardcode the viewer port or session_id.** Both are assigned at connect time. Read `viewer_url` and `session_id` from the `connect_5250` result.
> - **`session_id` cannot be specified** — it is auto-generated as `{host}-{telnetPort}-mcp{viewerPort}` (e.g. `192.168.1.191-23-mcp5251`). Use the returned value for every subsequent tool call.
> - **Each connect uses a new port/window.** Two connects to the same preset → two windows (ports 5251, 5252, …). Ports are freed on disconnect/close and reused from 5250 up (no exhaustion).
> - **The `auto_open` browser is a separate OS process you (the agent) cannot control.** To drive the browser yourself, use the Playwright MCP (option 3), not `auto_open`.
> - **`auto_open=browser` gives an independent window only if the default browser is Chromium (Chrome/Edge).** Other defaults (Firefox etc.) open a new *tab*. `browser-app` = chromeless window.
> - Never send 5250 keystrokes via Playwright's keyboard — use `send_key`/`send_text`/`run_autotyp` (Section 7).

### Closing a window ⇄ disconnecting (bidirectional)
- **You disconnect** (`disconnect_5250`) → that session's viewer window is closed automatically and its port is freed.
- **The human closes the browser window** → after a ~5 s grace period (a reload/transient drop is *not* misread as a close) the session is **auto-disconnected** and the port freed.
- So "close the window" and "disconnect" are two sides of the same action — an ACS-like feel.

### 1. Read `viewer_url` from the tool result
`connect_5250` and `list_sessions` return `viewer_url` (e.g. `http://localhost:5251/?session=192.168.1.191-23-mcp5251`) carrying that session's **actual** viewer port. Tell the user to open it, or open it yourself with the Playwright MCP (option 3).

### 2. Auto-open a browser (no agent action, works with the distributed exe)
Set the env var in the **`.mcp.json` that registers this server** (you do not touch this server's own `config.json`):

```jsonc
{
  "mcpServers": {
    "ibm5250": {
      "command": "C:\\path\\to\\ibm5250-mcp-windows-x64.exe",
      "env": {
        "SCREEN_VIEWER_AUTO_OPEN": "browser",          // off (default) | browser | browser-app
        "IBM5250_AUTOCONNECT_PRESET": "GURICAT"        // auto-connect at startup (optional)
      }
    }
  }
}
```

With `browser`/`browser-app`, **every `connect_5250` auto-opens that session's window**. `off` opens nothing and just returns `viewer_url`. Both launch the **OS browser directly — no Playwright needed**, so they work in the distributed exe and for any agent. (`env` overrides `config.json`'s `viewer.auto_open`; the built-in default is `browser-app`.)

**Startup behavior (2026-07-03):** when this server is started by an MCP client (you), **nothing opens at startup** — windows open per connect. The launcher (connection form) opens at startup only when the product is launched directly by a human (exe double-click / terminal = stdin is a TTY). If you need the connection form or need to reopen a session window the human closed by mistake, call the MCP tool **`open_viewer`** (no args = launcher; `session_id` = reopen that session's window; `mode` = browser | browser-app).

**ℹ️ Help (2026-07-03):** the ℹ️ button on the viewer's second toolbar row (or `http://localhost:<port>/help`) opens version + build date, EULA, third-party notices, and the User/Admin Guides (HTML with contents + search) in a normal browser window, following the viewer UI language.

**`auto_open` behavior matrix** — setting × default browser × how it launches × window shape:

| `auto_open` | default browser | launched / how | window shape |
|---|---|---|---|
| `off` | — | nothing | — |
| `browser` | Chrome | `chrome.exe --new-window` | independent normal window (tab / address bar / menu **shown**) |
| `browser` | Edge | `msedge.exe --new-window` | same |
| `browser` | other (Firefox etc.) / exe not found | default browser via OS handler | new **tab** (not a separate window) |
| `browser-app` (default) | Chrome | `chrome.exe --app=` | independent **chromeless** window (no tab / address bar / menu) |
| `browser-app` | Edge | `msedge.exe --app=` | same |
| `browser-app` | other | Chrome→Edge (whichever is installed) `--app=` | chromeless window |
| `browser-app` | other + neither installed | default browser via OS handler | new tab (app mode N/A) |

### 3. Open it with the Playwright MCP, headed (when your environment has Playwright)
Keep `SCREEN_VIEWER_AUTO_OPEN=off`, then point the agent-controlled browser at the URL from option 1:

```
browser_navigate("<viewer_url from connect_5250>")     // e.g. http://localhost:5251/?session=<id>
```

This is the **only** way the *agent-controlled* browser shows the screen — the `auto_open` browser is a separate OS process the agent cannot drive. Note: **Playwright is not bundled with Claude Code**; the user installs it once (`claude mcp add playwright -- npx -y @playwright/mcp@latest`, or the official plugin marketplace). For a chromeless window, configure `--app=...` in `playwright-mcp-config.json` (see the Administrator Guide). Always route 5250 keystrokes through `send_key` / `send_text` / `run_autotyp`, never Playwright's keyboard (Section 7).

> 各接続は専用 port・専用ウィンドウを開く (= ACS「同じ構成で実行」= 新セッション新窓)。**session_id は指定不可・自動生成**
> (`{host}-{telnetPort}-mcp{viewerPort}`、例 `192.168.1.191-23-mcp5251`)、戻り値の `session_id`/`viewer_url` を後続で使う
> (port もハードコードせず戻り値を読む)。**同じ preset を2回接続 → 別 port・別窓** (切断/窓クローズで port を 5250 起点で解放・再利用、枯渇なし)。
> **窓を閉じる ⇄ disconnect は双方向**: disconnect すると窓が閉じ port 解放、ブラウザ窓を閉じると約 5 秒の grace 後に自動 disconnect
> (リロード/一時断は誤検知しない)。`auto_open` は **connect ごとに窓を自動オープン** (`off`=開かない・viewer_url を返すのみ、
> `browser`=既定ブラウザ (既定が Chromium 系のみ独立窓・他はタブ)、`browser-app`=Chromium app モードのクロームレス窓)。
> OS ブラウザ直接起動で Playwright 不要・配布 exe で動作。エージェント制御で見せるには option 3 (Playwright、`auto_open=off` + `browser_navigate`)。
> **起動時は何も開かない** (= MCP クライアント起動時。窓が開くのは connect のとき。launcher が起動時に開くのは人間の直接起動 = stdin が端末 のみ)。
> 接続フォームや誤って閉じた session 窓を開くには MCP tool **`open_viewer`** (引数なし = launcher / `session_id` = その session の窓の開き直し)。
> **ℹ️ ヘルプ**: viewer ツールバー 2 段目の ℹ️ (または `/help`) でバージョン + ビルド日時 / EULA / サードパーティ表記 / User & Admin Guide (HTML、目次 + 検索) を通常ブラウザ窓に表示 (UI 言語追従)。
> 5250 キー入力は必ず `send_key`/`send_text` 経由 (Playwright キー入力は使わない、7 節)。

---

This file is intended to be read by the AI agent itself. For human-oriented documentation (screenshots,
installation, configuration), see the PDF manuals under `docs/manuals/`.
