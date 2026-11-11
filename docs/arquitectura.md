# Arquitectura

El sistema son **tres flujos de n8n** que se apoyan en Google Sheets/Drive como
base de datos y en Evolution API como canal de WhatsApp.

## Componentes

| Componente | Tecnología | Función |
|---|---|---|
| Canal de mensajería | Evolution API (WhatsApp) | Recibe y envía mensajes |
| Orquestación | n8n | Ejecuta los 3 flujos |
| IA | OpenAI (`gpt-4o-mini`, Whisper) | Triage de intención/riesgo, agente conversacional, transcripción de audios |
| Datos | Google Sheets | Solicitudes, Historial, Config, Reglas_pago, Borradores |
| Agendas | Google Drive (.xlsx) | Horarios de los profesionales por mes/servicio |

## Flujo de una conversación

1. El estudiante escribe por WhatsApp → **Evolution API** dispara el webhook del
   flujo **[1] Conversación**.
2. Se clasifica el evento (mensaje del estudiante, toma de control humana, o se ignora).
3. Si es texto o audio, se transcribe (audio) y pasa por **triage** (intención + señal de riesgo).
4. El **agente IA** responde y, cuando necesita datos o acciones, llama a las
   herramientas del sub-workflow **[0] Herramientas**:
   - `consultar_mis_solicitudes`, `consultar_horarios`, `registrar_solicitud`,
     `solicitar_cancelacion`, `escalar_a_humano`.
5. El sub-workflow lee/escribe en Google Sheets y lee las agendas desde Drive.
6. En paralelo, el flujo **[2] Motor de estados** corre cada 2 minutos y envía
   confirmaciones, recordatorios y avisos de pago/estado, y cierra citas pasadas.

## Modelo de datos (Google Sheets)

- **Solicitudes** (una hoja por mes, su ID vive en `Config → solicitudes_id`):
  una fila por solicitud de cita, con su `estado`, `estado_pago` y banderas de
  notificación (`confirmacion_enviada`, `recordatorio_enviado`, …).
- **Config**: pares clave/valor (p. ej. `solicitudes_id`, `carpeta_raiz_id`).
- **Historial**: bitácora de cambios de estado (auditoría).
- **Reglas_pago**: define si un servicio/tipo de atención tiene costo.
- **Borradores**: datos parciales de una solicitud para no volver a pedirlos si algo falla.

## Máquina de estados de una solicitud

```
PENDIENTE / PENDIENTE_VALIDACION
        │  (Salud Integral asigna fecha/hora/profesional)
        ▼
   CONFIRMADA ──(la cita pasa)──► CERRADA
        │
        ├─ CANCELACION_SOLICITADA ──► CANCELADA
        └─ RECHAZADA
```

El pago sigue su propia secuencia:
`NO_APLICA | POR_CONFIRMAR → PENDIENTE_RECIBO → RECIBO_ENVIADO →
COMPROBANTE_RECIBIDO → PAGO_VALIDADO | COMPROBANTE_RECHAZADO`.

Una cita con pago obligatorio **no se confirma** hasta que el pago esté `PAGO_VALIDADO`.

## Decisiones de diseño

- **Separación en 3 flujos**: la conversación (tiempo real) se mantiene ligera; la
  lógica pesada vive en el sub-workflow; las tareas periódicas en el motor de estados.
- **Google Sheets como base de datos**: permite que la encargada de Salud Integral
  gestione las citas con una herramienta que ya conoce, sin un panel aparte.
- **Red de seguridad determinística**: ante fallos de OpenAI, el estudiante recibe
  un mensaje fijo; las señales de riesgo siempre incluyen las líneas 123 y 192.
