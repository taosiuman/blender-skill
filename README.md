# Blender MCP Skill

An OpenClaw Agent Skill for connecting to and controlling Blender via the official [Blender MCP Server](https://www.blender.org/lab/mcp-server/).

**Version: 2.3.0** — Blender 5.1 + 5.2 LTS + 5.3 Alpha compatible

## What's New in v2.3.0

- **Blender 5.3 Alpha support** — 20 new APIs documented (Python API 6 + Geo Nodes 13 + GPU compat 1)
- **Geo Nodes volume processing** — Rasterize Points, Deactivate Voxels, Grid Topology Boolean
- **UI plugin enhancements** — NodeTreeInterface.root_panel, UILabel_multiline()
- **2D spatial queries** — mathutils.KDTree supports 2D trees
- **GPU compat warning** — gpu.types.GPUBatch.draw_instanced behavior change

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