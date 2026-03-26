# Usage Examples

## Example 1: Flujo completo — Closet con secciones

Muestra el protocolo completo: Descubrimiento → Generación → Iteración → Exportación.

**Conversación:**

> **User:** Quiero diseñar un closet
>
> **Agent:** ¡Perfecto! Vamos paso a paso. ¿Cuáles son las medidas del espacio donde irá el closet? (ancho × alto × fondo en cm)
>
> **User:** 180 de ancho, 240 de alto, 60 de fondo
>
> **Agent:** Buenas dimensiones. 180cm de ancho excede el tramo libre de melamina 16mm (75cm), así que el motor agregará divisores automáticamente. ¿Qué material prefieres?
> - **Melamina 16mm** (económica, tramo máx 75cm → 2 divisores)
> - **Melamina 18mm** (estándar, tramo máx 85cm → 1 divisor)
> - **MDF 18mm** (mejor acabado, tramo máx 90cm → 1 divisor)
>
> **User:** Melamina 18
>
> **Agent:** Con melamina 18mm solo necesitaremos 1 divisor (2 secciones). ¿Qué vas a almacenar? Esto define la distribución:
> - Sección izquierda: ¿repisas, cajones, barra de colgar?
> - Sección derecha: ¿igual o diferente?
>
> **User:** Izquierda para colgar ropa con cajones abajo, derecha repisas
>
> **Agent:** Entendido. Diseño propuesto:
> - Closet 180×240×60cm en melamina 18mm
> - Sección izquierda: barra de colgar + 3 cajones inferiores
> - Sección derecha: repisas
> - Zócalo de 10cm
>
> ¿Genero el diseño base?
>
> **User:** Sí

**Agent ejecuta (Fase B):**

```
1. start_design_server()
2. create_design("closet-dormitorio", "closet") → design_id = "closet-dormitorio"
3. design_furniture("closet", 180, 240, 60, "melamine_18", {
     "sections": [
       {"content": "drawers+hanging", "num_drawers": 3},
       {"content": "shelves"}
     ]
   })
4. validate_structure(spec)
5. optimize_cuts(spec.parts) → cut_data
6. update_design_report(design_id="closet-dormitorio", spec=spec, cut_data=cut_data,
     comment="Diseño inicial: closet 180x240x60 melamina 18mm")
```

**Agent responde:**

> Reporte listo en http://localhost:8432/closet-dormitorio
>
> **Resumen:**
> - 18 paneles: 2 laterales, 1 tapa, 1 fondo, 1 divisor, 1 zócalo, 3 repisas, barra cromada, 3 cajones (5 piezas c/u)
> - Validación: 0 errores, 0 warnings
> - 4 tableros de melamina 18mm (2440×1220), desperdicio: 38%
> - Barra cromada Ø25mm de 88cm

**Iteración (Fase C):**

> **User:** Agrega 2 repisas más a la derecha
>
> **Agent:** *(usa get_section_map para confirmar que "derecha" = S2, luego regenera)*

```
1. get_section_map(design_id="closet-dormitorio", resolve="derecha") → S2
2. design_furniture("closet", 180, 240, 60, "melamine_18", {
     "sections": [
       {"content": "drawers+hanging", "num_drawers": 3},
       {"content": "shelves", "num_shelves": 5}  // era 3, ahora 5
     ]
   })
3. validate_structure → optimize_cuts
4. update_design_report(design_id="closet-dormitorio", ..., comment="Agregadas 2 repisas en sección derecha (5 total)")
```

> Listo — el reporte se actualizó automáticamente en tu navegador. Ahora la sección derecha tiene 5 repisas. Puedes comparar con la versión anterior usando el slider de iteración.

---

## Example 2: Mueble de cocina rápido

Flujo más directo cuando el usuario da toda la información de entrada.

> **User:** Diseña un bajo de cocina de 80x90x60, melamina 16, con 2 cajones

**Agent (Fase A comprimida — toda la info ya está):**

> Entendido: bajo de cocina 80×90×60cm en melamina 16mm con 2 cajones. ¿Genero el diseño?

