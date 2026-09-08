# Plan y resultados de pruebas

| Test | Entrada | Resultado esperado | Resultado real |
|---|---|---|---|
| 1 | Idea válida | Generación RAG + `En revisión` | PASS — 7 módulos, 0 errores |
| 2 | Aprobación humana | `Publicado` + `Resultado final` + Log | PASS — 4 módulos, 0 errores |
| 3 | Rechazo humano | Slack + Log, sin publicación | PASS |
| 4 | Idea Semilla vacía | Error de validación, sin IA | PASS |
| 5 | Error de publicación simulado | Error Handler + Airtable + Slack | PASS |
| 6 | Modificar rechazo ya procesado | No reprocesar | PASS — solo ejecutó el trigger |
| 7 | Modificar dato incompleto ya registrado | No reprocesar | PASS — solo ejecutó el trigger |

## Resiliencia configurada

- Generación IA: Error Handler + Airtable + Slack + Retry/Break de 3 intentos, 1 minuto.
- Publicación: Error Handler + Airtable + Slack + Retry/Break de 3 intentos, 1 minuto.

## Evidencia de consumo observado

- Generación RAG: 8.03 créditos.
- Aprobación: 4 créditos.
- Rechazo + dato incompleto en una misma corrida: 5 créditos.
- Error de publicación simulado: 8 créditos.
- Validación anti-loop final: 1 crédito (solo trigger).

