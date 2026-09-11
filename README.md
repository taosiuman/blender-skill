# Blender MCP Skill

An OpenClaw Agent Skill for connecting to and controlling Blender via the official [Blender MCP Server](https://www.blender.org/lab/mcp-server/).

**Version: 2.5.4** — Blender 5.1 + 5.2 LTS + 5.3 Alpha compatible

## What's New in v2.5.4

- **✅ 5.3 Alpha 增量更新** — 4 项新增 API
  - CyclesPreferences.has_dlss_gpu_devices: DLSS GPU 设备检测
  - CyclesRenderSettings.preview_denoising_upscale_quality: 预览降噪升级质量
  - RegionView3D.use_view_flip_x: 视口水平翻转
  - SpaceNodeOverlay.show_text_info: 节点叠加层文本信息

## What's New in v2.5.3 (Previous)

- **✅ 5.3 Alpha 增量更新** — 5 项新增/变更 API
  - WindowManager.undo_stack: 编程访问撤销栈
  - UILayout.property_split_factor: 属性分割系数控制
  - WindowManager.invoke_popup/popover 参数变更: 新增 auto_keymap 参数

## What's New in v2.5.0 (Previous)

- **🔴 PR #348: Server instructions** — `FastMCP(instructions=...)` 传递代码正确性指南
  - 解决本地化 UI 中 `nodes["Principled BSDF"]` 返回 None 的问题（#26, #110）
  - 使用 `n.type == "BSDF_PRINCIPLED"` 而非节点名称（跨语言兼容）
- **🟡 PR #344: `export_scene` 命令** — 场景导出为第一类工具
  - 支持 GLB/FBX 格式导出到自定义路径
  - 参数：`filepath`, `format="glb"`, `object_names=None`, `selection_only=False`, `apply_modifiers=True`
- **🟡 PR #345: 插件自动启动改进** — 延迟启动 + 重试机制
  - 使用 Blender 持久化定时器延迟启动（避免冷启动时序问题）
  - 启动失败后自动重试，文件加载后重新调度

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
