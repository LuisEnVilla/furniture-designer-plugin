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
- `options` (dict, optional): Overrides — ver tabla de opciones abajo
- `compact` (bool, default: `true`): Return compact summary + JSON. Set `false` for pretty-printed JSON only.

**Returns:** Compact summary + furniture spec JSON (or full JSON if compact=false).

**Opciones disponibles (`options` dict):**

| Opción | Tipo | Default | Descripción |
|--------|------|---------|-------------|
| `num_shelves` | int | auto (por altura) | Cantidad de repisas. **Máximo: 20.** |
| `num_drawers` | int | `0` | Cantidad de cajones. **Máximo: 10.** Genera caja completa (5 partes + correderas). |
| `drawer_height` | int | `140` | Altura visible del frente de cajón en mm. |
| `has_doors` | bool | `false` | Agregar puertas (solo `closet`/`bookshelf`). |
| `door_type` | str | auto | `"single"` o `"double"`. Auto: single si ancho ≤60cm, double si >60cm. |
| `kickplate_height` | int | `100` | Altura del zócalo en mm (10cm default). |
| `back_type` | str | `"full"` | Tipo de respaldo: `"full"` (panel MDF completo), `"rails"` (travesaños ventilación), `"none"` (sin respaldo). |
| `sections` | list[dict] | `null` | Secciones personalizadas. Cada sección: `{"width_cm": 80, "num_shelves": 3, "num_drawers": 1}`. Si `width_cm` es `"auto"`, distribuye espacio restante. **Tiene prioridad sobre `num_shelves` global.** |

**Notes:**
- Input dimensions are in cm, output dimensions in mm (manufacturing standard)
- Automatically adds vertical dividers when width exceeds material max span
- Back panels are always 3mm MDF
- Edge banding is auto-assigned by panel role
- Si se pasa `sections` y `num_shelves` al mismo tiempo, `sections` tiene prioridad (con warning)
- El zócalo se genera como marco rectangular completo (4 piezas): frente retranqueado 5cm, fondo, y 2 retornos laterales. Los laterales del mueble se apoyan encima de este marco.

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
- `compact` (bool, default: `true`): Return compact summary + JSON.

**Returns:** Compact summary + BOM JSON.

### `optimize_cuts`

Optimize panel cuts on standard sheets using guillotine algorithm.

**Parameters:**
- `parts` (list, required): Parts with `id`, `width` (mm), `height` (mm), `qty`, `can_rotate`, `grain`
  - `grain` per piece: `"length"` (grain along width — no rotation), `"width"` (grain along height — auto-rotates), `"none"` (free rotation)
- `sheet_width` (float, default: 2440): Sheet width in mm. Grain runs along this axis on the sheet.
- `sheet_height` (float, default: 1220): Sheet height in mm
- `blade_kerf` (float, default: 3): Saw blade width in mm
- `grain_direction` (string, default: `"auto"`): Global grain default. `"auto"` = per-piece, `"length"` = all pieces respect grain, `"none"` = free rotation.
- `compact` (bool, default: `true`): Return compact summary + JSON.

**Returns:** Compact summary + optimization JSON. Campos clave del resultado:
- `sheets_needed` (int): Cantidad de tableros necesarios
- `sheet_size_mm` (dict): `{width, height}` del tablero usado
- `waste_percentage` (float): Porcentaje de desperdicio
- `sheets[]`: Lista de tableros con `sheet_number` y `pieces[]` (cada pieza: `id`, `x`, `y`, `width`, `height`, `rotated`)
- `warnings` (list): Avisos de grano relajado u otros ajustes

**Notes:**
- Blade kerf (3mm default) is deducted from every cut — both between shelves and between pieces within a shelf.
- When `grain="length"`, pieces are locked to their original orientation to keep the grain pattern aligned horizontally on the sheet.
- For melamine with visible grain pattern, use `grain_direction="length"` to ensure consistent appearance across all pieces.
- Si una pieza no cabe con restricción de grano, el optimizer relaja automáticamente a `grain="none"` con warning.
- Si se pasan partes con `width_mm`/`height_mm` (del spec), se auto-convierten a `width`/`height` con warning.
- **Filtrar paneles MDF** antes de optimizar: excluir `back`, `drawer_bottom`, y cualquier panel con `thickness_mm` menor al material principal. Estos se cortan de otro stock (MDF 3mm).

**⚠ Schema spec vs. schema optimize_cuts (NO MEZCLAR):**

