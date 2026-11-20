# Flujo 1 · Conversación (Evolution API)

**Archivo:** [`../../workflows/1-conversacion-evolution-api.json`](../../workflows/1-conversacion-evolution-api.json)
**ID:** `L3mbByNnsEAE5FrZ`

Flujo de **tiempo real** disparado por cada mensaje de WhatsApp. Clasifica, entiende
y responde, apoyándose en el sub-workflow [0 · Herramientas](0-herramientas.md).

## Entrada

Webhook `POST /webhook/unicau` alimentado por Evolution API.

## Recorrido

1. **Clasificar evento**: distingue mensaje de estudiante, toma de control humana
   (mensaje escrito desde el número del bot) o eventos a ignorar (grupos, estados,
   números del propio sistema). Incluye frenos anti-bucle (límite de mensajes por minuto).
2. **Estado del chat**: si una persona tomó el control del chat, el bot se pausa
   (hasta 12 h o hasta recibir `ACTIVAR BOT`).
3. **Tipo de mensaje**: texto, audio (se transcribe con Whisper), o imagen/documento
   (posible comprobante de pago → va al sub-workflow).
4. **Triage**: un LLM barato (`gpt-4o-mini`, temperatura 0) clasifica intención y
   detecta **señal de riesgo** (autolesión/suicidio/emergencia).
5. **Agente AVSI**: agente conversacional con memoria por teléfono y 5 herramientas
   (`consultar_mis_solicitudes`, `consultar_horarios`, `registrar_solicitud`,
   `solicitar_cancelacion`, `escalar_a_humano`).
6. **Red de seguridad**: convierte Markdown a formato WhatsApp y garantiza que los
   mensajes de riesgo siempre incluyan las líneas **123** y **192 opción 4**.
7. **Responder por WhatsApp** vía Evolution API.

## Protocolo de riesgo

Si el triage marca riesgo, se envía en paralelo una **alerta a la encargada** y la
respuesta al estudiante prioriza el acompañamiento y las líneas de ayuda.

## Tolerancia a fallos

Si OpenAI falla (triage o agente), el nodo `¿Avisar falla?` envía un mensaje fijo
de respaldo (máximo uno cada 10 min por chat) para que el estudiante nunca quede sin
respuesta.
