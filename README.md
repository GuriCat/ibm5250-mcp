# IBM 5250 MCP Server — Time-Limited Beta Release

A Model Context Protocol (MCP) server that provides IBM i (AS/400) 5250 terminal emulation,
with EBCDIC DBCS (Japanese) support, usable from MCP clients such as Claude.

> 日本語版は [`README_ja.md`](README_ja.md) を参照してください。 (Japanese version: see [`README_ja.md`](README_ja.md).)

---

## ⚠️ Important

- **This is a time-limited release. It works until 2026-07-31.**
  From 2026-08-01, startup and connection are automatically refused. Obtain a newer build to continue.
- **Supported platform: Windows (x64) only.**
- **A macOS build is NOT provided**, due to a cross-compilation bug in the bun build tool
  ([oven-sh/bun#25346](https://github.com/oven-sh/bun/issues/25346)) that prevents producing a
  working macOS executable from Windows.
- **Verified environment: this product has been verified only in a Japanese environment
  (primary language 2962, QCCSID 65535).** Operation in other language / CCSID environments is not guaranteed.

---

## ⚠️ Disclaimer

- This is an **experimental 5250 emulator implementation**; it is **not** intended to replace existing 5250 emulators used in production.
- Features such as data transfer, console connection, and multilingual support (including right-to-left / RTL languages) are **not** implemented.
- Because it can operate 5250 screens directly, **the possibility that an AI performs destructive operations is not zero**.
- It has **not been sufficiently tested, and no support is provided. Use it at your own risk.**
- It is developed as a **personal project** and does not reflect the intent of any specific organization or entity.

---

## Overview

- TN5250E protocol / EBCDIC DBCS (cp1399, cp5026, cp037, etc.) / 24x80 (DS3) and 27x132 (DS4)
- Web-based screen viewer (default `http://localhost:5250`)
- A single self-contained executable (runtime bundled). No Node.js installation required.

## Download

Download `ibm5250-mcp-windows-x64.exe` from the [**Releases**](https://github.com/GuriCat/ibm5250-mcp/releases) page of this repository.

## Quick Start

1. Place `ibm5250-mcp-windows-x64.exe` in any folder.
2. **Put `config.json` in the same folder** (with your IBM i connection presets). The server
   automatically reads `config.json`, presets, and logs from the **folder where the exe resides**
   (regardless of the launch working directory — so you do not need to set a working directory in
   your MCP client).
3. Register it in your MCP client (e.g. Claude Desktop):

```json
{
  "mcpServers": {
    "ibm5250": {
      "command": "C:\\path\\to\\ibm5250-mcp-windows-x64.exe"
    }
  }
}
```

Example `config.json`:

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

4. Calling the `connect_5250` tool with `{"preset": "MYSYS"}` connects to the host.
   Open the screen viewer at `http://localhost:5250`.

See the bundled manuals for details.

## Manuals

Located under `docs/manuals/`:

- English: `IBM5250-MCP-User-Guide-EN.pdf`, `IBM5250-MCP-Admin-Guide-EN.pdf`
- Japanese: `IBM5250-MCP-User-Guide.pdf`, `IBM5250-MCP-Admin-Guide.pdf`

## Changelog

See [`CHANGELOG.md`](CHANGELOG.md) for the user-facing changes in this release.

## For AI agents

If you drive this server from an AI agent (LLM such as Claude), read [`AI-GUIDE.md`](AI-GUIDE.md).

The PDF manuals are written for humans (screenshots, installation, configuration). `AI-GUIDE.md` is a
**dense, structured guide for the agent itself**: it explains that the full per-tool schemas are
exposed over MCP and embedded in the executable (**so the source code is not needed**), plus the 5250
concepts the agent must know (OIA, `X II` keyboard lock, DS3/DS4, fields and the attribute-byte
off-by-one, AID vs edit keys, DBCS/SO-SI), the typical connect → read (`get_screen` json) → type
(`send_text`) flow, a sign-on example, common operation patterns, and the frequent pitfalls. Reading
it lets an agent operate IBM i reliably without inspecting any source.

## License

Use of this software is subject to the End-User License Agreement ([`EULA.md`](EULA.md)).
**The source code is not published.** Reverse engineering, decompilation, modification, and
redistribution are prohibited.

Third-party attributions: see [`THIRD-PARTY-NOTICES.md`](THIRD-PARTY-NOTICES.md).
