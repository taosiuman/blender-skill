---
name: blender-mcp
description: 通过 Blender 官方 MCP Server 连接和控制 Blender。支持两种模式：(1) 完整 MCP Server + mcporter（推荐）；(2) 直连 Blender Addon TCP Socket（轻量）。覆盖 20 个内置工具 + 任意 bpy 代码执行。Use when: 用户要求通过 MCP 连接 Blender、安装 blender_mcp、配置 Blender MCP Server、或通过程序化方式控制 Blender。
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

# Blender MCP Server 连接技能

通过 Blender 官方 MCP Server 连接和控制 Blender 实例。

**版本要求**: Blender 5.1+（必须）

---

## 架构概览

```
┌─────────────┐     TCP Socket      ┌──────────────────┐     MCP Protocol     ┌──────────────┐
│  LLM Client │ ◄─────────────────► │  MCP Server      │ ◄──────────────────► │   Blender    │
│  (OpenClaw) │    stdio / HTTP     │  (Python process) │    port 9876         │   (Addon)    │
└─────────────┘                     └──────────────────┘                      └──────────────┘
```

**两组件架构**:
1. **Blender Addon** — 运行在 Blender 内部，提供 TCP Socket Server（默认 `localhost:9876`）
2. **MCP Server** — 独立 Python 进程，桥接 LLM 客户端和 Blender Addon

---

## 安装步骤

### 1. 安装 Blender Addon

**方式 A: 从 ZIP 安装**
1. 下载 addon ZIP: `https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip`
2. Blender → Edit → Preferences → Add-ons → Install from Disk
3. 启用 "Blender MCP" 插件
4. 在插件设置中确认 Host/Port（默认 `localhost:9876`）

**方式 B: 拖拽安装**
- 将 ZIP 文件拖入 Blender 窗口（需要拖拽两次：第一次添加 Blender Lab 仓库，第二次安装插件）

**方式 C: 从源码安装**
- 源码位于: `mcp/blmcp/` 和 `addon/blender_mcp_addon/`

### 2. 启动 Blender MCP Server

```bash
# 安装依赖
cd C:\Users\haowe\Downloads\blender_mcp
pip install mcp pyyaml starlette

# 启动 MCP Server（stdio 模式）
python -m blmcp --transport stdio

# 或 HTTP 模式（默认 127.0.0.1:8000）
python -m blmcp --transport http --host 127.0.0.1 --port 8000
```

### 3. 配置 mcporter

```bash
# 添加 Blender MCP Server 配置
mcporter config add blender-mcp --transport stdio --command "python -m blmcp --transport stdio"

# 或使用 HTTP 模式
mcporter config add blender-mcp --transport http --url "http://127.0.0.1:8000"

# 验证连接
mcporter list blender-mcp --schema
```

---

## 使用方式

### 方式一: 通过 mcporter 调用（推荐）

```bash
# 列出所有可用工具
mcporter list blender-mcp --schema

# 调用具体工具
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"objects": [o.name for o in bpy.data.objects]}'

mcporter call blender-mcp.get_objects_summary

mcporter call blender-mcp.get_object_detail_summary object_name="Cube"

# 搜索 API 文档
mcporter call blender-mcp.search_api_docs query="bpy.ops.object.delete"

# 搜索用户手册
mcporter call blender-mcp.search_manual_docs query="Geometry Nodes"

# 截图
mcporter call blender-mcp.get_screenshot_of_window_as_image

# 渲染 viewport
mcporter call blender-mcp.render_viewport_to_path output_path="C:\\render.png"
```

### 方式二: 直连 Blender Addon TCP Socket（轻量模式）

当不需要完整的 MCP Server 时，可以直接通过 TCP Socket 与 Blender Addon 通信：

```python
import socket
import json

def send_to_blender(code: str, host="localhost", port=9876, timeout=30.0) -> dict:
    """直接发送 Python 代码到 Blender Addon 执行"""
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

# 示例：获取场景中所有物体名称
response = send_to_blender(
    'import bpy\nresult = {"objects": [o.name for o in bpy.data.objects]}'
)
print(response)
# {"status": "ok", "result": {"objects": ["Cube", "Camera", "Light"]}}
```

### 方式三: Blender 后台模式（无头渲染/批处理）

```bash
# 启动 Blender 后台 MCP Server
blender --background myscene.blend --command blender_mcp --host localhost --port 9876

# 或使用 CLI 执行代码
blender --background myscene.blend --python-expr "
import bpy
# 你的代码
result = {'count': len(bpy.data.objects)}
print('__BLMCP_RESULT__' + str(result))
"
```

---

## 内置工具清单（20 个）

### 核心工具
| 工具名 | 描述 | 参数 |
|--------|------|------|
| `execute_blender_code` | 执行任意 Python 代码（完整 bpy 访问） | `code: str` |
| `execute_blender_code_for_cli` | 在后台 Blender 进程中执行代码 | `blend_file: str, code: str` |

