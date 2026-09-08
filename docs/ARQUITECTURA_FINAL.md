# Arquitectura final — Proyecto AI Automation

## Objetivo

Automatizar la generación de contenido con contexto privado, mantener control humano obligatorio antes de publicar y registrar toda la trazabilidad operativa.

## Escenario único en Make

`FINAL - Pipeline IA RAG + HITL + Resiliencia`

Frecuencia: cada 15 minutos.

### Trigger

Airtable `Watch Records` sobre la tabla `Contenido`, observando `Última modificación`.

### Router principal

#### Ruta 1 — Aprobación HITL
Condición: `Aprobado = true` AND `Estado = En revisión`.

Acciones:
- actualizar `Estado = Publicado`;
- copiar `Borrador IA` a `Resultado final`;
- actualizar fecha;
- notificar por Slack;
- registrar log exitoso.

Error handling:
- registrar `AIRTABLE_UPDATE_ERROR`;
- alertar por Slack;
- `Retry/Break`: 3 intentos, 1 minuto entre intentos.

#### Ruta 2 — Rechazo HITL
Condición: `Estado = Rechazado` AND `Rechazo procesado != true`.

Acciones:
- notificar rechazo;
- registrar log;
- marcar `Rechazo procesado = true`;
- no producir salida externa.

La marca interna evita que Airtable vuelva a disparar la misma ruta cuando el vínculo a Logs modifica `Última modificación`.

#### Ruta 3 — Dato incompleto
Condición: `Estado = Generando` AND `Idea Semilla` vacía AND `Dato incompleto registrado != true`.

Acciones:
- registrar `MISSING_IDEA_SEED`;
- alertar por Slack;
- marcar `Dato incompleto registrado = true`;
- bloquear consumo de IA.

La marca interna evita duplicados cuando Airtable actualiza el vínculo a Errores.

#### Ruta 4 — Generación RAG
Condición: `Estado = Generando` AND `Idea Semilla` no vacía.

Acciones:
- consultar `Base de Conocimiento`;
- agregar `Tema + Contenido`;
- generar borrador con `gpt-5-nano`;
- actualizar a `En revisión`;
- fijar `Aprobado = false`;
- notificar revisión por Slack;
- registrar log de generación.

Error handling:
- registrar `OPENAI_RUNTIME_ERROR`;
- alertar por Slack;
- `Retry/Break`: 3 intentos, 1 minuto entre intentos.

## Control HITL

La IA no tiene una ruta que publique directamente después de generar. El borrador queda detenido en `En revisión`. La publicación solo ocurre luego de una acción humana explícita sobre `Aprobado`.

## Seguridad y resiliencia

- no se almacenan secretos en GitHub;
- validación previa al consumo de IA;
- contexto RAG privado;
- filtros de estado para evitar procesamiento indebido;
- logs y errores persistentes en Airtable;
- alertas operativas en Slack;
- reintentos controlados en operaciones críticas;
- protección anti-loop en rechazo y dato incompleto;
- separación entre generación, validación, rechazo y publicación mediante rutas del Router.

## Validación runtime

Se validó el flujo con ejecuciones reales:

- Generación RAG: PASS.
- Aprobación HITL y publicación: PASS.
- Rechazo: PASS.
- Idea Semilla vacía: PASS.
- Error de publicación simulado: PASS del Error Handler.
- Anti-loop: PASS; una modificación posterior sobre registros ya procesados ejecutó únicamente el trigger y ninguna ruta secundaria.

Los detalles se documentan en `docs/VALIDACION_RUNTIME.md`.

## Evidencias para CoderHouse

1. escenario completo con Router y 4 rutas;
2. contenido generado en `En revisión` y `Aprobado = false`;
3. filtro HITL bloqueando publicación sin aprobación;
4. publicación posterior a aprobación humana;
5. rechazo humano sin salida externa;
6. validación de Idea Semilla vacía sin consumo IA;
7. Error Handler + Retry/Break;
8. tablas Logs / Errores y Dashboard Ejecutivo.

## Estado operativo

Los escenarios históricos separados quedan desactivados. Solo debe permanecer activo el escenario consolidado final.
