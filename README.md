# Blender MCP Skill

[![Architecture diagram](https://gitdiagram.com/diagram-badge.svg)](https://gitdiagram.com/taosiuman/blender-skill?utm_source=readme&utm_medium=badge)

An OpenClaw Agent Skill for connecting to and controlling Blender via the official [Blender MCP Server](https://www.blender.org/lab/mcp-server/).

用于连接和控制 Blender 的 OpenClaw Agent 技能，通过官方 [Blender MCP Server](https://www.blender.org/lab/mcp-server/) 实现。

**Version: 3.1.1** — Blender 5.1 + 5.2 LTS（当前 5.2.2）+ 5.3 dev compatible

---

## ⚠️ Security Warning / 安全警告

**English:**
This skill provides full access to Blender's Python API (`bpy`) through the MCP channel. Code executed via `execute_blender_code` can:
- Read, modify, or delete files on your system
- Execute arbitrary Python code (including network requests)
- Alter or corrupt your Blender project data

**Always review code before execution. Keep backups of important `.blend` files.**

**中文：**
此技能通过 MCP 通道提供对 Blender Python API (`bpy`) 的完全访问。通过 `execute_blender_code` 执行的代码可以：
- 读取、修改或删除系统上的文件
- 执行任意 Python 代码（包括网络请求）
- 更改或损坏 Blender 项目数据

**执行前请始终审查代码。保持重要 `.blend` 文件的备份。**

---

## What's New in v3.1.1 / 更新内容

### Docs Consistency Fix / 文档一致性修复（v3.1.1 · 2026-10-08）
- ✅ README 变更日志补齐 v3.0.0 / v3.1.0 / v3.1.1 —— 修复「版本行写 3.1.0、正文却停在 v2.5.9」的版本漂移
- ✅ README changelog caught up with the actual version (no more v2.5.9 drift)
- ✅ 核验日期更新至 2026-10-07（10-06 / 10-07 每日快扫均无 API 增量）
- ✅ 无 API 内容变更，纯文档版本 / Docs-only release, no API content change

### Blender 5.3 API Verification / 5.3 API 全量核验（v3.1.0 · 2026-10-05）
- 🔴 **函数签名变更**（按位置传参会静默出错或 TypeError）：`ObjectConstraints.new` / `PoseBoneConstraints.new` 由 `(type)` → `(type, name)`；`UILayout.textbox` / `textbox_with_state` 位置参数重排；`WindowManager.popover*` 新增 `auto_keymap` → **一律改用关键字传参**
- 🟢 新增 API：`BlendFileColorspace` OCIO 诊断三项、`MaterialGPencilStyle.gradient`、`SpaceOutliner.show_users_column` / `sort_method`、`UILayout.label_markdown`
- 📌 完整清单见工作区 `knowledge/blender/api-changes-5.3.md`
- 📌 版本状态：最新稳定版 **5.2.2 LTS**（2026-09-15）；5.3 官方 API 文档已生成但未发布 → 按前瞻兼容处理

### Workspace Rebuild / 工作区重建（v3.0.0 · 2026-10-05）
- 🏗 技能元数据 `metadata.openclaw` → `metadata.octop`（现由 octop agent **B 哥** 维护）
- 🏗 知识库拆分：API 变更明细移入工作区 `knowledge/`，技能保留「速览 + 兼容模板 + 排障」
- 🏗 修复版本号漂移：`version` / README 版本行 / Changelog 三处强制一致

### Previous Updates / 之前的更新

**v2.5.9 (2026-09-23):** Security release — removed dangerous `curl | sh` pattern, added explicit bpy-execution warnings, pinned dependency versions, bilingual (EN/CN) docs

**v2.5.8:** 8 new APIs + 2 breaking changes documented
- CollectionChild.sort_index, ColorManagedInputColorspaceSettings.interop_id, etc.

**v2.5.7:** BrushGpencilSettings.use_cyclic_stroke (Grease Pencil cyclic stroke)

**v2.5.6:** PointCloud.type (point cloud data access)

---

## Features / 功能

- **20 built-in tools** — scene analysis, screenshots, rendering, navigation, doc search
- **Arbitrary bpy code execution** — full Python API access through the MCP channel
- **3 connection modes** — mcporter (recommended), direct TCP socket, background/headless
- **Geometry Nodes MCP integration** — data relationship queries, scene debugging, node tree inspection
- **Auto Start** — enable in addon preferences to auto-start MCP Server on Blender launch
- **Blender 5.1 / 5.2 LTS / 5.3 Alpha compatible** — includes workarounds for all breaking API changes

- **20 个内置工具** — 场景分析、截图、渲染、导航、文档搜索
- **任意 bpy 代码执行** — 通过 MCP 通道访问完整的 Python API
- **3 种连接模式** — mcporter（推荐）、直接 TCP 套接字、后台/无头
- **几何节点 MCP 集成** — 数据关系查询、场景调试、节点树检查
- **自动启动** — 在插件首选项中启用，Blender 启动时自动启动 MCP Server
- **兼容 Blender 5.1 / 5.2 LTS / 5.3 Alpha** — 包含所有破坏性 API 变更的解决方案

---

## Quick Start / 快速开始

### Option A: mcporter (recommended / 推荐)
```bash
npm install -g mcporter
mcporter config add blender-mcp --transport stdio --command "python -m blmcp --transport stdio"
```

### Option B: uvx (with pinned version / 固定版本)
```bash
# Install uv first (safe method / 安全方法)
# Windows: Download from https://github.com/astral-sh/uv/releases or use winget
# Mac/Linux: curl -LsSf https://astral.sh/uv/install.sh -o install.sh && chmod +x install.sh && ./install.sh

# Run with pinned version / 使用固定版本运行
uvx blender-mcp==1.0.0
```

### Then call tools / 然后调用工具
```bash
mcporter call blender-mcp.get_objects_summary
mcporter call blender-mcp.execute_blender_code code='import bpy; result = {"count": len(bpy.data.objects)}'
```

---

## Install Dependencies / 安装依赖

```bash
# Install with pinned versions / 使用固定版本安装
pip install mcp==1.8.0 pyyaml==6.0.2 starlette==0.45.0

# Start MCP Server (stdio mode)
python -m blmcp --transport stdio

# Or HTTP mode
python -m blmcp --transport http --host 127.0.0.1 --port 8000
```

---

## Telemetry / 遥测

**English:** Anonymous usage telemetry may be collected by default. To disable:
```bash
export DISABLE_TELEMETRY=true  # Linux/Mac
set DISABLE_TELEMETRY=true    # Windows CMD
$env:DISABLE_TELEMETRY=true   # PowerShell
```

**中文：** 默认情况下可能会收集匿名使用遥测数据。禁用方法：
```bash
export DISABLE_TELEMETRY=true  # Linux/Mac
set DISABLE_TELEMETRY=true    # Windows CMD
$env:DISABLE_TELEMETRY=true   # PowerShell
```

---

## Requirements / 系统要求

- Blender 5.1+ (5.2.x LTS + 5.3 dev supported)
- Python 3.13+
- Node.js 18+ (for mcporter)

---

## Blender Version Support / Blender 版本支持

| Version | Status | Support |
|---------|--------|---------|
| 5.2 LTS (current 5.2.2) | Stable | Until 2028-07 |
| 5.3 | In Development (docs generated, not released) | 前瞻兼容，最新核验 2026-10-07 |
| 5.1 | Stable | Legacy |
| 4.5 LTS | Supported | Until 2027-07 |

---

## Repository Contents / 仓库内容

| File | Description |
|------|-------------|
| SKILL.md | Complete skill definition — architecture, installation, 20 tools, protocol, 5.1/5.2/5.3 compatibility notes, troubleshooting |
| README.md | Project overview and quick start guide (bilingual) |

| 文件 | 描述 |
|------|------|
| SKILL.md | 完整的技能定义 — 架构、安装、20 个工具、协议、5.1/5.2/5.3 兼容性说明、故障排除 |
| README.md | 项目概述和快速入门指南（双语） |

---

## License / 许可证

Apache-2.0

---

## Changelog / 变更日志

### v2.5.9 (2026-09-23) — Security Release / 安全发布
- 🔒 Removed dangerous `curl | sh` pattern from documentation / 从文档中移除危险的 `curl | sh` 模式
- 🔒 Added explicit security warnings about bpy code execution / 添加关于 bpy 代码执行的明确安全警告
- 🔒 Pinned dependency versions (uvx, pip) / 固定依赖版本（uvx、pip）
- 🔒 Clarified telemetry defaults and opt-out / 明确遥测默认值和退出方式
- 🔒 Added bilingual documentation (EN/CN) / 添加双语文档（中/英）
- 🔒 Removed release automation scripts (unrelated to skill) / 移除发布自动化脚本（与技能无关）

### v2.5.8 (2026-09-23)
- ✅ 5.3 Alpha API update: 8 new APIs + 2 breaking changes documented

### v2.5.7 (2026-09-20)
- ✅ BrushGpencilSettings.use_cyclic_stroke

### v2.5.6 (2026-09-18)
- ✅ PointCloud.type
ender/api-changes-5.3.md`
- 📌 版本状态：最新稳定版 5.2.2 LTS（2026-09-15）；5.3 文档已生成但未发布

### v3.0.0 (2026-10-05) — Workspace Rebuild / 工作区重建
- 🏗 技能元数据 `metadata.openclaw` → `metadata.octop`
- 🏗 知识库拆分：API 变更明细移入工作区 `knowledge/`，技能保留速览 + 兼容模板
- 🏗 修复版本号漂移：`version` / README 版本行 / Changelog 三处强制一致

### v2.5.9 (2026-09-23) — Security Release / 安全发布
- 🔒 Removed dangerous `curl | sh` pattern from documentation / 从文档中移除危险的 `curl | sh` 模式
- 🔒 Added explicit security warnings about bpy code execution / 添加关于 bpy 代码执行的明确安全警告
- 🔒 Pinned dependency versions (uvx, pip) / 固定依赖版本（uvx、pip）
- 🔒 Clarified telemetry defaults and opt-out / 明确遥测默认值和退出方式
- 🔒 Added bilingual documentation (EN/CN) / 添加双语文档（中/英）
- 🔒 Removed release automation scripts (unrelated to skill) / 移除发布自动化脚本（与技能无关）

### v2.5.8 (2026-09-23)
- ✅ 5.3 Alpha API update: 8 new APIs + 2 breaking changes documented

### v2.5.7 (2026-09-20)
- ✅ BrushGpencilSettings.use_cyclic_stroke

### v2.5.6 (2026-09-18)
- ✅ PointCloud.type
