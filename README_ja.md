# IBM 5250 MCP Server — 期間限定β配布版 / Time-Limited Beta Release

IBM i (AS/400) の 5250 端末エミュレーションを提供する MCP (Model Context Protocol) サーバーです。
EBCDIC DBCS (日本語) に対応し、Claude などの MCP クライアントから IBM i セッションを操作できます。

A Model Context Protocol (MCP) server providing IBM i (AS/400) 5250 terminal emulation,
with EBCDIC DBCS (Japanese) support, usable from MCP clients such as Claude.

---

## ⚠️ 重要 / Important

- **本配布物は期間限定β公開版です。2026 年 7 月 31 日まで利用できます。**
  2026 年 8 月 1 日以降は、起動および接続が自動的に拒否されます。継続利用には新しい版を入手してください。
- **This is a time-limited release. It works until 2026-07-31.**
  From 2026-08-01, startup and connection are automatically refused. Obtain a newer build to continue.

- **対応プラットフォーム: Windows (x64) のみ。**
- **macOS 版は提供していません。** ビルドツール bun のクロスコンパイル不具合
  ([oven-sh/bun#25346](https://github.com/oven-sh/bun/issues/25346)) により、
  Windows 環境から動作する macOS 実行ファイルを生成できないためです。
- **Supported platform: Windows (x64) only.** **A macOS build is NOT provided**, due to a
  cross-compilation bug in the bun build tool ([oven-sh/bun#25346](https://github.com/oven-sh/bun/issues/25346))
  that prevents producing a working macOS executable from Windows.

- **検証環境: 本製品は日本語環境 (一次言語 2962、QCCSID 65535) でのみ検証されています。** 他の言語・CCSID 環境での動作は保証されません。
- **Verified environment: this product has been verified only in a Japanese environment (primary language 2962, QCCSID 65535).** Operation in other language / CCSID environments is not guaranteed.

---

## ⚠️ 免責事項 / Disclaimer

- **本製品は試験的な 5250 エミュレーターの実装であり、本番業務で利用されている既存の 5250 エミュレーターを代替する目的で作成されたものではありません。**
- **データ転送、コンソール接続、右横書き (RTL) 言語を含む多国語対応などは実装されていません。**
- **5250 画面を直接操作する能力を持つため、AI が破壊的な操作を行う可能性はゼロではありません。**
- **十分なテストは行われておらず、サポートも提供されません。利用する場合は自己責任でお願いします。**
- **個人的なプロジェクトとして開発されており、特定の組織や団体の意図を反映したものではありません。**
- This is an **experimental** 5250 emulator implementation; it is **not** intended to replace existing 5250 emulators used in production.
- Features such as data transfer, console connection, and multilingual support (including right-to-left / RTL languages) are **not** implemented.
- Because it can operate 5250 screens directly, **the possibility that an AI performs destructive operations is not zero**.
- It has **not been sufficiently tested, and no support is provided. Use it at your own risk.**
- It is developed as a **personal project** and does not reflect the intent of any specific organization or entity.

---

## 概要 / Overview

- TN5250E プロトコル / EBCDIC DBCS (cp1399, cp5026, cp037 など) / 24x80 (DS3)・27x132 (DS4)
- 画面ビューア (Web): 既定で `http://localhost:5250`
- 単一実行ファイル形式 (ランタイム同梱)。Node.js のインストールは不要です。
- Single self-contained executable (runtime bundled). No Node.js installation required.

## ダウンロード / Download

本リポジトリの [**Releases**](https://github.com/GuriCat/ibm5250-mcp/releases) の **Assets** から `ibm5250-mcp-windows-x64.zip` を入手し、**必ず展開してから**使ってください。zip には exe・`config.example.json`・README が入っています。
Download `ibm5250-mcp-windows-x64.zip` from **Assets** on the [**Releases**](https://github.com/GuriCat/ibm5250-mcp/releases) page and **extract it before use**. The zip contains the exe, `config.example.json`, and README.

**よくある間違い / Common mistakes**:

- ダウンロードするのは Releases ページの **Assets 欄にある `ibm5250-mcp-windows-x64.zip`** です。その下の「**Source code (zip / tar.gz)**」は別物で、プログラム本体は**入っていません** (ドキュメントのみ)。
  Download **`ibm5250-mcp-windows-x64.zip` under "Assets"**. The separate "Source code (zip / tar.gz)" links below do **NOT** contain the program (documents only).
- zip は **必ず右クリック →「すべて展開」で展開してから**中のファイルを使ってください。エクスプローラーで zip を開いただけの窓から exe を実行・コピーすると正しく動作しません。
  **Always extract** (right-click → "Extract All") before using the files. Running or copying the exe from inside the zip preview window will not work.

## 更新 (アップデート) / Updating

新しいバージョンへの入れ替え手順です。**使用中の exe は上書きできない**点に注意してください。
Steps to update to a new version. Note that the exe **cannot be overwritten while it is in use**.

1. VS Code / IBM Bob 等、この MCP を使う IDE を**すべて終了**します (= exe のロック解除)。
   **Close all IDEs** (VS Code, IBM Bob, etc.) that use this MCP server (releases the file lock).
2. Releases から新しい `ibm5250-mcp-windows-x64.zip` をダウンロードして展開します。
   Download the new `ibm5250-mcp-windows-x64.zip` from Releases and extract it.
3. 展開した中の `ibm5250-mcp-windows-x64.exe` を、既存の exe と**同じフォルダに、同じファイル名で上書き**します。心配な場合は先に旧 exe を `.bak` にリネームして退避してください。
   Copy the extracted `ibm5250-mcp-windows-x64.exe` over your existing exe (**same folder, same file name**). If unsure, rename the old exe to `.bak` first.
4. **既存の `config.json` はそのまま残します** — 削除・上書きしないでください。設定は新バージョンにそのまま引き継がれます (zip 内は `config.example.json` なので既存 `config.json` を上書きしません)。
   **Keep your existing `config.json`** — do not delete or overwrite it. Your settings carry over (the zip ships `config.example.json`, so it never clobbers your `config.json`).
5. IDE を起動し直します。 / Restart your IDE.

トラブルシューティング / Troubleshooting:

- 「上書きできません」「使用中です」と出る場合: IDE がまだ完全に終了していません。タスクマネージャーで `ibm5250-mcp-windows-x64.exe` を終了してからやり直してください。
  "Cannot overwrite" / "file in use": the IDE has not fully exited. End `ibm5250-mcp-windows-x64.exe` in Task Manager and retry.
- exe をコピーせず zip のまま置いても**更新されません**。必ず展開して元の exe を置き換えてください。
  Leaving the file as a zip does **not** update anything. Extract it and replace the original exe.
- 更新できたかの確認: 画面ビューア上部の ℹ️ (About) にバージョンとビルド日時が表示されます。
  To verify the update: the ℹ️ (About) button at the top of the screen viewer shows the version and build date.

## クイックスタート / Quick Start

1. 展開した `ibm5250-mcp-windows-x64.exe` を任意のフォルダに配置します。
2. **同じフォルダに `config.json` を置きます** (接続先 IBM i のプリセットを記述)。同梱の
   `config.example.json` を `config.json` にリネーム (またはコピー) して編集するのが簡単です。
   サーバーは exe と同じフォルダから `config.json`・プリセット・ログを自動的に読み書きします
   (起動時の作業ディレクトリに関わらず、exe 自身のフォルダを基準にするため、MCP クライアント側で
   作業ディレクトリを指定する必要はありません)。
3. MCP クライアント (例: Claude Desktop) の設定に登録します:

```json
{
  "mcpServers": {
    "ibm5250": {
      "command": "C:\\path\\to\\ibm5250-mcp-windows-x64.exe"
    }
  }
}
```

`config.json` の記述例 / Example `config.json`:

```json
{
  "presets": {
    "MYSYS": {
      "host": "ibm-i.example.com",
      "port": 23,
      "code_page": "cp1399",
      "screen_size": "27x132",
      "allow_ds4": true
    }
  }
}
```

4. クライアントから `connect_5250` ツールを `{"preset": "MYSYS"}` で呼び出すと接続します。
   画面ビューアは `http://localhost:5250` で開けます。

詳細は同梱マニュアルを参照してください。/ See the bundled manuals for details.

## マニュアル / Manuals

`docs/manuals/` フォルダに収録 / Located under `docs/manuals/`:

- 日本語: `IBM5250-MCP-User-Guide.pdf` (ユーザー・ガイド) / `IBM5250-MCP-Admin-Guide.pdf` (管理ガイド)
- English: `IBM5250-MCP-User-Guide-EN.pdf` / `IBM5250-MCP-Admin-Guide-EN.pdf`

## 変更履歴 / Changelog

本リリースの利用者向け変更点は [`CHANGELOG_ja.md`](CHANGELOG_ja.md) を参照してください。
(For user-facing changes, see [`CHANGELOG_ja.md`](CHANGELOG_ja.md) / English: [`CHANGELOG.md`](CHANGELOG.md).)

## AI エージェント向け / For AI agents

本サーバーを AI エージェント (Claude 等の LLM) から操作する場合は [`AI-GUIDE.md`](AI-GUIDE.md) を参照してください。
PDF マニュアルは人間向け (画面写真・インストール・構成) ですが、`AI-GUIDE.md` は **エージェント自身のための密な手引き**です。
各ツールの完全な schema は MCP 経由で提供され実行ファイルに埋め込まれているため **ソースコードは不要**であること、
押さえるべき 5250 概念 (OIA・`X II` キーロック・DS3/DS4・フィールドの属性バイトのずれ・AID/編集キー・DBCS)、
接続 → 画面取得 (`get_screen` json) → 入力 (`send_text`) の基本フロー、サインオン例、操作パターン、よくある落とし穴を
構造化して記載しています。これにより、エージェントはソースを見ずに IBM i を確実に操作できます。

For AI agents (LLMs) driving this server, see [`AI-GUIDE.md`](AI-GUIDE.md) — a dense, agent-oriented guide
covering how the full tool schemas are exposed over MCP and embedded in the exe (**no source needed**),
the 5250 concepts, the connect → read → type flow, sign-on, operation patterns, and pitfalls.

## ライセンス / License

本ソフトウェアの使用には [`EULA.md`](EULA.md) (ソフトウェア使用許諾契約) への同意が必要です。
**ソースコードは非公開です。** 逆コンパイル・逆アセンブル・改変・再配布は禁止されています。
Use of this software is subject to the End-User License Agreement ([`EULA.md`](EULA.md)).
**The source code is not published.** Reverse engineering, decompilation, modification, and
redistribution are prohibited.

第三者ソフトウェアの帰属表示は [`THIRD-PARTY-NOTICES.md`](THIRD-PARTY-NOTICES.md) を参照してください。
Third-party attributions: see [`THIRD-PARTY-NOTICES.md`](THIRD-PARTY-NOTICES.md).
