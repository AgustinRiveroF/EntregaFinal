# Validación runtime final

Fecha de validación: 2026-09-07 / 2026-09-08 (America/Cordoba).

Escenario validado: `FINAL - Pipeline IA RAG + HITL + Resiliencia`.

## 1. Generación RAG

Caso: registro nuevo con `Estado = Generando` e `Idea Semilla` válida.

Resultado observado:
- Airtable Watch Records: OK
- Buscar conocimiento: OK
- Text Aggregator: OK
- GPT-5 nano: OK
- Update a `En revisión`: OK
- Slack de revisión: OK
- Log de generación: OK

Ejecución: 7 módulos, 0 errores, 8.03 créditos.

El registro terminó con:
- `Estado = En revisión`
- `Borrador IA` completo
- `Aprobado` sin marcar
- sin `Resultado final`

Esto confirma que la IA genera pero no publica.

## 2. Aprobación HITL

Caso: el mismo registro fue marcado manualmente con `Aprobado = true` manteniendo `Estado = En revisión`.

Resultado observado:
- actualización a `Publicado`: OK
- copia de `Borrador IA` a `Resultado final`: OK
- Slack de publicación: OK
- Log de publicación: OK

Ejecución: 4 módulos, 0 errores, 4 créditos.

## 3. Rechazo HITL

Caso: registro con `Estado = Rechazado` y motivo de rechazo.

Resultado observado:
- Slack de rechazo: OK
- Log `Rechazado`: OK
- marcado `Rechazo procesado = true`: OK
- cero publicación externa: OK

## 4. Dato incompleto

Caso: registro con `Estado = Generando` e `Idea Semilla` vacía.

Resultado observado:
- creación de `MISSING_IDEA_SEED`: OK
- Slack de alerta: OK
- marcado `Dato incompleto registrado = true`: OK
- no se ejecutó el modelo: OK

## 5. Error de publicación

Se modificó temporalmente el módulo de publicación para provocar de forma determinística un fallo de Airtable.

Resultado observado:
- módulo de publicación: error esperado
- Error Handler: ejecutado
- registro en tabla `Errores`: OK
- alerta Slack: OK
- escenario controló la contingencia

Después de la prueba, el módulo fue restaurado a su configuración normal.

## 6. Reintentos

Se añadieron módulos `Retry/Break` en las dos operaciones críticas:
- generación IA
- publicación

Configuración:
- 3 intentos
- 1 minuto entre intentos

## 7. Protección anti-loop

Durante las pruebas se detectó que crear Logs/Errores modifica automáticamente campos vinculados del registro de Contenido, lo que vuelve a actualizar `Última modificación`.

Corrección aplicada:
- campo `Rechazo procesado`
- campo `Dato incompleto registrado`
- filtros de ruta exigen que dichos flags todavía no sean `true`
- módulo `Rechazado - Marcar procesado` al final de la ruta de rechazo
- módulo `Incompleto - Marcar procesado` al final de la ruta de dato incompleto

Prueba final definitiva:
- se crearon dos registros nuevos, uno `Rechazado` y otro `Generando` sin Idea Semilla;
- ejecución `75075260a2b743da9824014faabf42ec`: 7 operaciones, 0 errores; ejecutó Slack + Log/Error + módulos de marcado 31 y 32;
- ejecución inmediatamente posterior `7c45126558b64fc4bb2982f67c53e458`: 1 operación, 0 errores, 1 crédito; solo se ejecutó el trigger y ninguna ruta volvió a procesar los registros.

Resultado anti-loop: PASS.

## 8. Veredicto

Las rutas funcionales principales, HITL, RAG, validación de entrada, trazabilidad, alertas, manejo de errores, retries e idempotencia fueron comprobados con ejecuciones reales.

El escenario final queda apto para demostración y entrega. La única ejecución incompleta visible en Make corresponde a la prueba intencional de error de publicación; no representa un fallo actual del blueprint.
