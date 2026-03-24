# Usage Examples

## Example 1: Full Design Workflow

**User says:** "Design a kitchen base cabinet, 60cm wide, 90cm tall, 60cm deep"

**Agent workflow:**

1. Call `design_furniture("kitchen_base", 60, 90, 60)`
2. Call `validate_structure(spec)` → expect 0 errors, 0 warnings
3. Call `generate_bom(spec)` → panels, hardware, edge banding
4. Call `optimize_cuts(parts)` → sheets and waste
5. Call `get_assembly_steps(spec)` → step-by-step instructions

**Expected summary:**

> Kitchen base cabinet 60x90x60cm in melamine 16mm:
> - 9 panels: 2 sides, 1 bottom, 2 rails, 1 back, 1 shelf, 1 kickplate, 1 door
> - Validation: 0 errors, 0 warnings
> - BOM: 8 structural panels + 1 back panel, 4.55m edge banding
> - Cuts: 2 sheets (2440x1220mm), ~62% waste (normal for a single small cabinet)
> - Assembly: 9 steps

---

## Example 2: Consult Material Standards

**User says:** "What material should I use for a 100cm wide shelf?"

**Agent workflow:**

1. Call `get_material_specs("melamine_16")` → max span 75cm (too narrow!)
2. Call `get_material_specs("mdf_18")` → max span 90cm (still too narrow!)
3. Call `get_material_specs("plywood_18")` → max span 100cm (exact fit)

**Agent recommends:** plywood_18 or solid_pine_20 for a 100cm span, or add a vertical divider with melamine_16.

---

## Example 3: Build 3D Model in FreeCAD

**User says:** "Build a bookshelf 80x200x30 in FreeCAD"

**Agent workflow:**

1. Call `design_furniture("bookshelf", 80, 200, 30)`
2. Call `build_3d_model(spec)` → builds model directly in FreeCAD
3. If user asks to see: call `mcp__freecad__get_view("Isometric")` → shows screenshot

**Note:** FreeCAD must be open with RPC server running.

---

## Example 4: Exploded View for Assembly Reference

**User says:** "Show me how the cabinet assembles"

**Agent workflow:**

1. Reuse existing spec (or generate new one)
2. Call `build_exploded_view(spec, gap_mm=80)`
3. Execute in FreeCAD and show view

---

## Example 5: Cut Optimization

**User says:** "How many sheets do I need for a closet 120x240x60?"

**Agent workflow:**

1. Call `design_furniture("closet", 120, 240, 60)` → generates spec with vertical divider (120cm > 75cm max span)
2. Call `optimize_cuts(parts)` → number of sheets and waste
3. Present the cut layout with waste percentage

---

## Example 6: Wide Cabinet with Auto-Divider

**User says:** "Design a 150cm wide bookshelf"

**Agent notes:** 150cm exceeds the max unsupported span for melamine_16 (75cm), so the engine automatically adds a vertical divider at the center.

1. Call `design_furniture("bookshelf", 150, 200, 30, "melamine_16")`
2. Spec will include a `divider_center` panel
3. Validation will note the divider was added

---

## Example 7: Assembly Specifications

**User says:** "¿Qué tornillos uso para unir los laterales al piso? ¿Necesito pegamento?"

**Agent workflow:**

1. Call `get_assembly_specs("panel_to_panel", brief=true)` → joint methods and recommendations
2. Look at `by_joint_type.side_to_bottom` → recommends confirmat + dowels, min 3 confirmats

**Agent recommends:** Confirmat 7x50mm (pre-taladrar con broca de 5mm, 40mm profundidad) + tarugos de madera 8x35mm con cola blanca. Mínimo 3 confirmats. No poner pegamento en los confirmats, solo en los tarugos.

---

## Example 8: Pre-Drilling Guide

**User says:** "¿A qué profundidad taladro para bisagras en melamina de 18mm?"

**Agent workflow:**

1. Call `get_assembly_specs("pre_drilling", brief=true)` → drilling specs per material
2. Call `get_assembly_specs("hinge_mounting", brief=true)` → full mounting process

**Agent responds:** Copa Forstner de 35mm a 13mm de profundidad. CUIDADO: el panel solo tiene 18mm. Luego fijar bisagra con tornillos 3.5x16mm.

---

## Example 9: Import and Validate from FreeCAD

**User says:** "Modifiqué el mueble en FreeCAD, agregué una repisa. ¿Puedes validarlo?"

**Agent workflow:**

1. Call `build_import_script("Furniture")` → extraction script
2. Call `mcp__freecad__execute_code(script)` → raw output with JSON
3. Call `parse_freecad_import(raw_output)` → spec with all panels
4. Check `import_warnings` — inform user if any panel has unknown role
5. Call `validate_structure(spec)` → check structural integrity
6. Call `generate_bom(spec)` → updated bill of materials
7. Call `optimize_cuts(parts)` → new cut layout with the added shelf

**Expected summary:**

> Imported 8 panels from FreeCAD (document "Furniture").
> Validation: 0 errors, 1 warning (shelf >80cm, consider reinforcement).
> BOM updated: 9 panels, 5.2m edge banding.
> Cuts: 2 sheets, 54% waste.

---

## Example 10: Import Manual Model

**User says:** "Hice un mueble desde cero en FreeCAD. ¿Puedes generar el BOM y los cortes?"

**Agent workflow:**

1. Call `build_import_script("MiMueble")` → extraction script
2. Execute and parse → spec
3. If `import_warnings` has unknown roles: "Encontré 2 paneles sin rol definido (Box003, Box005). ¿Son repisas, laterales, o algo más?"
4. After user clarifies → adjust spec and continue with `generate_bom` + `optimize_cuts`

**Note:** For best results with manual models, name objects descriptively in FreeCAD (e.g., "side_left", "shelf_1", "back") or add the `Role` property via FreeCAD's property editor.

---

## Common Options

```json
{
  "num_shelves": 4,
  "has_doors": true,
  "door_type": "double",
  "kickplate_height": 10,
  "has_modesty_panel": true
}
```

- `num_shelves`: Number of adjustable shelves
- `has_doors`: Add doors (for bookshelf, closet, wall cabinet)
- `door_type`: "single" or "double" (auto-selected based on width if omitted)
- `kickplate_height`: Kickplate height in cm (default: 10)
- `has_modesty_panel`: Front modesty panel for desks (default: true)
