# furniture-designer-plugin

Plugin de Claude Code para diseño profesional de muebles. Agrega un skill de IA que entiende ingeniería de muebles, validación estructural, optimización de cortes, especificaciones de ensamble y visualización 3D en FreeCAD.

## Resumen ejecutivo

Con este plugin instalado, el agente de IA puede llevar un mueble desde la idea hasta un paquete listo para fabricación:

```
"Diseñame un closet de 120x240x60"
            ↓
┌─────────────────────────────────────────────────────────┐
│  El agente orquesta automáticamente 5-6 herramientas:   │
│                                                         │
│  1. design_furniture  → spec completa (paneles, pos.)   │
│  2. validate_structure → 0 errores, 0 warnings          │
│  3. generate_bom → paneles, herrajes, cantos            │
│  4. optimize_cuts → tableros, desperdicio, veta         │
│  5. get_assembly_steps → pasos con tornillería          │
│  6. build_3d_model → modelo 3D en FreeCAD (opcional)    │
└─────────────────────────────────────────────────────────┘
            ↓
Resumen: 14 paneles en melamine 16mm, 3 tableros 2440x1220,
         38% desperdicio, 12 pasos de ensamble, 0 errores.
```

### Qué obtiene el usuario

- **Especificación técnica** de cada panel: dimensiones en mm, posición XYZ, material, canto
- **Validación** contra 10 reglas de ingeniería (pandeo, anclaje, travesaños, respaldo, etc.)
- **Lista de materiales** agrupada: paneles + herrajes + cantos en metros lineales
- **Layout de corte** optimizado por tablero con respeto a dirección de veta y desbaste de sierra
- **Instrucciones de ensamble** paso a paso con tipo de fijación y cantidades
- **Especificaciones de ensamble** detalladas: qué tornillo, a qué profundidad taladrar, cuándo usar pegamento
- **Modelo 3D** en FreeCAD con componentes organizados por rol y propiedades de material (opcional)
- **Importación desde FreeCAD** — leer un modelo existente (editado o manual) y validarlo, generar BOM y cortes

### Consideraciones importantes

- Las **dimensiones de entrada** son en cm; las de salida en mm (estándar de fabricación)
- La **veta del material** se respeta en la optimización de cortes (`grain_direction="length"`) — importante para melaminas con textura visible
- El **desbaste de sierra** (kerf 3mm) se descuenta automáticamente en cada corte
- Los **paneles traseros** son siempre MDF 3mm — el validador genera error si faltan
- **Muebles anchos** (>75cm en melamine_16): se agrega división vertical automáticamente
- **Muebles altos** (>180cm): el validador advierte sobre anclaje a pared
- Las knowledge tools usan **modo `brief`** por defecto para ahorrar contexto (34-84% menos tokens)
- El modelo 3D en **FreeCAD** usa `App::Part` por panel, grupos por rol (`Estructura`, `Puertas`, `Repisas`, `Respaldo`), y propiedades custom (`Material`, `Thickness_mm`, `RealDimensions`, `EdgeBanding`)

## Instalación

```bash
claude plugin add https://github.com/LuisEnVilla/furniture-designer-plugin
```

Esto automáticamente:
1. Instala el servidor MCP `furniture-designer-mcp` (via uvx)
2. Registra el skill `/furniture-designer:furniture`

## Uso

### Invocación directa

```
/furniture-designer:furniture diseña un gabinete base de cocina 60x90x60
```

### O simplemente hablar

El agente invoca el skill automáticamente cuando detecta intención de diseño de muebles:

```
"Necesito un librero de 80cm de ancho, 200cm de alto, 30cm de fondo"
"¿Cuántos tableros necesito para un closet de 120x240x60?"
"¿Qué material aguanta un tramo de 100cm sin pandeo?"
"¿Qué tornillos uso para unir los laterales al piso?"
"Construye el mueble en FreeCAD"
```

## FreeCAD 3D (opcional)

Para modelos 3D, vistas explosionadas y diagramas de corte:

1. Instalar [FreeCAD](https://www.freecad.org/) (0.21+)
2. Instalar el addon MCP: `uvx freecad-mcp`
3. Abrir FreeCAD → workbench "MCP Addon" → "Start RPC Server"
4. Agregar al `.mcp.json` del proyecto:

```json
{
  "mcpServers": {
    "freecad": {
      "command": "uvx",
      "args": ["freecad-mcp", "--only-text-feedback"]
    }
  }
}
```

> `--only-text-feedback` elimina screenshots de las respuestas, ahorrando tokens. El agente solo pide screenshot cuando el usuario lo solicita explícitamente.

## Tipos de mueble

| Tipo | Descripción | Características clave |
|---|---|---|
| `kitchen_base` | Gabinete base de cocina | Zócalo, travesaños, sin tapa (para cubierta) |
| `kitchen_wall` | Gabinete aéreo | Sin zócalo, tapa superior e inferior |
| `closet` | Closet / armario | Zócalo, divisiones automáticas, alerta de anclaje si >180cm |
| `bookshelf` | Librero / estantería | Repisas múltiples, puertas opcionales |
| `desk` | Escritorio | Espacio para rodillas 60cm, panel de recato |
| `vanity` | Vanitorio / baño | Estructura tipo base, material resistente a humedad |

## Materiales

| Material | Espesor | Tramo máx. sin soporte | Mejor para |
|---|---|---|---|
| `melamine_16` (default) | 16mm | 75cm | Uso general, más económico |
| `melamine_18` | 18mm | 85cm | Cocinas y closets de gama alta |
| `mdf_15` | 15mm | 80cm | Muebles pintados |
| `mdf_18` | 18mm | 90cm | Puertas con diseño CNC |
| `plywood_18` | 18mm | 100cm | Tramos largos, alta resistencia |
| `solid_pine_20` | 20mm | 110cm | Muebles de madera maciza |

## Arquitectura

```
┌─────────────────────┐     ┌─────────────────────────────┐
│  furniture-designer  │     │  furniture-designer-mcp      │
│  -plugin (este repo) │────→│  (servidor MCP, repo aparte) │
│                      │     │                               │
│  • Skill SKILL.md    │     │  • 15 tools MCP               │
│  • .mcp.json         │     │  • Engine: designer, validator │
│  • reference.md      │     │  • Knowledge: materials, rules │
│  • examples.md       │     │  • FreeCAD bridge scripts      │
└─────────────────────┘     └──────────────┬────────────────┘
                                           │ (scripts Python)
                                           ↓
                            ┌─────────────────────────────┐
                            │  FreeCAD (opcional)          │
                            │  via freecad-mcp + XML-RPC   │
                            │  puerto 9875                 │
                            └─────────────────────────────┘
```

- **Este plugin** — capa de UX: enseña al agente de IA cómo orquestar las herramientas
- **[furniture-designer-mcp](https://github.com/LuisEnVilla/furniture-designer-mcp)** — motor: 13 tools MCP que funcionan con cualquier cliente MCP
- **freecad-mcp** (opcional) — puente para ejecutar scripts Python en FreeCAD

## Licencia

MIT