| Campo | En `spec.parts[]` | En `optimize_cuts.parts[]` |
|-------|-------------------|---------------------------|
| Ancho | `width_mm` | `width` |
| Alto | `height_mm` | `height` |
| Espesor | `thickness_mm` | *(no aplica)* |
| Posición | `position_mm` | *(no aplica)* |
| Cantidad | *(implícita: 1)* | `qty` (default: 1) |
| Grano | *(no aplica)* | `grain` (`"length"`, `"width"`, `"none"`) |
| Rotación | *(no aplica)* | `can_rotate` (bool) |

**Grain direction — referencia clara:**

| Valor | Significado | Rotación | Uso típico |
|-------|-------------|----------|------------|
| `"length"` | Grano corre a lo largo del `width` de la pieza | **Bloqueada** — la pieza no rota | Melamina con textura visible |
| `"width"` | Grano corre a lo largo del `height` de la pieza | **Auto-rota** para alinear con el tablero | Cuando la veta va en la otra dirección |
| `"none"` | Sin restricción de grano | **Libre** — rota para minimizar desperdicio | MDF, materiales sin veta, o cuando no importa |
| `"auto"` *(global)* | Usa el `grain` individual de cada pieza | Depende de la pieza | Default — la opción más flexible |

**Fórmulas de hardware (referencia):**

| Hardware | Fórmula | Ejemplo |
|----------|---------|---------|
| Confirmats por unión | 2 por esquina + 1 cada 30cm de largo | Panel 200cm ≈ 8 confirmats |
| Bisagras por puerta | 2 si puerta ≤ 100cm, 3 si ≤ 180cm, 4 si > 180cm | Puerta 220cm → 4 bisagras |
| Pernos de repisa | 4 por repisa ajustable | 5 repisas → 20 pernos |
| Correderas por cajón | 1 par (izq + der) | 2 cajones → 2 pares |

### `get_assembly_steps`

Generate step-by-step assembly instructions.

**Parameters:**
- `spec` (dict, required): A furniture spec
- `compact` (bool, default: `true`): Return compact summary + JSON.

**Returns:** Compact summary + assembly steps JSON. Drawer boxes have detailed 3-step assembly (slides → box → front).

---

## FreeCAD Bridge Tools

These tools execute directly in FreeCAD via XML-RPC (port 9875). They return a short summary — no scripts pass through the agent's context window.

### `build_3d_model`

Build the furniture as a 3D model directly in FreeCAD.

**Parameters:**
- `spec` (dict, required): A furniture spec
- `doc_name` (string, default: "Furniture"): FreeCAD document name

**Returns:** Summary of panels built.

**Panel colors by role:**
- Sides: warm wood (0.90, 0.75, 0.55)
- Bottom/Top: slightly darker (0.85, 0.70, 0.50)
- Shelves: green tint (0.70, 0.85, 0.65)
- Back: gray (0.75, 0.75, 0.75)
- Doors: blue tint (0.60, 0.75, 0.90)
- Drawer front: blue tint (0.60, 0.75, 0.90)
- Drawer sides/back/bottom: lighter blues
- Rails: dark brown (0.65, 0.55, 0.42)
- Kickplate: dark gray (0.50, 0.50, 0.50)
- Dividers: orange (0.90, 0.72, 0.40)

### `build_exploded_view`

Build an exploded assembly view directly in FreeCAD.

**Parameters:**
- `spec` (dict, required): A furniture spec
- `gap_mm` (float, default: 50): Gap between exploded parts in mm
- `doc_name` (string, default: "Exploded"): FreeCAD document name

**Returns:** Summary. Panels separate along their assembly axis.

### `build_cut_diagram`

Build a cut layout diagram directly in FreeCAD.

**Parameters:**
- `cut_result` (dict, required): Result from `optimize_cuts`
- `doc_name` (string, default: "CutLayout"): FreeCAD document name

**Returns:** Summary. Top-down view of sheets with placed panels.

### `build_techdraw`

Construye un plano técnico TechDraw directamente en FreeCAD con vistas ortogonales.

**Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `spec` | dict | (requerido) | Furniture spec como retorna `design_furniture` |
| `doc_name` | str | `"TechDraw"` | Nombre del documento FreeCAD |

**Returns:** Resumen de lo construido.

**Contenido del plano:**
- Página A3 landscape con template estándar
- Vista frontal (frente del mueble, plano XZ)
- Vista superior (plano XY)
- Vista lateral derecha (plano YZ)
- Auto-escalado para ajustarse al papel
- Anotaciones de dimensiones generales (ancho, alto, fondo)

