# Blender MCP Skill

An OpenClaw Agent Skill for connecting to and controlling Blender via the official [Blender MCP Server](https://www.blender.org/lab/mcp-server/).

**Version: 2.4.0** — Blender 5.1 + 5.2 LTS + 5.3 Alpha compatible

## What's New in v2.4.0

- **5.3 Alpha complete scan** — 50+ new Python APIs documented from official change_log.html
- **🔴 Critical breaking change** — `NodesModifier.panels` removed (affects all Geo Nodes modifier plugins!)
- **🔴 Theme API removals** — `ThemeFileBrowser.selected_file`, `ThemeSpaceGeneric/Gradient.header_text` removed
- **Project API** — `BlendData.project/project_init/project_clear` for project management
- **Render Pause/Resume** — `RenderEngine.view_pause/view_resume`, `RegionView3D.pause_render`
- **Scene Compositor Effects** — `Scene.compositor_effects` for scene-level compositing
- **ID deep_hash** — Content-based hashing for all data blocks
- **Outliner 11 filters** — Fine-grained filtering for materials, modifiers, constraints, etc.
- **Asset Library auth** — `UserAssetLibrary.auth_token/use_auth_token/uuid` for online libraries
- **Brush enhancements** — `curve_auto_smooth/curve_hardness/curve_spacing` + unified properties
- **Rotation conversion** — `Object/PoseBone.convert_rotation_mode()`
- **EEVEE denoising** — `ViewLayerEEVEE.denoising_store_passes`
- **Brush rename corrected** — `use_inverse_smooth_pressure` → `use_smooth_pressure` (not reversed)

## Features

- **20 built-in tools** — scene analysis, screenshots, rendering, navigation, doc search
- **Arbitrary bpy code execution** — full Python API access through the MCP channel
- **3 connection modes** — mcporter (recommended), direct TCP socket, background/headless
- **Geometry Nodes MCP integration** — data relationship queries, scene debugging, node tree inspection
- **Auto Start** — enable in addon preferences to auto-start MCP Server on Blender launch
- **uvx quick install** — uvx blender-mcp one-command startup
- **Blender 5.1 / 5.2 LTS / 5.3 Alpha compatible** — includes workarounds for all breaking API changes

## Quick Start

```bash
# Option A: mcporter (recommended)
npm install -g mcporter
mcporter config add blender-mcp --transport stdio --command "python -m blmcp --transport stdio"

# Option B: uvx (simplest)
uvx blender-mcp

# Then call tools
mcporter call blender-mcp.get_objects_summary
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"count": len(bpy.data.objects)}'
```

## Repository Contents

| File | Description |
|------|-------------|
| SKILL.md | Complete skill definition — architecture, installation, 20 tools, protocol, 5.1/5.2/5.3 compatibility notes, troubleshooting |
| README.md | Project overview and quick start guide |

## Requirements

- Blender 5.1+ (5.2 LTS + 5.3 Alpha supported)
- Python 3.13+
- Node.js 18+

## Blender Version Support

| Version | Status | Support |
|---------|--------|---------|
| 5.2 LTS | Stable | Until 2028-07 |
| 5.3 Alpha | In Development | Alpha until 2026-09-30 |
| 5.1 | Stable | Legacy |
| 4.5 LTS | Supported | Until 2027-07 |

## License

Apache-2.0