# Blender MCP Skill

An OpenClaw Agent Skill for connecting to and controlling Blender via the official [Blender MCP Server](https://www.blender.org/lab/mcp-server/).

## Features

- **20 built-in tools** — scene analysis, screenshots, rendering, navigation, doc search
- **Arbitrary bpy code execution** — full Python API access through the MCP channel
- **3 connection modes** — mcporter (recommended), direct TCP socket, background/headless
- **Blender 5.1 compatible** — includes workarounds for breaking API changes

## Quick Start

```bash
# 1. Install mcporter
npm install -g mcporter

# 2. Install the Blender MCP Addon
# Download from: https://projects.blender.org/lab/blender_mcp/releases
# Then: Edit → Preferences → Add-ons → Install from Disk

# 3. Start Blender, enable the addon, click "Start Server"

# 4. Configure mcporter
mcporter config add blender-mcp --transport stdio --command "python -m blmcp --transport stdio"

# 5. Call tools
mcporter call blender-mcp.get_objects_summary
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"count": len(bpy.data.objects)}'
```

## Repository Contents

| File | Description |
|------|-------------|
| `blender-mcp-SKILL.md` | Complete skill definition — architecture, installation, 20 tools, protocol, troubleshooting |
| `README.md` | Project overview and quick start guide |

## Requirements

- Blender 5.1+
- Python 3.13+
- Node.js 18+

## License

MIT
