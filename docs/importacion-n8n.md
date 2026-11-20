# Importación y configuración en n8n

## 1. Importar los flujos

Importa en este orden (el 1 y el 2 dependen del 0):

1. `workflows/0-herramientas-subworkflow.json`
2. `workflows/1-conversacion-evolution-api.json`
3. `workflows/2-motor-de-estados.json`

En n8n: **Workflows → Import from File** (o *Import from URL*).

## 2. Crear las credenciales

Los flujos referencian estas credenciales por su nombre. Créalas en n8n
(**Credentials → New**) y luego reasígnalas en cada nodo que las use:

| Credencial (nombre en los flujos) | Tipo de nodo | Para qué |
|---|---|---|
| `Google Service Account account` | Google API (Service Account) | Leer/escribir Sheets y Drive |
| `Evolution account` | Evolution API | Enviar/recibir WhatsApp |
| `OpenAI account` | OpenAI | Triage, agente y transcripción |

> La clave del Service Account de Google (archivo `*.json` que descargas de Google Cloud)
> **no** debe subirse al repositorio. Cárgala solo dentro de la credencial de n8n.

## 3. Conectar el sub-workflow

Si al importar el flujo **0** su ID cambió, abre los flujos **1** y **2** y corrige
en cada nodo `toolWorkflow` / `executeWorkflow` la referencia al sub-workflow
(`0 · Herramientas`). El ID original es `A4Pgkq5rbnBw26ef`.

## 4. Configurar el webhook de Evolution API

El flujo **1** expone un webhook en la ruta `unicau` (POST). Configura tu instancia
de Evolution API para enviar los eventos de mensajes a:

```
https://<tu-n8n>/webhook/unicau
```

## 5. Valores a reemplazar

Estos valores están **escritos directamente en los nodos** y deben ajustarse a tu
entorno. Para un despliegue limpio conviene moverlos a la hoja `Config` o a
variables de entorno de n8n.

| Valor actual | Dónde aparece | Qué es |
|---|---|---|
| `573234062106` | Nodos "WhatsApp a Natalia", alertas | WhatsApp de la encargada |
| `573180032204` | `Clasificar evento` (lista de ignorados) | Número del propio bot |
| `573108264135` | `Clasificar evento` (lista de ignorados) | Otro asistente a ignorar |
| `1Occlfl05QhW_...XxZ6g` | Nodos "Config", "Historial", "Reglas_pago" | ID de la hoja de control |
| `Erika` | Nodos de Evolution API (`instanceName`) | Nombre de la instancia de WhatsApp |
| `gpt-4o-mini` | Nodos de modelo OpenAI | Modelo de lenguaje |
| `America/Bogota` | Nodos de código | Zona horaria |

> **Antes de publicar el repo:** si va a ser público, reemplaza los números de
> teléfono reales por marcadores y documenta su significado. Son datos personales.

## 6. Hoja `Config` (Google Sheets)

El sistema resuelve la hoja de solicitudes del mes en tiempo de ejecución. En la
pestaña `Config` deben existir al menos:

| clave | valor |
|---|---|
| `solicitudes_id` | ID de la hoja de "Solicitudes" del mes vigente |
| `carpeta_raiz_id` | ID de la carpeta raíz de agendas en Drive |
| `telefono_encargada` | WhatsApp de la encargada (formato internacional sin `+`) |

Un Apps Script rota `solicitudes_id` cada mes.

## 7. Activar

Activa los 3 flujos. Verifica:

- El flujo **1** responde a un mensaje de prueba por WhatsApp.
- El flujo **2** (cron cada 2 min) aparece ejecutándose en *Executions*.
