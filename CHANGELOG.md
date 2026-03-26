# Changelog

Todos los cambios notables de este plugin se documentan en este archivo.

## [0.1.0] — 2026-03-26

Primera versión pública del plugin de diseño de muebles para Claude Code.

### Skill conversacional

- Protocolo de 4 fases: Descubrimiento -> Generación -> Iteración -> Exportación
- El agente pregunta secuencialmente (tipo, dimensiones, material, funcionalidad, distribución)
- Validación contra estándares después de cada respuesta del usuario
- Recomendaciones automáticas cuando hay conflictos estructurales

### Reglas anti-alucinación

- 10 reglas críticas documentadas en SKILL.md para prevenir errores comunes del agente
- Nunca saltar fase de Descubrimiento
- Nunca pasar `spec.parts` directo a `optimize_cuts` (schemas diferentes)
- Nunca construir spec manualmente — usar `design_furniture`
- Filtrar paneles MDF antes de optimizar cortes
- Zócalo es marco de 4 piezas, no panel suelto
- Cada iteración genera reporte actualizado

### Documentación

- `SKILL.md` — Protocolo conversacional completo con 4 fases y acciones especiales
- `reference.md` — Referencia de 23 tools MCP con parámetros, valores de retorno y schema del spec
- `examples.md` — 5 conversaciones ejemplo: flujo completo, rápido, retomado, consulta, importación FreeCAD

### Herramientas documentadas (23)

| Categoría | Cantidad |
|---|---|
| Diseño | 1 |
| Conocimiento | 5 |
| Validación y fabricación | 4 |
| Multi-diseño y reporte | 6 |
| FreeCAD exportar | 4 |
| FreeCAD importar | 2 |
| Desarrollo | 1 |