**Notes:**
- El script es autocontenido — reconstruye la geometría como compound, no requiere que el modelo 3D ya exista
- El usuario puede agregar más cotas manualmente en FreeCAD después de generar el plano
- Ideal para documentación de fabricación y presentaciones a clientes

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

---

## Multi-Design Tools

### `create_design`

Create a new design project with its own directory.

**Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `name` | str | (requerido) | Nombre legible (ej: "Closet Dormitorio Principal") |
| `furniture_type` | str | (requerido) | Tipo de mueble (closet, kitchen_base, etc.) |

**Returns:** `design_id` (slug), ruta del directorio, URL si el servidor está activo.

**Notes:**
- El `design_id` es un slug generado del nombre (ej: "closet-dormitorio-principal")
- Si ya existe un diseño con el mismo slug, agrega un sufijo numérico
- Crea el directorio `./designs/{design_id}/` con `metadata.json`

### `list_designs`

List all active designs with metadata.

**Parameters:** Ninguno.

**Returns:** Lista de diseños con nombre, tipo, iteraciones, fecha de última actualización.

### `get_design_context`

Retrieve the full context of a design for resuming work in a new session.

**Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `design_id` | str | (requerido) | Identificador del diseño (slug) |

**Returns:** Metadata + spec compacto + spec JSON completo.

**Use case:** Al iniciar una nueva sesión, el agente puede recuperar todo el contexto de un diseño anterior para continuar iterando.

---

## Design Server

### `start_design_server`

Start the local HTTP server for serving design reports with live reload.

**Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `port` | int | `8432` | Puerto HTTP. Fallback a puerto aleatorio si está ocupado. |
| `designs_dir` | str | `"./designs"` | Directorio para archivos de diseño. |

**Returns:** URL del servidor y estado.

**Routes:**
```
GET /                          → Página índice con lista de diseños
GET /{design_id}               → Sirve report.html
GET /api/{design_id}/spec      → JSON spec del diseño
WS  /ws/{design_id}            → WebSocket para live reload
```

**Notes:**
- Solo necesita llamarse una vez por sesión — queda activo hasta que se cierra el MCP.
- El HTML generado por `update_design_report` incluye un cliente WebSocket que se conecta automáticamente.
- Si el puerto está ocupado, usa un puerto aleatorio y reporta la URL correcta.

### `get_section_map`

Get section labels from a furniture spec and optionally resolve natural language references.

**Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `design_id` | str | `""` | Cargar spec desde un diseño guardado |
| `spec` | str | `""` | JSON string de un spec (alternativa a design_id) |
| `resolve` | str | `""` | Texto en lenguaje natural para resolver a un section ID |

**Proveer `design_id` o `spec`** — uno de los dos es obligatorio.

**Returns:** Mapa de secciones con etiquetas, aliases, límites X, y partes por sección. Si `resolve` está presente, incluye el section ID resuelto.

**Ejemplo de resolución:**
- `resolve="izquierda"` → `{ "section_id": "S1", ... }`
- `resolve="centro"` → `{ "section_id": "S2", ... }`
- `resolve="S3"` → `{ "section_id": "S3", ... }` (pass-through)

**Section labels por cantidad:**
- 1 sección: "principal"
- 2 secciones: "izquierda", "derecha"
- 3 secciones: "izquierda", "centro", "derecha"
- 4+: "izquierda", "centro-izq", "centro-der", "derecha"

---

## Design Report Tools

### `update_design_report`

Generate or update an interactive HTML design report with 3D visualization.

**Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `spec` | dict | (requerido) | Furniture spec |
| `comment` | str | `""` | Descripción de los cambios en esta iteración |
| `iteration_name` | str | `""` | Nombre de la versión (auto: "v1", "v2"...) |
| `design_id` | str | `null` | Identificador de diseño (de `create_design`). **Recomendado** — usa DesignStore + live reload. |
| `output_path` | str | `null` | Ruta del HTML. Solo si `design_id` no está definido. Default: `./design_report.html` |
| `cut_data` | dict | `null` | Resultado de `optimize_cuts` (para renderizar layout de cortes) |

**Returns:** Ruta absoluta del archivo HTML generado.

**Features del reporte HTML:**
- Paleta clara minimalista (DM Sans, estilo ficha de producto)
- 4 páginas: Diseño (3D armado + resumen), Partes (explosión + tabla), Cortes (SVG por tablero), Historial
- Viewer 3D interactivo (Three.js v0.169) con orbit/zoom/pan
- Click en panel → muestra dimensiones, posición, canteado
- Canteado visual en tabla de partes (dots de color por borde)
- SVG de cortes con layout por tablero, barras de uso, tablas de piezas
- Slider de iteraciones para comparar versiones
- WebSocket live-reload (se actualiza solo al guardar nueva iteración)

