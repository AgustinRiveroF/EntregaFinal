# Guion Video Demo — 3 minutos

> Objetivo: mostrar el sistema funcionando sin exponer API Keys, tokens ni pantallas de credenciales.

## 0:00–0:20 — Presentación

“Este es mi proyecto final de AI Automation: un ecosistema de contenido con Make, Airtable, OpenAI, RAG, HITL y Slack. El objetivo es generar contenido con contexto privado, validarlo con una persona y mantener trazabilidad y resiliencia.”

## 0:20–0:50 — Airtable / cerebro

Mostrar las tablas `Contenido`, `Base de Conocimiento`, `Logs` y `Errores`.

Señalar:
- estados del pipeline;
- relaciones entre tablas;
- `Aprobado`;
- `Rechazo procesado` y `Dato incompleto registrado` como protección anti-loop;
- `Slack Thread TS` para mantener la conversación en un hilo.

## 0:50–1:35 — Make / corazón

Mostrar el escenario `FINAL - Pipeline IA RAG + HITL + Resiliencia` completo.

Explicar rápidamente las 4 rutas del Router:
1. generación RAG;
2. aprobación HITL;
3. rechazo;
4. dato incompleto.

Mostrar:
- búsqueda de conocimiento;
- Text Aggregator;
- OpenAI GPT-5 nano con límite de salida;
- Error Handlers;
- Break/Retry de 3 intentos.

## 1:35–2:10 — HITL y Slack

Mostrar un registro en `En revisión` con `Aprobado = false`.

Explicar que la IA no puede publicar por sí sola.

Marcar `Aprobado = true` y mostrar el resultado `Publicado` + `Resultado final`.

En Slack, mostrar que el mensaje de publicación queda como respuesta dentro del mismo hilo del mensaje de revisión.

## 2:10–2:35 — Camino infeliz y resiliencia

Mostrar en Airtable/Logs la evidencia de:
- dato incompleto bloqueado antes de IA;
- error registrado;
- rechazo sin publicación;
- anti-loop.

No es necesario volver a provocar el error en vivo: se puede mostrar la evidencia ya registrada.

## 2:35–2:55 — Dashboard

Mostrar `Dashboard Ejecutivo`:
- Total piezas;
- Contenido por estado;
- Total ejecuciones;
- Tasa de errores;
- Errores registrados;
- Errores por severidad.

## 2:55–3:00 — Cierre

“En GitHub se encuentran el PDF de arquitectura y documentación, los schemas JSON, el blueprint de Make, las pruebas, las evidencias y los enlaces públicos a Airtable y al Dashboard.”

## Seguridad de grabación

- No abrir Connections de Make.
- No mostrar API Keys, tokens o secretos.
- No mostrar pantallas de autorización OAuth.
- Usar únicamente datos ficticios de prueba.
