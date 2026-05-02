---
name: blender-mcp
description: Connect to and control Blender via the official Blender MCP Server. Supports two modes: (1) full MCP Server + mcporter (recommended); (2) direct TCP Socket to Blender Addon (lightweight). Covers 20 built-in tools + arbitrary bpy code execution. Use when: user requests MCP connection to Blender, installs blender_mcp, configures Blender MCP Server, or wants programmatic Blender control.
homepage: https://www.blender.org/lab/mcp-server/
metadata:
  openclaw:
    emoji: 🔗
    requires:
      bins: ["blender", "mcporter"]
    install:
      - id: mcporter
        kind: node
        package: mcporter
        bins: ["mcporter"]
        label: "Install mcporter (npm)"
---

# Blender MCP Server Connection Skill

Connect to and control a running Blender instance via the official Blender MCP Server.

**Version requirement**: Blender 5.1+ (mandatory)

---

## Architecture Overview

```
┌─────────────┐     TCP Socket      ┌──────────────────     MCP Protocol     ┌──────────────
│  LLM Client │ ◄─────────────────► │  MCP Server      │ ◄──────────────────► │   Blender    │
│  (OpenClaw) │    stdio / HTTP     │  (Python process) │    port 9876         │   (Addon)    │
└─────────────┘                     └──────────────────┘                      └──────────────
```

**Two-component architecture**:
1. **Blender Addon** — runs inside Blender, provides a TCP Socket Server (default `localhost:9876`)
2. **MCP Server** — standalone Python process that bridges the LLM client and the Blender Addon

---

## Installation

### 1. Install the Blender Addon

**Option A: From ZIP**
1. Download addon ZIP: `https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip`
2. Blender → Edit → Preferences → Add-ons → Install from Disk
3. Enable the "Blender MCP" addon
4. Confirm Host/Port in addon settings (default `localhost:9876`)

**Option B: Drag & Drop**
- Drag the ZIP file into the Blender window (twice: first to add the Blender Lab repository, second to install the addon)

**Option C: From Source**
- Source code locations: `mcp/blmcp/` and `addon/blender_mcp_addon/`

### 2. Start the Blender MCP Server

```bash
# Install dependencies
cd path/to/blender_mcp
pip install mcp pyyaml starlette

# Start MCP Server (stdio mode)
python -m blmcp --transport stdio

# Or HTTP mode (default 127.0.0.1:8000)
python -m blmcp --transport http --host 127.0.0.1 --port 8000
```

### 3. Configure mcporter

```bash
# Add Blender MCP Server config
mcporter config add blender-mcp --transport stdio --command "python -m blmcp --transport stdio"

# Or use HTTP mode
mcporter config add blender-mcp --transport http --url "http://127.0.0.1:8000"

# Verify connection
mcporter list blender-mcp --schema
```

---

## Usage

### Method 1: Via mcporter (recommended)

```bash
# List all available tools
mcporter list blender-mcp --schema

# Call a specific tool
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"objects": [o.name for o in bpy.data.objects]}'

mcporter call blender-mcp.get_objects_summary

mcporter call blender-mcp.get_object_detail_summary object_name="Cube"

# Search API docs
mcporter call blender-mcp.search_api_docs query="bpy.ops.object.delete"

# Search user manual
mcporter call blender-mcp.search_manual_docs query="Geometry Nodes"

# Screenshot
mcporter call blender-mcp.get_screenshot_of_window_as_image

# Render viewport
mcporter call blender-mcp.render_viewport_to_path output_path="C:\\render.png"
```

### Method 2: Direct TCP Socket to Blender Addon (lightweight)

When you don't need the full MCP Server, you can communicate directly with the Blender Addon via TCP Socket:

