# UnicaU · Asistente Virtual de Salud Integral

Asistente conversacional por **WhatsApp** para el área de **Salud Integral** de la
Universidad del Cauca. Permite a los estudiantes pedir, consultar y cancelar citas
de **psicología, medicina y odontología**, con apoyo de IA, y automatiza las
confirmaciones, recordatorios y la gestión de pagos.

Construido con [n8n](https://n8n.io) (automatización), [Evolution API](https://doc.evolution-api.com)
(WhatsApp), OpenAI (triage + agente conversacional) y Google Sheets / Drive como fuente de datos.

> ⚠️ **Proyecto piloto.** Maneja datos personales de estudiantes. Lee la sección
> [Privacidad y seguridad](#privacidad-y-seguridad) antes de desplegarlo o publicarlo.

---

## Tabla de contenido

- [Arquitectura](#arquitectura)
- [Flujos de trabajo](#flujos-de-trabajo)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Privacidad y seguridad](#privacidad-y-seguridad)
- [Documentación](#documentación)
- [Licencia](#licencia)

---

## Arquitectura

```
Estudiante (WhatsApp)
        │
        ▼
 Evolution API ──► [1] Conversación ──► Agente IA (OpenAI)
                        │                    │
                        │                    ├─ consultar_mis_solicitudes
                        │                    ├─ consultar_horarios
                        │                    ├─ registrar_solicitud      ──► [0] Herramientas
                        │                    ├─ solicitar_cancelacion         (sub-workflow)
                        │                    └─ escalar_a_humano                   │
                        │                                                          ▼
                        │                                              Google Sheets (Solicitudes,
                        │                                              Historial, Config, Reglas_pago,
                        │                                              Borradores) + Google Drive (agendas)
                        ▼
 [2] Motor de estados (cron cada 2 min)
     confirmaciones · recordatorios · avisos de pago · cierres automáticos
```

Diagrama detallado en [docs/arquitectura.md](docs/arquitectura.md).

## Flujos de trabajo

Los 3 flujos de n8n viven en [`workflows/`](workflows/):

| Archivo | Nombre en n8n | Rol |
|---|---|---|
| [`0-herramientas-subworkflow.json`](workflows/0-herramientas-subworkflow.json) | `0 · Herramientas (sub-workflow)` | Lógica de negocio: registrar/consultar/cancelar citas, horarios, pagos |
| [`1-conversacion-evolution-api.json`](workflows/1-conversacion-evolution-api.json) | `1 · Conversación (Evolution API)` | Webhook de WhatsApp, triage de intención/riesgo y agente conversacional |
| [`2-motor-de-estados.json`](workflows/2-motor-de-estados.json) | `2 · Motor de estados` | Cron: confirmaciones, recordatorios, avisos de estado y pago |

Cada flujo está documentado en [`docs/flujos/`](docs/flujos/).

## Requisitos

- **n8n** (self-hosted o cloud) con los nodos community:
  - `n8n-nodes-evolution-api`
  - `@n8n/n8n-nodes-langchain` (incluido en n8n reciente)
- **Evolution API** con una instancia de WhatsApp conectada.
- Cuenta de **OpenAI** con acceso a `gpt-4o-mini` y transcripción de audio (Whisper).
- **Google Service Account** con acceso a las hojas y carpetas de Drive.

## Instalación

1. Clona el repositorio.
2. Importa los 3 flujos de [`workflows/`](workflows/) en tu instancia de n8n
   (ver [docs/importacion-n8n.md](docs/importacion-n8n.md)).
3. Crea las credenciales en n8n (Google, Evolution, OpenAI) y vincúlalas a los nodos.
4. Copia `.env.example` a `.env` y completa los valores (ver [Configuración](#configuración)).
5. Activa los flujos.

Guía paso a paso en [docs/importacion-n8n.md](docs/importacion-n8n.md).

## Configuración

Los valores sensibles están documentados en [`.env.example`](.env.example). Incluyen:

- IDs de hojas de Google (Config, Solicitudes, Historial…)
- Nombre de la instancia de Evolution API
- Números de WhatsApp (encargada, bot)
- IDs de credenciales de n8n

> Los flujos actuales tienen varios de estos valores **escritos directamente en los nodos**.
> Antes de reutilizarlos en otro entorno, revisa [docs/importacion-n8n.md](docs/importacion-n8n.md),
> sección *Valores a reemplazar*.

## Privacidad y seguridad

Este asistente trata **datos personales de salud** (nombre, documento, servicio solicitado)
y debe cumplir la política de tratamiento de datos de la Universidad del Cauca.

- **No** subas al repositorio credenciales, tokens ni claves de servicio.
  El archivo [`.gitignore`](.gitignore) ya excluye `.env` y archivos de credenciales.
- Los exports de n8n incluyen **IDs de credenciales** (no los secretos) y **números de
  teléfono reales** incrustados en los nodos. Si el repositorio va a ser **público**,
  sanitízalos primero (ver [docs/importacion-n8n.md](docs/importacion-n8n.md)).
- Los flujos no guardan información clínica (síntomas, diagnósticos): solo los datos
  mínimos para gestionar la cita.

## Documentación

- [Arquitectura](docs/arquitectura.md)
- [Importación y configuración en n8n](docs/importacion-n8n.md)
- [Flujo 0 · Herramientas](docs/flujos/0-herramientas.md)
- [Flujo 1 · Conversación](docs/flujos/1-conversacion.md)
- [Flujo 2 · Motor de estados](docs/flujos/2-motor-de-estados.md)

## Licencia

Ver [LICENSE](LICENSE).
