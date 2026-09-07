# Plan de pruebas de estrés

| Test | Entrada | Resultado esperado | Evidencia |
|---|---|---|---|
| 1 | Idea válida + aprobación | Publicado + Log Éxito | Make + Airtable |
| 2 | Idea válida sin aprobación | Bloqueado por HITL | Filtro = 0 |
| 3 | Rechazo humano | Estado Rechazado + Log Rechazado | Airtable |
| 4 | Idea Semilla vacía | Error de validación, sin IA | Errores + Make |
| 5 | Fallo IA simulado | Retry/Break + Error + Slack | Error Handler |

No marcar un test como aprobado sin evidencia real.