```python
import socket
import json

def send_to_blender(code: str, host="localhost", port=9876, timeout=30.0) -> dict:
    """Send Python code directly to Blender Addon for execution."""
    request = json.dumps({
        "type": "execute",
        "code": code,
        "strict_json": False,
    }) + "\0"
    
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.settimeout(timeout)
        sock.connect((host, port))
        sock.sendall(request.encode("utf-8"))
        
        buf = bytearray()
        while True:
            chunk = sock.recv(65536)
            if not chunk:
                break
            buf.extend(chunk)
            if b"\0" in buf:
                break
    
    line, _, _ = buf.partition(b"\0")
    return json.loads(line.decode("utf-8"))

# Example: get all object names in the scene
response = send_to_blender(
    'import bpy\nresult = {"objects": [o.name for o in bpy.data.objects]}'
)
print(response)
# {"status": "ok", "result": {"objects": ["Cube", "Camera", "Light"]}}
```

### Method 3: Blender Background Mode (headless rendering / batch)

```bash
# Start Blender background MCP Server
blender --background myscene.blend --command blender_mcp --host localhost --port 9876

# Or execute code via CLI
blender --background myscene.blend --python-expr "
import bpy
# your code
result = {'count': len(bpy.data.objects)}
print('__BLMCP_RESULT__' + str(result))
"
```

---

## Built-in Tools (20 total)

### Core
| Tool | Description | Params |
|------|-------------|--------|
| `execute_blender_code` | Execute arbitrary Python code (full bpy access) | `code: str` |
| `execute_blender_code_for_cli` | Execute code in a background Blender process | `blend_file: str, code: str` |

### Scene Analysis
| Tool | Description | Params |
|------|-------------|--------|
| `get_objects_summary` | Get scene object hierarchy and basic info | none |
| `get_object_detail_summary` | Get detailed info for a specific object | `object_name: str` |
| `get_blendfile_summary_datablocks` | Analyze .blend file data-blocks | `blend_file: str` |
| `get_blendfile_summary_missing_files` | Check for missing external file references | `blend_file: str` |
| `get_blendfile_summary_of_linked_libraries` | List linked external libraries | `blend_file: str` |
| `get_blendfile_summary_path_info` | Analyze .blend file path information | `blend_file: str` |
| `get_blendfile_summary_usage_guess` | Guess the purpose of a .blend file | `blend_file: str` |

### Screenshots & Rendering
| Tool | Description | Params |
|------|-------------|--------|
| `get_screenshot_of_window_as_image` | Capture Blender window screenshot | none |
| `get_screenshot_of_area_as_image` | Capture specific area screenshot | `area_type: str` |
| `get_screenshot_of_window_as_json` | Get window layout as JSON description | none |
| `render_viewport_to_path` | Render viewport to file | `output_path: str` |
| `render_thumbnail_to_path` | Render thumbnail to file | `output_path: str` |

### Navigation
| Tool | Description | Params |
|------|-------------|--------|
| `jump_to_tab_by_name` | Jump to a named editor tab | `tab_name: str` |
| `jump_to_tab_by_space_type` | Jump to a space type | `space_type: str` |
| `jump_to_view3d_object_by_name` | Focus on an object in 3D View | `object_name: str` |
| `jump_to_view3d_object_data_by_name` | Focus on object data in 3D View | `data_name: str` |

### Documentation Search
| Tool | Description | Params |
|------|-------------|--------|
| `search_api_docs` | Search Blender Python API docs | `query: str` |
| `search_manual_docs` | Search Blender user manual | `query: str` |
| `get_python_api_docs` | Get Python API docs | `query: str` |

---

## Communication Protocol

### TCP Socket Protocol

**Request format** (null-byte delimited JSON):
```json
{"type": "execute", "code": "import bpy\nresult = {'key': 'value'}", "strict_json": false}\0
```

**Response format**:
```json
// Success
{"status": "ok", "result": {"key": "value"}, "stdout": "", "stderr": ""}\0

// Error
{"status": "error", "message": "Traceback...", "stdout": "", "stderr": ""}\0
```

