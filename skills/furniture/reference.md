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
- `brief` (bool, default: `false`): Return compact summary instead of full JSON. **Recommended: use `true`.**

**Returns:** Min/max/default dimensions, clearances, and ergonomic notes.

### `get_material_specs`

Get technical specifications for a material.

**Parameters:**
- `material` (string, required): One of: `mdf_15`, `mdf_18`, `melamine_16`, `melamine_18`, `plywood_18`, `solid_pine_20`
- `brief` (bool, default: `false`): Return compact summary instead of full JSON. **Recommended: use `true`.**

**Returns:** Thickness, max unsupported span, sheet sizes, edge banding options.

### `get_structural_rules`

Get all structural validation rules.

**Parameters:**
- `brief` (bool, default: `false`): Return compact summary instead of full JSON. **Recommended: use `true`.**

**Returns:** 10 rules covering back panels, max spans, cross rails, vertical dividers, kickplates, shelf reinforcement, tall furniture anchoring, confirmat spacing, floor panel load bearing.

### `get_hardware_catalog`

Get hardware catalog with selection rules.

**Parameters:**
- `category` (string, optional): One of: `hinges`, `slides`, `connectors`, `shelf_pins`. Omit for full catalog.
- `brief` (bool, default: `false`): Return compact summary instead of full JSON. **Recommended: use `true`.**

**Returns:** Hardware specs with placement rules and quantity formulas.

### `get_assembly_specs`

Get detailed assembly specifications for furniture construction.

**Parameters:**
- `topic` (string, optional): One of: `panel_to_panel`, `back_panel`, `hinge_mounting`, `drawer_slide_mounting`, `shelf_pins`, `adhesive_guide`, `pre_drilling`. Omit for all topics.
- `brief` (bool, default: `false`): Return compact summary instead of full JSON. **Recommended: use `true`.**

**Returns:** Assembly specifications with step-by-step processes, fastener types, joint methods, adhesive recommendations, and material-specific pre-drilling depths.

**Topics:**

| Topic | Content |
|-------|---------|
| `panel_to_panel` | Joint methods (confirmat, confirmat+dowel, minifix), quantity rules by panel length, recommendations per joint type |
| `back_panel` | Nailed vs grooved installation, spacing, adhesive usage |
| `hinge_mounting` | Cup drilling specs, placement rules, 3-axis adjustment, overlay types |
| `drawer_slide_mounting` | Clearance calculations, step-by-step process, common mistakes |
| `shelf_pins` | 32mm system, hole depths, jig recommendation |
| `adhesive_guide` | PVA, contact cement, polyurethane — when to use and when NOT to use each |
| `pre_drilling` | Pilot hole diameters and depths per material for confirmats, dowels, hinges, shelf pins |

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
- `parts` (list, required): Parts with `id`, `width` (mm), `height` (mm), `qty`, `can_rotate`, `grain`
  - `grain` per piece: `"length"` (grain along width — no rotation), `"width"` (grain along height — auto-rotates), `"none"` (free rotation)
- `sheet_width` (float, default: 2440): Sheet width in mm. Grain runs along this axis on the sheet.
- `sheet_height` (float, default: 1220): Sheet height in mm
- `blade_kerf` (float, default: 3): Saw blade width in mm
- `grain_direction` (string, default: `"auto"`): Global grain default. `"auto"` = per-piece, `"length"` = all pieces respect grain, `"none"` = free rotation.

**Returns:** Sheets used, piece positions, waste percentage, grain arrows, text diagram.

**Notes:**
- Blade kerf (3mm default) is deducted from every cut — both between shelves and between pieces within a shelf.
- When `grain="length"`, pieces are locked to their original orientation to keep the grain pattern aligned horizontally on the sheet.
- For melamine with visible grain pattern, use `grain_direction="length"` to ensure consistent appearance across all pieces.

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

---

## FreeCAD Import Tools

These tools read existing FreeCAD models back into the furniture spec format.

### `build_import_script`

Generate a FreeCAD script that extracts all panels from an existing document.

**Parameters:**
- `doc_name` (string, default: "Furniture"): FreeCAD document name to read

**Returns:** Python code. Execute with `mcp__freecad__execute_code`. The stdout contains JSON prefixed with `FURNITURE_SPEC_JSON:`.

**Reads from each panel:**
- App::Part with custom properties → Role, Material, Thickness_mm, RealDimensions, EdgeBanding, Placement
- Standalone Part::Box → Length, Width, Height, Placement (roles inferred from name/geometry)

### `parse_freecad_import`

Parse the import script output into a furniture spec.

**Parameters:**
- `raw_output` (string, required): The stdout from executing the import script

**Returns:** A furniture spec (JSON) compatible with `validate_structure`, `generate_bom`, `optimize_cuts`, etc.

**Behavior:**
- Panels with custom properties (from our system) → exact reconstruction
- Part::Box with names like "side_left", "shelf_1" → role inferred from name
- Part::Box with generic names → role inferred from geometry (thinnest axis)
- Unrecognizable panels → `role: "unknown"` with warnings in `import_warnings`

**Workflow:**
```
build_import_script("MyDoc")
    → execute_code(script) → raw output
        → parse_freecad_import(raw) → spec
            → validate_structure(spec) / generate_bom(spec) / ...
```
