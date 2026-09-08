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
- límite de salida configurado: 700 tokens
- Update a `En revisión`: OK
- Slack de revisión: OK
- guardado de `Slack Thread TS`: OK
- Log de generación: OK

Prueba posterior a la rúbrica: ejecución `e4f6d114b4ee4e188b98ba95073ed906`: 8 operaciones, 0 errores, 9.21 créditos.

El registro terminó con:
- `Estado = En revisión`
- `Borrador IA` completo
- `Aprobado` sin marcar
- `Slack Thread TS = 1788835171.121349`
- sin `Resultado final`

Esto confirma que la IA genera pero no publica.

## 2. Aprobación HITL

Caso: el mismo registro fue marcado manualmente con `Aprobado = true` manteniendo `Estado = En revisión`.

Resultado observado:
- actualización a `Publicado`: OK
- copia de `Borrador IA` a `Resultado final`: OK
- Slack de publicación: OK
- la publicación respondió en el mismo hilo usando `thread_ts`: OK
- Log de publicación: OK

Prueba posterior a la rúbrica: ejecución `6194e7390cd54569b90690e8a9709707`: 4 operaciones, 0 errores, 4 créditos.

La entrada real del módulo Slack confirmó `thread_ts = 1788835171.121349` y la respuesta de Slack devolvió el mismo `thread_ts`.

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

Prueba definitiva:
- ejecución `75075260a2b743da9824014faabf42ec`: procesó rechazo + dato incompleto con sus flags;
- ejecución posterior `7c45126558b64fc4bb2982f67c53e458`: 1 operación, 0 errores, solo trigger; ninguna ruta se reprocesó.

Resultado anti-loop: PASS.

## 8. Optimización de IA

Configuración final del módulo OpenAI:
- modelo: `gpt-5-nano`
- máximo de salida: `700` tokens
- prompt dinámico con `Idea Semilla` y contexto RAG agregado
- instrucción adicional: máximo 220 palabras

## 9. Estado final

- escenario activo: sí
- ejecuciones incompletas: 0
- conexiones Airtable y Slack: OK
- RAG: PASS
- HITL: PASS
- Thread ID Slack: PASS
- Error Handlers + Break/Retry: PASS
- anti-loop: PASS

El escenario queda apto para demostración y entrega.
