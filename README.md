# Roblox Studio MCP server

An MCP (Model Context Protocol) server plus a Roblox Studio plugin that lets an AI client such as Claude Code read and edit a live Studio session: instances, properties, scripts, GUIs, builds, playtests and more.

This is a fork of [Chrrxs/robloxstudio-mcp](https://github.com/Chrrxs/robloxstudio-mcp) (MIT). All credit for the original design and implementation goes to the upstream author; see `LICENSE`.

## Prerequisites

- Node.js 18 or newer (22 tested) and npm
- Roblox Studio (Windows or macOS)
- An MCP client, e.g. [Claude Code](https://claude.com/claude-code)

## 1. Build the server

```
npm install
npm run build
```

This produces `packages/robloxstudio-mcp/dist/index.js` (full read/write server) and `packages/robloxstudio-mcp-inspector/dist/index.js` (read-only variant).

## 2. Build and install the Studio plugin

```
cd studio-plugin
npm install
npm run build          # compiles TypeScript to Luau with roblox-ts
cd ..
node scripts/build-plugin.mjs
```

`scripts/build-plugin.mjs` writes `studio-plugin/MCPPlugin.rbxmx` and also tries to copy it into your Studio Plugins folder automatically (set `MCP_PLUGINS_DIR` to override the destination). If the copy did not happen, copy the file by hand:

- Windows: `%LOCALAPPDATA%\Roblox\Plugins\`
- macOS: `~/Documents/Roblox/Plugins/`

(Shortcut for the two build steps: `npm run build:plugin` from the repo root.)

## 3. Configure Studio

1. Open Studio, then **Game Settings > Security** and turn on **Allow HTTP Requests** for your place.
2. Fully close and reopen Studio so the plugin loads.
3. Click the **MCP Server** button in the Plugins tab. It should turn green once the server is running and connected (the server listens on port 58741 by default; override with `ROBLOX_STUDIO_PORT`).

## 4. Register with Claude Code

```
claude mcp add robloxstudio -- node /absolute/path/to/this/repo/packages/robloxstudio-mcp/dist/index.js
```

On Windows use a path like `C:\path\to\repo\packages\robloxstudio-mcp\dist\index.js`. Restart Claude Code, keep Studio open, and ask it to do something with your place.

Optional environment variables: `ROBLOX_OPEN_CLOUD_API_KEY`, `ROBLOX_CREATOR_USER_ID` / `ROBLOX_CREATOR_GROUP_ID` (asset upload and place-version tools), `ROBLOX_STUDIO_PORT`, `MCP_PLUGINS_DIR`.

## Tools overview

90 tools (39 read, 51 write), defined in `packages/core/src/tools/definitions.ts`:

| Area | Count | Highlights |
| --- | --- | --- |
| Browsing and inspection | ~12 | `get_file_tree`, `search_objects`, `get_instance_properties`, `get_descendants`, `compare_instances` |
| Properties, objects, attributes, tags | ~25 | `set_property`, `mass_set_property`, `create_object`, `mass_create_objects`, `smart_duplicate`, `add_tag` |
| Scripts | ~9 | `get_script_source`, `edit_script_lines`, `grep_scripts`, `find_and_replace_in_scripts` |
| Luau execution | 3 | `execute_luau`, `eval_server_runtime`, `eval_client_runtime` |
| GUI | 3 | `create_gui_element`, `create_remote_event`, `wire_gui_to_server` |
| Playtest and diagnostics | 12 | `solo_playtest`, `multiplayer_playtest`, `get_runtime_logs`, `capture_script_profiler`, `breakpoints`, device simulator |
| Build library | 8 | `export_build`, `generate_build`, `import_build`, `import_scene`, `search_materials` |
| Assets and capture | 8 | `search_assets`, `insert_asset`, `generate_model`, `upload_asset`, `capture_screenshot` |
| Input simulation | 2 | `simulate_mouse_input`, `simulate_keyboard_input` |
| Misc | rest | undo/redo, `export_rbxm`/`import_rbxm`, `get_roblox_docs`, `manage_instance`, `get_connected_instances`, memory and scene analysis, plus 7 deprecated playtest aliases |

## Development

`npm test` runs the core test suite; `npm run typecheck` and `npm run lint` are available too.

## License

MIT. Original copyright belongs to the upstream author (see `LICENSE`).
