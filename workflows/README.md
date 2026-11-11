# Flujos de n8n

Exportaciones JSON de los flujos de n8n del asistente. Impórtalos en tu instancia
siguiendo [../docs/importacion-n8n.md](../docs/importacion-n8n.md).

| Archivo | Nombre en n8n | ID de workflow |
|---|---|---|
| `0-herramientas-subworkflow.json` | `0 · Herramientas (sub-workflow)` | `A4Pgkq5rbnBw26ef` |
| `1-conversacion-evolution-api.json` | `1 · Conversación (Evolution API)` | `L3mbByNnsEAE5FrZ` |
| `2-motor-de-estados.json` | `2 · Motor de estados` | `5krnOYkTXjpIqndP` |

> El flujo **1** y el **2** invocan al sub-workflow **0** por su ID. Si al importar
> cambia el ID, actualiza las referencias (ver guía de importación).

## Actualizar un flujo

1. Edita en n8n.
2. Exporta: menú `⋯` → *Download*.
3. Reemplaza el archivo correspondiente aquí, **conservando el nombre**.
4. Revisa el `git diff` antes de confirmar (que no se cuelen secretos ni datos reales).