### Code Execution Conventions

- Assign return value to the `result` variable (must be a dict)
- `strict_json=True`: strict JSON serialization; non-serializable values will error
- `strict_json=False`: uses `repr()` as fallback for non-JSON values
- Supports deferred responses: return a `check_is_finished` callable for long-running tasks

### Example: Get Object Info

```python
import bpy
obj = bpy.data.objects.get("Cube")
if obj:
    result = {
        "name": obj.name,
        "type": obj.type,
        "location": list(obj.location),
        "vertices": len(obj.data.vertices) if obj.type == "MESH" else 0,
    }
else:
    result = {"error": "Object not found"}
```

### Example: Create an Object

```python
import bpy
bpy.ops.mesh.primitive_torus_add(
    align='WORLD',
    location=(0, 0, 0),
    major_radius=1.0,
    minor_radius=0.3,
)
result = {"status": "created", "name": bpy.context.active_object.name}
```

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `BLENDER_MCP_HOST` | `localhost` | Blender Addon host address |
| `BLENDER_MCP_PORT` | `9876` | Blender Addon port |
| `BLENDER_PATH` | `blender` | Path to Blender executable |

---

## Security Notes

⚠️ **Official security warning**: The MCP Server executes LLM-generated code with **no sandboxing**.

**Built-in weak sandbox** (`WeakSandboxForLLM`):
- Blocks `sys.exit()` calls
- Blocks dangerous operators: `wm.quit_blender`, `wm.read_factory_settings`, `wm.read_factory_userpref`, `wm.read_userpref`

**Recommendation**: Run in a VM or on a system without sensitive data.

---

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `ConnectionRefusedError` | Blender not running or Addon not started | Start Blender, enable MCP Addon, click "Start Server" |
| `ConnectionError: Empty response` | Network timeout or Addon crashed | Check Blender console output, verify port |
| `result is not JSON-serializable` | Returned a Blender object | Use `strict_json=False` or manually convert to dict |
| `Blender executable not found` | Blender command not found | Set `BLENDER_PATH` environment variable |
| `Deferred responses not supported` | Background mode doesn't support deferred responses | Use synchronous code or switch to GUI mode |

---

## Blender 5.1 API Compatibility Notes

### Action Layered Structure (Breaking Change)

In Blender 5.1, `Action.fcurves` was removed. The new layered animation system uses:

```python
# ❌ Old (5.0 and earlier)
fcurves = action.fcurves

# ✅ Blender 5.1+
fcurves = action.layers[0].strips[0].channelbags[0].fcurves
```

### Principled BSDF Node Input Renames

| Old (5.0) | New (5.1) |
|-----------|-----------|
| `inputs['Transmission']` | `inputs['Transmission Weight']` |
| `inputs['Emission']` | `inputs['Emission Color']` + `inputs['Emission Strength']` |

---

## OpenClaw Integration

### Configure openclaw.json

```json
{
  "plugins": {
    "entries": {
      "mcp": {
        "servers": {
          "blender-mcp": {
            "command": "python",
            "args": ["-m", "blmcp", "--transport", "stdio"],
            "cwd": "path/to/blender_mcp",
            "env": {
              "BLENDER_MCP_HOST": "localhost",
              "BLENDER_MCP_PORT": "9876"
            }
          }
        }
      }
    }
  }
}
```

### Via mcporter CLI

```bash
# List tools
mcporter call blender-mcp.get_objects_summary

# Execute code
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"count": len(bpy.data.objects)}'
```

---

## Quick Start Checklist

- [ ] Blender 5.1+ installed
- [ ] MCP Addon installed and enabled
- [ ] MCP Server started (`python -m blmcp --transport stdio`)
- [ ] mcporter installed (`npm install -g mcporter`)
- [ ] Port 9876 available (default)
- [ ] `BLENDER_PATH` environment variable set (if needed)