**Comportamiento acumulativo:**
- Primera llamada → crea el HTML con v1
- Llamadas subsiguientes → lee iteraciones existentes del HTML, agrega nueva
- El historial completo se preserva en el archivo

---

## Development Tools

### `reload_engine`

Reload all engine modules to pick up code changes without restarting the MCP server.

**Parameters:** None.

**Returns:** List of reloaded modules.

**Use case:** After modifying engine files (designer.py, cut_optimizer.py, freecad_scripts.py, etc.) during development, call this instead of restarting the server.

---

## Furniture Spec Schema

### Estructura raíz

Campos requeridos:
- `furniture_type` (str): tipo de mueble (closet, kitchen_base, etc.)
- `parts` (list[dict]): lista de paneles — **NO usar `panels`**
- `dimensions_cm` (dict): `{width, height, depth}` en centímetros

Campos opcionales:
- `material` (str): clave del material (melamine_18, etc.)
- `material_thickness_mm` (float)
- `dimensions_mm` (dict): `{width, height, depth}` en mm
- `hardware` (dict o list): herrajes
- `notes` (list[str]): notas adicionales

### Estructura de cada part

Campos requeridos:
- `id` (str): identificador único — **NO usar `name`**
- `role` (str): rol del panel
- `width_mm` (float): ancho en mm — **NO usar `width`**
- `height_mm` (float): alto en mm — **NO usar `height`**
- `thickness_mm` (float): espesor en mm

Campos opcionales:
- `position_mm` (dict): `{x, y, z}` posición en mm — **NO usar `position`**
- `material` (str): material específico del panel
- `edge_banding` (list[str] | str): cantos
- `adjustable` (bool): si es repisa ajustable

### Roles válidos

| Role | Descripción |
|------|-------------|
| `side` | Lateral (izquierdo o derecho) |
| `bottom` | Piso del mueble |
| `top_panel` | Tapa superior |
| `floor` | Piso intermedio |
| `shelf` | Repisa |
| `back` | Respaldo (normalmente MDF 3mm) |
| `door` | Puerta |
| `rail` | Travesaño |
| `kickplate` | Zócalo (frente y fondo del marco base) |
| `kickplate_return` | Retorno lateral del zócalo (conecta frente con fondo) |
| `divider` | División vertical |
| `drawer_front` | Frente de cajón (canteado en 4 lados) |
| `drawer_side` | Lateral de cajón |
| `drawer_back` | Trasera de cajón |
| `drawer_bottom` | Fondo de cajón (MDF 3mm) |

### Errores comunes de formato

| Campo incorrecto | Campo correcto | Contexto |
|---|---|---|
| `panels` | `parts` | Nivel raíz del spec |
| `name` | `id` | Dentro de cada part |
| `width` | `width_mm` | Dentro de cada part (spec) |
| `height` | `height_mm` | Dentro de cada part (spec) |
| `thickness` | `thickness_mm` | Dentro de cada part (spec) |
| `position` | `position_mm` | Dentro de cada part |

> **Nota:** `optimize_cuts` usa un schema diferente: `width` y `height` (sin `_mm`) en mm. No confundir con el spec.

### Ejemplo mínimo válido

```json
{
  "furniture_type": "bookshelf",
  "dimensions_cm": {"width": 80, "height": 200, "depth": 30},
  "material": "melamine_18",
  "parts": [
    {"id": "side_left", "role": "side", "width_mm": 300, "height_mm": 2000, "thickness_mm": 18, "position_mm": {"x": 0, "y": 0, "z": 0}},
    {"id": "side_right", "role": "side", "width_mm": 300, "height_mm": 2000, "thickness_mm": 18, "position_mm": {"x": 782, "y": 0, "z": 0}},
    {"id": "top", "role": "top_panel", "width_mm": 764, "height_mm": 300, "thickness_mm": 18, "position_mm": {"x": 18, "y": 0, "z": 1982}},
    {"id": "bottom", "role": "bottom", "width_mm": 764, "height_mm": 300, "thickness_mm": 18, "position_mm": {"x": 18, "y": 0, "z": 0}},
    {"id": "back", "role": "back", "width_mm": 764, "height_mm": 2000, "thickness_mm": 3, "material": "mdf_3", "position_mm": {"x": 18, "y": 297, "z": 0}}
  ]
}
```
