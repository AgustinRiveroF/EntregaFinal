# Cambios requeridos en Make

## Escenario 1
- Validar Idea Semilla no vacía antes del RAG.
- En datos inválidos: crear registro en Errores y finalizar.
- Error Handler en OpenAI: Retry + Break; registrar Errores y Log con Tiene error=true; avisar por Slack.
- Después de En revisión: enviar aviso Slack y crear Log de éxito.
- Mantener filtro anti-loop Estado=Generando.

## Escenario 2
- Mantener Aprobado=true AND Estado=En revisión.
- Tras aprobar: enviar salida Slack, actualizar Publicado, Resultado final y fecha, crear Log.
- Ruta Rechazado: registrar Log Rechazado y no publicar.
- Error Handler en Slack y Airtable Update.
