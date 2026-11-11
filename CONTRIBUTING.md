# Guía de contribución

Gracias por tu interés en mejorar el asistente de Salud Integral.

## Cómo trabajar con los flujos

1. **Exporta desde n8n**: abre el flujo → menú `⋯` → *Download* (JSON).
2. Guarda el archivo en [`workflows/`](workflows/) con su nombre correspondiente
   (ver la tabla del [README](README.md#flujos-de-trabajo)). Mantén los nombres
   estables para que los `diff` de git sean legibles.
3. Antes de hacer commit, **revisa el diff**: que no se cuelen secretos, tokens ni
   datos personales nuevos (ver abajo).

## Reglas al confirmar cambios (commits)

- No incluyas credenciales, claves de API ni archivos `.env`.
- No incluyas capturas con datos reales de estudiantes.
- Si cambias un mensaje que ve el estudiante, verifica el formato de WhatsApp
  (negritas con un solo `*`, emojis de la firma 💙❤️ con moderación).
- Usa mensajes de commit claros, en presente, p. ej.:
  `motor: corrige cierre de citas pasadas`.

## Estilo de ramas

- `main`: estado estable y desplegable.
- `feat/<descripción>`: nuevas funcionalidades.
- `fix/<descripción>`: correcciones.

## Revisión antes de publicar

Si el repositorio es público, confirma que los flujos están **sanitizados**
(números de teléfono, IDs de hoja, IDs de credenciales) según
[docs/importacion-n8n.md](docs/importacion-n8n.md).
