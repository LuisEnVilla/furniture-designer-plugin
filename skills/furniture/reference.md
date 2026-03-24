# Tool Reference

Complete reference for all `furniture-designer` MCP tools.

---

## Design Tools

### `design_furniture`

Generate a complete furniture spec with standards applied.

**Parameters:**
- `furniture_type` (string, required): One of: `kitchen_base`, `kitchen_wall`, `closet`, `bookshelf`, `desk`, `vanity`
- `width` (float, required): Total width in cm
- `height` (float, required): Total height in cm
- `depth` (float, required): Total depth in cm
- `material` (string, default: `melamine_16`): Material key
- `options` (dict, optional): Overrides like `{"num_shelves": 3, "has_doors": true, "door_type": "double", "kickplate_height": 10}`

**Returns:** Complete furniture spec (JSON) with parts list, positions (mm), hardware, and structural notes.

**Notes:**
- Input dimensions are in cm, output dimensions in mm (manufacturing standard)
- Automatically adds vertical dividers when width exceeds material max span
- Back panels are always 3mm MDF
- Edge banding is auto-assigned by panel role

---

## Knowledge Tools

### `get_standards`

Get ergonomic standards for a furniture type.

**Parameters:**
- `furniture_type` (string, required): One of: `kitchen_base`, `kitchen_wall`, `closet`, `bookshelf`, `desk`, `vanity`, `general`

**Returns:** Min/max/default dimensions, clearances, and ergonomic notes.

### `get_material_specs`

Get technical specifications for a material.

**Parameters:**
- `material` (string, required): One of: `mdf_15`, `mdf_18`, `melamine_16`, `melamine_18`, `plywood_18`, `solid_pine_20`

**Returns:** Thickness, max unsupported span, sheet sizes, edge banding options.

### `get_structural_rules`

Get all structural validation rules.

**Returns:** 10 rules covering back panels, max spans, cross rails, vertical dividers, kickplates, shelf reinforcement, tall furniture anchoring, confirmat spacing, floor panel load bearing.

### `get_hardware_catalog`

Get hardware catalog with selection rules.

**Parameters:**
- `category` (string, optional): One of: `hinges`, `slides`, `connectors`, `shelf_pins`. Omit for full catalog.

**Returns:** Hardware specs with placement rules and quantity formulas.

---

## Validation & Manufacturing Tools

### `validate_structure`

Validate a furniture spec against structural rules.

**Parameters:**
- `spec` (dict, required): A furniture spec as returned by `design_furniture`

**Returns:** Validation report with `errors` (must fix) and `warnings` (recommended).

### `generate_bom`

Generate a Bill of Materials.

**Parameters:**
- `spec` (dict, required): A furniture spec

**Returns:** Grouped panels (with quantities), back panels, hardware list, edge banding details and totals in meters, summary.

### `optimize_cuts`

Optimize panel cuts on standard sheets using guillotine algorithm.

**Parameters:**
- `parts` (list, required): Parts with `id`, `width` (mm), `height` (mm), `qty`, `can_rotate`
- `sheet_width` (float, default: 2440): Sheet width in mm
- `sheet_height` (float, default: 1220): Sheet height in mm
- `blade_kerf` (float, default: 3): Saw blade width in mm

**Returns:** Sheets used, piece positions, waste percentage, text diagram.

### `get_assembly_steps`

Generate step-by-step assembly instructions.

**Parameters:**
- `spec` (dict, required): A furniture spec

**Returns:** Ordered assembly steps with part references, hardware per step, and tips (in Spanish).

---

## FreeCAD Bridge Tools

These tools generate Python scripts to execute in FreeCAD via the `freecad` MCP server.

### `build_3d_model`

Generate a FreeCAD script that builds the furniture as a 3D model.

**Parameters:**
- `spec` (dict, required): A furniture spec
- `doc_name` (string, default: "Furniture"): FreeCAD document name

**Returns:** Python code. Execute with `mcp__freecad__execute_code`.

**Panel colors by role:**
- Sides: warm wood (0.90, 0.75, 0.55)
- Bottom/Top: slightly darker (0.85, 0.70, 0.50)
- Shelves: green tint (0.70, 0.85, 0.65)
- Back: gray (0.75, 0.75, 0.75)
- Doors: blue tint (0.60, 0.75, 0.90)
- Rails: dark brown (0.65, 0.55, 0.42)
- Kickplate: dark gray (0.50, 0.50, 0.50)
- Dividers: orange (0.90, 0.72, 0.40)

### `build_exploded_view`

Generate a FreeCAD script for an exploded assembly view.

**Parameters:**
- `spec` (dict, required): A furniture spec
- `gap_mm` (float, default: 50): Gap between exploded parts in mm
- `doc_name` (string, default: "Exploded"): FreeCAD document name

**Returns:** Python code. Panels separate along their assembly axis.

### `build_cut_diagram`

Generate a FreeCAD script to visualize cut optimization layout.

**Parameters:**
- `cut_result` (dict, required): Result from `optimize_cuts`
- `doc_name` (string, default: "CutLayout"): FreeCAD document name

**Returns:** Python code. Top-down view of sheets with placed panels.
