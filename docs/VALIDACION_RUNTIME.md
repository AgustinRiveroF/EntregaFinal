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
- cero publicación externa: OK

## 4. Dato incompleto

Caso: registro con `Estado = Generando` e `Idea Semilla` vacía.

Resultado observado:
- creación de `MISSING_IDEA_SEED`: OK
- Slack de alerta: OK
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
- cada ruta marca su flag al terminar

Prueba final:
- se modificaron nuevamente ambos registros ya marcados
- el trigger detectó cambios
- ninguna ruta de rechazo o dato incompleto volvió a ejecutarse
- la ejecución consumió únicamente 1 crédito correspondiente al trigger

Resultado: PASS.

## Veredicto

Las rutas funcionales principales, HITL, RAG, validación de entrada, trazabilidad, alertas, manejo de errores y control anti-loop fueron comprobados con ejecuciones reales.
