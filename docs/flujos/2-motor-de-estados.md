# Flujo 2 · Motor de estados

**Archivo:** [`../../workflows/2-motor-de-estados.json`](../../workflows/2-motor-de-estados.json)
**ID:** `5krnOYkTXjpIqndP`

Flujo **periódico** (cron cada 2 minutos) que revisa la hoja `Solicitudes` y envía
las notificaciones que correspondan. Procesa **una acción por fila por ejecución**
para cuidar los límites de escritura de Google.

## Recorrido

1. **Cada 2 minutos** (`scheduleTrigger`) → lee la hoja del mes (vía `Config`).
2. **Clasificar acciones**: por cada fila decide qué acción toca, según su `estado`,
   `estado_pago` y banderas de notificación.
3. Un `switch` reparte a la rama correspondiente y, tras enviar el WhatsApp, marca
   la bandera para no repetir.

## Acciones

| Acción | Cuándo | Resultado |
|---|---|---|
| `notificar_nueva` | Solicitud nueva sin avisar | Reintento de aviso a la encargada |
| `notificar_cancelacion` | Cancelación sin avisar | Reintento de aviso a la encargada |
| `confirmar_estudiante` | Cita `CONFIRMADA` sin confirmación enviada | WhatsApp de confirmación + enlace a calendario |
| `recordatorio` | Cita confirmada dentro de 24 h | WhatsApp de recordatorio |
| `aviso_pago` | Cambio en `estado_pago` | Avisa recibo/validación/rechazo |
| `aviso_estado` | `RECHAZADA`/`CANCELADA` por avisar | Informa al estudiante |
| `aviso_bloqueo` | La encargada bloqueó la franja | Informa que se reasignará |
| `cerrar` | La cita ya pasó (día siguiente) | Marca `CERRADA` + historial |

## Notas de diseño

- Las confirmaciones y recordatorios indican **qué profesional** asignó Salud Integral.
- Nunca se confirma ni recuerda una cita cuya fecha ya pasó.
- Una cita con pago obligatorio no se confirma hasta `PAGO_VALIDADO`.
- Antes de cerrar una cita, **relee** la hoja por si cambió mientras tanto.