> **User:** Dale

**Agent ejecuta:**

```
1. start_design_server()
2. create_design("bajo-cocina-80", "kitchen_base")
3. design_furniture("kitchen_base", 80, 90, 60, "melamine_16", {"num_drawers": 2})
4. validate_structure(spec)
5. optimize_cuts(spec.parts) → cut_data
6. update_design_report(design_id="bajo-cocina-80", spec=spec, cut_data=cut_data,
     comment="Bajo de cocina 80x90x60 con 2 cajones")
```

> Reporte en http://localhost:8432/bajo-cocina-80
>
> - 13 paneles incluyendo 2 cajones completos (frente + caja)
> - Zócalo retranqueado 5cm
> - 2 tableros, 48% desperdicio (normal para un mueble individual)

---

## Example 3: Retomar un diseño existente

> **User:** ¿Qué diseños tengo guardados?
>
> **Agent:**

```
list_designs()
```

> Tienes 2 diseños:
> 1. **closet-dormitorio** — Closet, 3 iteraciones, última: hace 2 horas
> 2. **bajo-cocina-80** — Kitchen base, 1 iteración
>
> ¿Cuál quieres retomar?

> **User:** El closet

```
get_design_context("closet-dormitorio")
```

> Recuperado. Última versión (v3): closet 180×240×60 en melamina 18mm con sección izquierda (cajones + barra) y derecha (5 repisas). ¿Qué cambio necesitas?

> **User:** Cambia el material a MDF 18

```
1. design_furniture("closet", 180, 240, 60, "mdf_18", {mismas opciones})
2. validate_structure → optimize_cuts
3. update_design_report(design_id="closet-dormitorio", ..., comment="Cambio de material: melamina 18 → MDF 18")
```

> Actualizado. Con MDF 18mm el tramo máximo sube a 90cm (vs 85cm de melamina 18). El divisor sigue siendo necesario (90cm < 90cm de ancho interno). Desperdicio bajó a 36%.

---

## Example 4: Consulta de estándares y hardware

> **User:** ¿Qué tornillos uso para unir los laterales? ¿Necesito pegamento?

```
get_assembly_specs("panel_to_panel", brief=true)
```

> Para unión lateral-fondo (panel a panel):
> - **Confirmat 7×50mm**: pre-taladrar con broca de 5mm, 40mm profundidad. Mínimo 3 por unión.
> - **Tarugos 8×35mm**: con cola blanca D3 para refuerzo. No poner pegamento en los confirmats.
> - Espaciado: cada 20-25cm en juntas largas.

---

## Example 5: Importar y validar desde FreeCAD

> **User:** Modifiqué el mueble en FreeCAD, agregué una repisa. ¿Puedes validarlo?

```
1. build_import_script("Furniture")
2. mcp__freecad__execute_code(script)
3. parse_freecad_import(raw_output) → spec con 9 paneles
4. validate_structure(spec) → 0 errores, 1 warning
5. generate_bom(spec) → BOM actualizado
6. optimize_cuts(spec.parts) → 2 tableros
```

> Importé 9 paneles desde FreeCAD:
> - Validación: 0 errores, 1 warning (repisa de 82cm — considerar refuerzo)
> - BOM: 9 paneles, 5.2m de canteado
> - Cortes: 2 tableros, 54% desperdicio

---

## Section Content Options

Opciones disponibles para el campo `sections[].content` en `design_furniture`:

| Content | Descripción | Opciones adicionales |
|---------|-------------|---------------------|
| `shelves` | Repisas ajustables | `num_shelves` (default: auto por altura) |
| `drawers` | Solo cajones | `num_drawers`, `drawer_height` |
| `hanging` | Barra de colgar | `hanging_bar_height_cm` (default: 160) |
| `drawers+hanging` | Cajones abajo + barra arriba | `num_drawers`, `hanging_bar_height_cm` |
| `drawers+shelves` | Cajones abajo + repisas arriba | `num_drawers`, `num_shelves` |
| `empty` | Sin contenido | — |
