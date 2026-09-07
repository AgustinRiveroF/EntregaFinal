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
- alertar por Slack.

#### Ruta 2 — Rechazo HITL
Condición: `Estado = Rechazado`.

Acciones:
- notificar rechazo;
- registrar log;
- no producir salida externa.

#### Ruta 3 — Dato incompleto
Condición: `Estado = Generando` AND `Idea Semilla` vacía.

Acciones:
- registrar `MISSING_IDEA_SEED`;
- alertar por Slack;
- bloquear consumo de IA.

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
- alertar por Slack.

## Control HITL

La IA no tiene una ruta que publique directamente después de generar. El borrador queda detenido en `En revisión`. La publicación solo ocurre luego de una acción humana explícita sobre `Aprobado`.

## Seguridad y resiliencia

- no se almacenan secretos en GitHub;
- validación previa al consumo de IA;
- contexto RAG privado;
- filtros de estado para evitar procesamiento indebido;
- logs y errores persistentes en Airtable;
- alertas operativas en Slack;
- separación entre generación, validación, rechazo y publicación mediante rutas del Router.

## Evidencias para CoderHouse

Las capturas recomendadas son:

1. escenario completo con Router y 4 rutas;
2. contenido generado en `En revisión` y `Aprobado = false`;
3. filtro HITL bloqueando publicación sin aprobación;
4. publicación posterior a aprobación humana;
5. rechazo humano sin salida externa;
6. validación de Idea Semilla vacía sin consumo IA;
7. tablas Logs / Errores y Dashboard Ejecutivo.

## Estado operativo

Los escenarios históricos separados quedan desactivados. Solo debe permanecer activo el escenario consolidado final.
