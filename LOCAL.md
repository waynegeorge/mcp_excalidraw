# Excalidraw MCP — Local Setup

Local-only notes for this install. Upstream README in `README.md` is generic; this file captures Frontier-specific configuration.

## What this is

Community MCP server (`yctimlin/mcp_excalidraw`) that exposes 26 tools for creating and editing Excalidraw diagrams from Claude Code. Backs onto a live canvas at `http://localhost:3333` — open the URL in a browser to watch edits happen in real time.

## Local config

- **Install path:** `B:\frontier\tools\excalidraw`
- **Canvas port:** `3333` (not 3000 — that's used by the Cooper's Bakehouse Next.js demo)
- **MCP scope:** `user` — registered in `C:\Users\Wayne\.claude.json`, available from any project
- **Sync:** `ENABLE_CANVAS_SYNC=true`, so MCP element ops push to the live canvas via WebSocket

## Day-to-day

### Start the canvas server

```bash
cd /b/frontier/tools/excalidraw
PORT=3333 ENABLE_CANVAS_SYNC=true node dist/server.js
```

Leave running. Open `http://localhost:3333/` in a browser to view/edit the live canvas.

### Stop it

Ctrl+C in the terminal, or kill the node PID listening on 3333:

```powershell
Get-NetTCPConnection -LocalPort 3333 -State Listen | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
```

### Use from Claude Code

The MCP server is auto-spawned by Claude Code each session. Just ask: "create a rectangle at 100,100", "add a flowchart with these nodes", "export the canvas to scene.excalidraw", etc.

Verify connection in any session:

```bash
claude mcp list
```

Should show `excalidraw: node B:/frontier/tools/excalidraw/dist/index.js - ✓ Connected`.

### Rebuild after upstream changes

```bash
cd /b/frontier/tools/excalidraw
git pull
npm ci && npm run build
```

## Importing a shared excalidraw.com link

Excalidraw's `#json=ID,KEY` share links are end-to-end encrypted — there's no public API to fetch them. Pull them in manually:

1. Open the share link in a browser, e.g.
   `https://excalidraw.com/#json=lgJHxlg-W7FZCiHyKIuT1,dTOmS7jK8TLthkhWKsCm_w`
2. Click the **hamburger menu** (top-left) → **Save to...** → pick a path under
   `B:\frontier\tools\excalidraw\scenes\` (create folder as needed)
3. The exported file is `.excalidraw` JSON — drop the path into Claude Code:
   "import `tools/excalidraw/scenes/whatever.excalidraw` into the canvas".

The MCP tool `import_scene` accepts either a `filePath` or raw scene `data`.

## Exporting back

- **To a new shareable link:** ask "export to excalidraw URL" — the MCP server uploads the current scene and returns a fresh `excalidraw.com/#json=...` URL.
- **To a `.excalidraw` file:** ask "export the scene to `path/to/file.excalidraw`".
- **To PNG / SVG:** ask "export to PNG at `path/to/file.png`" (canvas server must be running for image export).

## Where outputs live

Suggested layout under this folder:

```
tools/excalidraw/
├── scenes/        # imported / hand-saved .excalidraw files
└── exports/       # PNG / SVG renders
```

These two folders aren't in `.gitignore` upstream — decide per-scene whether they belong in the repo or are throwaway.
