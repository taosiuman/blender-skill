# Blender Skill

OpenClaw Agent 技能 — 通过 Blender 官方 MCP Server 连接和控制 Blender。

## 技能文件

- `blender-mcp-SKILL.md` — Blender MCP 连接技能，支持 20 个内置工具 + 任意 bpy 代码执行

## 快速开始

```bash
# 安装 mcporter
npm install -g mcporter

# 配置 Blender MCP Server
mcporter config add blender-mcp --transport stdio --command "python -m blmcp --transport stdio"

# 调用工具
mcporter call blender-mcp.get_objects_summary
```

## 版本要求

- Blender 5.1+
- Python 3.13+
- Node.js 18+

## 许可证

MIT
