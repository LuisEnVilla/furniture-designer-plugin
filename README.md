# furniture-designer-plugin

Claude Code plugin for professional furniture design. Adds an AI skill that understands furniture engineering, structural validation, cut optimization, and 3D visualization.

## What This Plugin Does

When installed, your AI agent gains the ability to:

- **Design furniture** with proper ergonomic standards and structural rules
- **Validate structures** against 10 engineering rules (back panels, max spans, reinforcements, etc.)
- **Generate Bills of Materials** with panel counts, hardware, and edge banding totals
- **Optimize panel cuts** using guillotine algorithm on standard sheets (2440x1220mm)
- **Generate assembly instructions** with step-by-step guidance
- **Build 3D models** in FreeCAD (optional, requires freecad-mcp)

## Install

```bash
claude plugin add https://github.com/LuisEnVilla/furniture-designer-plugin
```

This automatically:
1. Installs the `furniture-designer-mcp` server (via uvx)
2. Registers the `/furniture-designer:furniture` skill

## Usage

### Invoke the skill directly

```
/furniture-designer:furniture design a kitchen base cabinet 60x90x60
```

### Or just talk naturally

The AI agent will automatically invoke the skill when you mention furniture design:

```
"I need a bookshelf, 80cm wide, 200cm tall, 30cm deep"
"How many sheets do I need for a closet 120x240x60?"
"What material supports a 100cm shelf span?"
"Build the cabinet in FreeCAD"
```

## Optional: FreeCAD 3D Visualization

For 3D model building, exploded views, and cut diagrams in FreeCAD:

1. Install [FreeCAD](https://www.freecad.org/) (0.21+)
2. Install the MCP addon: `uvx freecad-mcp`
3. Open FreeCAD and start the RPC server (MCP Addon workbench > Start RPC Server)
4. Add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "freecad": {
      "command": "uvx",
      "args": ["freecad-mcp"]
    }
  }
}
```

## Supported Furniture Types

| Type | Description |
|------|-------------|
| `kitchen_base` | Kitchen base cabinet with kickplate and rails |
| `kitchen_wall` | Wall-mounted cabinet |
| `closet` | Wardrobe/closet with kickplate |
| `bookshelf` | Open shelving with optional doors |
| `desk` | Writing desk with leg clearance |
| `vanity` | Bathroom vanity |

## Supported Materials

| Material | Thickness | Max Unsupported Span |
|----------|-----------|---------------------|
| `melamine_16` | 16mm | 75cm |
| `melamine_18` | 18mm | 85cm |
| `mdf_15` | 15mm | 80cm |
| `mdf_18` | 18mm | 90cm |
| `plywood_18` | 18mm | 100cm |
| `solid_pine_20` | 20mm | 120cm |

## Architecture

This plugin is the UX layer. The engine lives in a separate package:

- **[furniture-designer-mcp](https://github.com/LuisEnVilla/furniture-designer-mcp)** — MCP Server with 12 tools (works with any MCP client)
- **This plugin** — Claude Code skill that teaches the AI agent how to orchestrate those tools

## License

MIT
