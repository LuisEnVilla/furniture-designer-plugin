# furniture-designer-plugin

> **v0.1.0** — Plugin de Claude Code para diseño profesional de muebles

Plugin que agrega a Claude Code un asistente especializado en diseño de muebles de melamina y madera. El agente guía al usuario conversacionalmente desde la idea hasta un paquete listo para fabricación, con reporte HTML interactivo, validación estructural, optimización de cortes y exportación 3D opcional.

<img width="1200" alt="reporte" src="https://github.com/user-attachments/assets/07470ff4-27d4-4e02-9a2a-cb771c515384" />


## Que hace

Con este plugin instalado, puedes hablar naturalmente sobre muebles y el agente orquesta todo automáticamente:

```
Tu: "Quiero disenar un closet de 180x240x60 en melamina 18,
     con cajones a la izquierda y repisas a la derecha"

Agente:
  1. Valida dimensiones contra estandares ergonomicos
  2. Genera spec completa (18 paneles, posiciones, hardware)
  3. Valida estructura (10 reglas de ingenieria)
  4. Optimiza cortes (3 tableros, 32% desperdicio)
  5. Genera reporte HTML interactivo

  "Reporte listo en http://localhost:8432/closet-dormitorio"

Tu: "Haz la seccion izquierda mas ancha"

Agente:
  → Resuelve "izquierda" -> S1
  → Regenera spec con ajuste
  → El reporte se actualiza solo en el navegador (WebSocket)
```

### Que obtiene el usuario

| Entregable | Descripcion |
|---|---|
| **Reporte HTML interactivo** | Viewer 3D (Three.js), tabla de partes, layout de cortes SVG, historial de iteraciones — se actualiza en vivo |
| **Especificacion tecnica** | Cada panel con dimensiones mm, posición XYZ, material, canteado |
| **Validacion estructural** | 10 reglas de ingeniería: pandeo, anclaje, travesaños, respaldo, zócalo |
| **Lista de materiales** | Paneles agrupados + herrajes + cantos en metros lineales |
| **Optimizacion de cortes** | Layout por tablero con veta, kerf, y % desperdicio |
| **Instrucciones de ensamble** | Pasos ordenados con tipo de fijación y cantidades |
| **Especificaciones de ensamble** | Qué tornillo, a qué profundidad, cuándo usar pegamento |
| **Modelo 3D en FreeCAD** | Componentes organizados por rol con propiedades de material (opcional) |
| **Vista explosionada** | Separación por eje de ensamble en FreeCAD (opcional) |
| **Plano tecnico 2D** | Vistas ortogonales auto-escaladas a A3 en FreeCAD (opcional) |
| **Diagrama de cortes** | Layout visual de tableros en FreeCAD (opcional) |
| **Importacion desde FreeCAD** | Leer un modelo existente, validarlo, generar BOM y cortes (opcional) |

<img width="1200" alt="Reporte HTML — pagina de diseno con viewer 3D" src="https://github.com/user-attachments/assets/8e4e669f-b827-40fa-bf1e-ae96f6f597c1" />

<img width="1200" alt="Reporte HTML" src="https://github.com/user-attachments/assets/1740adc6-2980-42f0-a3c5-e62a776960c0" />


## Instalacion

### 1. Instalar el plugin

```bash
claude plugin add https://github.com/LuisEnVilla/furniture-designer-plugin
```

Esto automaticamente:
1. Registra el servidor MCP `furniture-designer-mcp` (via uvx)
2. Registra el skill `/furniture` para invocación directa

### 2. Verificar

```bash
claude
> /furniture diseña un librero de 80x200x30
```

Si el agente responde con preguntas sobre el diseño, el plugin esta funcionando correctamente.

### 3. FreeCAD (opcional)

Solo necesario si quieres modelos 3D, vistas explosionadas o planos técnicos:

1. Instalar [FreeCAD](https://www.freecad.org/) (0.21+)
2. Instalar el addon MCP: `uvx freecad-mcp`
3. Abrir FreeCAD -> workbench "MCP Addon" -> "Start RPC Server"
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

> `--only-text-feedback` elimina screenshots de las respuestas, ahorrando tokens.

## Uso

### Invocacion directa (skill)

```
/furniture diseña un gabinete base de cocina 60x90x60
```

### Conversacion natural

El agente detecta automaticamente la intención de diseño de muebles:

```
"Necesito un librero de 80x200x30"
"Cuantos tableros necesito para un closet de 120x240x60?"
"Que material aguanta un tramo de 100cm sin pandeo?"
"Que tornillos uso para unir los laterales al piso?"
"Agrega un cajon a la seccion derecha"
"Construye el modelo en FreeCAD"
```

### Protocolo de 4 fases

El agente sigue un flujo conversacional estructurado:

#### Fase A: Descubrimiento

El agente pregunta secuencialmente (no todo de golpe):
1. Tipo de mueble
2. Dimensiones del espacio
3. Material preferido
4. Que se almacenara
5. Distribución de secciones

Después de cada respuesta valida contra estándares y recomienda si hay conflictos.

#### Fase B: Generacion

Cuando el usuario confirma, el agente:
1. Crea el proyecto de diseño
2. Genera la especificación completa
3. Valida estructura
4. Optimiza cortes
5. Genera el reporte HTML interactivo
6. Proporciona la URL del reporte

<img width="600" alt="reporte generado con resumen del agente" src="https://github.com/user-attachments/assets/d6761835-1262-47d8-989f-b9e2e9c2b1a6" />

<img width="600" alt="reporte generado con resumen del agente" src="https://github.com/user-attachments/assets/464b4db1-eec3-4255-83f2-f651a95ecb61" />


#### Fase C: Iteracion

El usuario pide cambios con lenguaje natural:
- "Haz la izquierda mas ancha"
- "Agrega un cajon al centro"
- "Cambia el material a melamina 18"

El reporte se actualiza automaticamente en el navegador via WebSocket.

<!-- ![Fase C — reporte actualizado despues de iteracion](docs/screenshots/iteration-live-reload.png) -->

#### Fase D: Exportacion

Cuando el diseño esta listo:
- "Generamelo en FreeCAD" -> modelo 3D
- "Necesito el plano tecnico" -> TechDraw A3
- "Dame la lista de compra final" -> BOM detallado

<!-- ![Fase D — modelo 3D en FreeCAD](docs/screenshots/freecad-3d-model.png) -->

<!-- ![Fase D — plano tecnico TechDraw](docs/screenshots/freecad-techdraw.png) -->

## Tipos de mueble

| Tipo | Descripcion | Caracteristicas clave |
|---|---|---|
| `kitchen_base` | Gabinete base de cocina | Zócalo 4 piezas, travesaños, sin tapa (para cubierta) |
| `kitchen_wall` | Gabinete aereo | Sin zócalo, tapa superior e inferior |
| `closet` | Closet / armario | Zócalo, divisiones auto si >75cm, alerta anclaje si >180cm |
| `bookshelf` | Librero / estanteria | Repisas multiples, puertas opcionales |
| `desk` | Escritorio | Espacio para rodillas 60cm, panel de recato |
| `vanity` | Vanitorio / bano | Estructura tipo base, material resistente a humedad |

## Materiales

| Material | Espesor | Tramo max. | Mejor para |
|---|---|---|---|
| `melamine_16` (default) | 16mm | 75cm | Uso general, mas economico |
| `melamine_18` | 18mm | 85cm | Cocinas y closets de gama alta |
| `mdf_15` | 15mm | 80cm | Muebles pintados |
| `mdf_18` | 18mm | 90cm | Puertas con diseno CNC |
| `plywood_18` | 18mm | 100cm | Tramos largos, alta resistencia |
| `solid_pine_20` | 20mm | 120cm | Muebles de madera maciza |

## Consideraciones importantes

- Las **dimensiones de entrada** son en cm; las de salida en mm (estándar de fabricación)
- La **veta del material** se respeta en la optimización de cortes — importante para melaminas con textura
- El **desbaste de sierra** (kerf 3mm) se descuenta automáticamente en cada corte
- Los **paneles traseros** son siempre MDF 3mm — se excluyen del optimizador de cortes de melamina
- El **zócalo** es un marco de 4 piezas (frente, fondo, 2 retornos laterales) — no un panel suelto
- **Muebles anchos** (>75cm en melamine_16): se agrega división vertical automáticamente
- **Muebles altos** (>180cm): el validador advierte sobre anclaje a pared
- **Cajones completos**: caja de 5 piezas + correderas telescópicas
- El agente usa **modo brief** por defecto en knowledge tools (ahorro 60-80% tokens)
- El agente usa **modo compact** por defecto en design tools (resumen + JSON reducido)

## Arquitectura

```
+----------------------------+     +------------------------------------+
|  furniture-designer-plugin |     |  furniture-designer-mcp            |
|  (este repo)               |---->|  (servidor MCP, repo aparte)       |
|                            |     |                                    |
|  Que hacer y cuando        |     |  Como hacerlo                      |
|  -------------------------  |     |  ----------------------------       |
|  - Flujo conversacional    |     |  - 23 tools MCP                    |
|  - Cuando llamar cada tool |     |  - Engine: designer, validator     |
|  - Como interpretar datos  |     |  - Knowledge: materials, rules     |
|  - Reglas anti-alucinacion |     |  - HTTP server + WebSocket         |
|  - Ejemplos de conversacion|     |  - Multi-diseno + persistencia     |
|  - Referencia de tools     |     |  - Reporte HTML interactivo        |
+----------------------------+     |  - Section mapper                  |
                                   |  - FreeCAD bridge scripts          |
                                   +----------------+-------------------+
                                                    |
                                                    | (XML-RPC, opcional)
                                                    v
                                   +------------------------------------+
                                   |  FreeCAD (opcional)                 |
                                   |  via freecad-mcp + XML-RPC          |
                                   |  puerto 9875                       |
                                   +------------------------------------+
```

- **Este plugin** — capa de UX: enseña al agente como orquestar las 23 herramientas
- **[furniture-designer-mcp](https://github.com/LuisEnVilla/furniture-designer-mcp)** — motor: servidor MCP que funciona con cualquier cliente MCP
- **freecad-mcp** (opcional) — puente para ejecutar scripts Python en FreeCAD

## Estructura del plugin

```
furniture-designer-plugin/
  .claude-plugin/
    plugin.json            <- Metadata del plugin (nombre, version, autor)
  .mcp.json                <- Configuracion del servidor MCP
  skills/
    furniture/
      SKILL.md             <- Protocolo conversacional de 4 fases + reglas
      reference.md         <- Referencia completa de 23 tools con schemas
      examples.md          <- 5 conversaciones ejemplo
  README.md
  LICENSE
```

## Herramientas disponibles (23)

El plugin expone 23 herramientas MCP organizadas en 7 categorías:

| Categoria | Tools | Cantidad |
|---|---|---|
| Diseño | `design_furniture` | 1 |
| Conocimiento | `get_standards`, `get_material_specs`, `get_structural_rules`, `get_hardware_catalog`, `get_assembly_specs` | 5 |
| Validación y fabricación | `validate_structure`, `generate_bom`, `optimize_cuts`, `get_assembly_steps` | 4 |
| Multi-diseño y reporte | `create_design`, `list_designs`, `get_design_context`, `get_section_map`, `start_design_server`, `update_design_report` | 6 |
| FreeCAD exportar | `build_3d_model`, `build_exploded_view`, `build_cut_diagram`, `build_techdraw` | 4 |
| FreeCAD importar | `build_import_script`, `parse_freecad_import` | 2 |
| Desarrollo | `reload_engine` | 1 |

Ver [reference.md](skills/furniture/reference.md) para la documentación completa de cada herramienta con parámetros y valores de retorno.

## Licencia

MIT
