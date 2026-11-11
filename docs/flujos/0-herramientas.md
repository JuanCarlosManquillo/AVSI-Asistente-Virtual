# Flujo 0 · Herramientas (sub-workflow)

**Archivo:** [`../../workflows/0-herramientas-subworkflow.json`](../../workflows/0-herramientas-subworkflow.json)
**ID:** `A4Pgkq5rbnBw26ef`

Sub-workflow que concentra la **lógica de negocio**. No se activa solo: lo invocan
el flujo de conversación (como herramientas del agente) y el motor de estados.

## Entrada

Un `executeWorkflowTrigger` recibe un objeto con el campo `accion` y los datos
necesarios. Un nodo `switch` enruta según `accion`:

| `accion` | Qué hace |
|---|---|
| `registrar` | Valida y guarda una nueva solicitud de cita |
| `cancelar` | Cancela una solicitud del estudiante y avisa a la encargada |
| `escalar` | Notifica a la encargada para atención humana |
| `horarios` / `verificar` | Consulta horarios libres (o verifica uno antes de registrar) |
| `comprobante` | Procesa una foto/PDF de pago |
| `mis_solicitudes` | Devuelve las solicitudes del estudiante, ya resumidas |

## Piezas clave

- **Preparar solicitud**: valida nombre, código, documento, edad, programa, correo,
  servicio y tipo de atención. Recupera datos de la pestaña `Borradores` (hasta 48 h)
  para no volver a pedir lo ya entregado. Si algo falta, responde `ok: false` con la
  lista exacta de campos.
- **Consulta de horarios**: navega las carpetas de Drive (año → mes → servicio),
  descarga las agendas `.xlsx`, las interpreta leyendo el XML interno (celdas y
  celdas combinadas) y devuelve solo los horarios **LIBRES**, descontando los ya
  apartados por otras solicitudes activas.
- **Decidir registro**: revalida el horario y aplica las `Reglas_pago` para decidir
  si la cita queda `PENDIENTE`/`PENDIENTE_VALIDACION` y si requiere pago.
- **Historial**: cada acción deja traza en la pestaña `Historial` (auditoría).

## Salida

Devuelve al llamador un objeto con `ok` y un `mensaje` redactado para que el agente
lo comunique al estudiante (o `faltantes` / `opciones` cuando corresponde).