### 场景分析
| 工具名 | 描述 | 参数 |
|--------|------|------|
| `get_objects_summary` | 获取场景物体层次结构和基本信息 | 无 |
| `get_object_detail_summary` | 获取指定物体的详细信息 | `object_name: str` |
| `get_blendfile_summary_datablocks` | 分析 .blend 文件的 data-blocks | `blend_file: str` |
| `get_blendfile_summary_missing_files` | 检查缺失的外部文件引用 | `blend_file: str` |
| `get_blendfile_summary_of_linked_libraries` | 列出链接的外部库 | `blend_file: str` |
| `get_blendfile_summary_path_info` | 分析 .blend 文件路径信息 | `blend_file: str` |
| `get_blendfile_summary_usage_guess` | 推测 .blend 文件用途 | `blend_file: str` |

### 截图/渲染
| 工具名 | 描述 | 参数 |
|--------|------|------|
| `get_screenshot_of_window_as_image` | 获取 Blender 窗口截图 | 无 |
| `get_screenshot_of_area_as_image` | 获取特定区域截图 | `area_type: str` |
| `get_screenshot_of_window_as_json` | 获取窗口布局的 JSON 描述 | 无 |
| `render_viewport_to_path` | 渲染 viewport 到文件 | `output_path: str` |
| `render_thumbnail_to_path` | 渲染缩略图到文件 | `output_path: str` |

### 导航
| 工具名 | 描述 | 参数 |
|--------|------|------|
| `jump_to_tab_by_name` | 跳转到指定编辑器标签 | `tab_name: str` |
| `jump_to_tab_by_space_type` | 跳转到指定空间类型 | `space_type: str` |
| `jump_to_view3d_object_by_name` | 在 3D 视图中定位到物体 | `object_name: str` |
| `jump_to_view3d_object_data_by_name` | 在 3D 视图中定位到物体数据 | `data_name: str` |

### 文档搜索
| 工具名 | 描述 | 参数 |
|--------|------|------|
| `search_api_docs` | 搜索 Blender Python API 文档 | `query: str` |
| `search_manual_docs` | 搜索 Blender 用户手册 | `query: str` |
| `get_python_api_docs` | 获取 Python API 文档 | `query: str` |

---

## 通信协议详解

### TCP Socket 协议

**请求格式**（null-byte 分隔的 JSON）:
```json
{"type": "execute", "code": "import bpy\nresult = {'key': 'value'}", "strict_json": false}\0
```

**响应格式**:
```json
// 成功
{"status": "ok", "result": {"key": "value"}, "stdout": "", "stderr": ""}\0

// 失败
{"status": "error", "message": "Traceback...", "stdout": "", "stderr": ""}\0
```

### 代码执行约定

- 必须将返回值赋值给 `result` 变量（必须为 dict）
- `strict_json=True`: 严格 JSON 序列化，非序列化值会报错
- `strict_json=False`: 使用 `repr()` 作为非 JSON 值的回退
- 支持延迟响应（deferred response）: 返回 `check_is_finished` 可调用对象用于长时间任务

### 示例：获取物体信息

```python
# 发送的代码
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

### 示例：创建物体

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

## 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `BLENDER_MCP_HOST` | `localhost` | Blender Addon 主机地址 |
| `BLENDER_MCP_PORT` | `9876` | Blender Addon 端口 |
| `BLENDER_PATH` | `blender` | Blender 可执行文件路径 |

---

## 安全注意事项

⚠️ **官方安全警告**: MCP Server 会执行 LLM 生成的代码，**没有任何安全沙箱**。

**Addon 内置的弱沙箱**（`WeakSandboxForLLM`）:
- 阻止 `sys.exit()` 调用
- 阻止危险操作符: `wm.quit_blender`, `wm.read_factory_settings`, `wm.read_factory_userpref`, `wm.read_userpref`

**建议**: 在虚拟机或无敏感数据的系统中运行。

---

## 常见错误排查

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| `ConnectionRefusedError` | Blender 未运行或 Addon 未启动 | 启动 Blender，启用 MCP Addon，点击 "Start Server" |
| `ConnectionError: Empty response` | 网络超时或 Addon 崩溃 | 检查 Blender 控制台输出，确认端口正确 |
| `result is not JSON-serializable` | 返回了 Blender 对象 | 使用 `strict_json=False` 或手动转换为 dict |
| `Blender executable not found` | 未找到 blender 命令 | 设置 `BLENDER_PATH` 环境变量 |
| `Deferred responses not supported` | 后台模式不支持延迟响应 | 使用同步代码或切换到 GUI 模式 |

---

## OpenClaw 集成

### 配置 openclaw.json

```json
{
  "plugins": {
    "entries": {
      "mcp": {
        "servers": {
          "blender-mcp": {
            "command": "python",
            "args": ["-m", "blmcp", "--transport", "stdio"],
            "cwd": "C:\\Users\\haowe\\Downloads\\blender_mcp",
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

### 通过 mcporter CLI

```bash
# 列出工具
mcporter call blender-mcp.get_objects_summary

# 执行代码
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"count": len(bpy.data.objects)}'
```

---

## 快速启动检查清单

- [ ] Blender 5.1+ 已安装
- [ ] MCP Addon 已安装并启用
- [ ] MCP Server 已启动（`python -m blmcp --transport stdio`）
- [ ] mcporter 已安装（`npm install -g mcporter`）
- [ ] 端口 9876 可用（默认）
- [ ] `BLENDER_PATH` 环境变量已设置（如需要）
