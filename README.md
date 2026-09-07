# Ecosistema de Automatización IA Autónomo para Negocios

Proyecto final de AI Automation: pipeline autónomo de contenido con **Make + Airtable + OpenAI + RAG + HITL + Slack**.

## Arquitectura
1. Airtable detecta `Estado = Generando`.
2. Valida que `Idea Semilla` exista.
3. Recupera contexto de `Base de Conocimiento`.
4. Agrega el contexto RAG.
5. OpenAI genera un borrador mediante prompt estructurado.
6. Airtable guarda el borrador en `En revisión`.
7. Slack notifica que existe una pieza pendiente.
8. El sistema queda bloqueado por HITL.
9. Cuando `Aprobado = true`, el flujo entrega la salida y actualiza a `Publicado`.
10. Si se rechaza, se registra `Rechazado` y no se ejecuta salida externa.
11. Logs y Errores alimentan el Dashboard Ejecutivo.

## Escenarios Make finales
- `FINAL - Generación RAG + HITL + Resiliencia`
  - RAG + OpenAI
  - Slack de revisión
  - Log de éxito
  - Error Handler sobre OpenAI
  - Registro en Airtable `Errores`
  - Slack de contingencia
  - Break con 3 reintentos
- `FINAL - Aprobación HITL + Salida + Logs`
  - filtro `Aprobado = true` + `Estado = En revisión`
  - actualización a `Publicado`
  - Slack de salida
  - Log de publicación
  - Error Handler de actualización
- `FINAL - Rechazo HITL + Logs`
  - procesa `Estado = Rechazado`
  - notifica en Slack
  - registra log sin salida externa
- `FINAL - Validación de Datos Incompletos`
  - bloquea `Estado = Generando` con `Idea Semilla` vacía
  - registra error antes de consumir IA
  - alerta en Slack

## Entregables
- `docs/arquitectura_sistema.pdf`
- `docs/manual_operativo_datos.pdf`
- `docs/matriz_costos.pdf`
- `docs/seguridad_resiliencia.pdf`
- `blueprints/` - Blueprints exportados de Make
- `schemas/` - JSON Schema de transferencia
- `prompts/` - Prompt estructurado y ejemplo de `cache_control`
- `screenshots/` - Evidencias de ejecución
- `tests/stress_test_plan.md`

## Base de datos
Airtable contiene las tablas **Contenido**, **Base de Conocimiento**, **Logs** y **Errores**, vinculadas para mantener trazabilidad.

Estados contemplados: `Generando`, `En revisión`, `Aprobado`, `Rechazado`, `Publicado`.

## Human-in-the-loop
El flujo no permite la salida crítica mientras `Aprobado = false`. Solo `Aprobado = true` junto con `Estado = En revisión` habilita el paso final.

## Seguridad y resiliencia
- Minimización de datos.
- Sin API keys ni credenciales en este repositorio.
- Trigger filtrado para evitar loops.
- Validación de datos antes del modelo.
- Error Handling con registro operativo.
- Break con reintentos automáticos.
- Revisión humana antes de interactuar con el exterior.

## Dashboard
La interfaz de Airtable `Centro de Comando HITL` contiene:
- total de piezas;
- piezas por estado;
- total de ejecuciones;
- tasa de errores;
- ejecuciones por estado;
- errores registrados;
- errores por severidad.

**URL pública del Dashboard:** `PENDIENTE_COPIAR_ENLACE_PUBLICO`

## Demo
**Video demo (3 min):** `PENDIENTE_GRABAR_VIDEO`

## Estado de validación
- [x] RAG funcional
- [x] HITL bloqueado demostrado
- [x] HITL aprobado demostrado
- [x] Airtable relacional creado
- [x] Dashboard construido y publicado en Airtable
- [x] Error Handler final agregado en Make
- [x] Logs y tabla de Errores conectados a Make
- [x] Slack integrado en revisión, salida y contingencia
- [x] Ruta Rechazado creada
- [x] Validación de datos incompletos creada
- [ ] Ejecutar 5 pruebas finales del flujo actualizado
- [ ] Copiar enlace público de Shared View / interfaz
- [ ] Exportar blueprints actualizados de Make
- [ ] Subir evidencias finales de ejecución
- [ ] Video demo de 3 minutos

## Bloqueo actual de pruebas
Make devolvió: `Scenario cannot be run because its organization or team is paused. Resolve the exceeded operations or data transfer limit and try again.`

La arquitectura ya está configurada; para completar las pruebas reales hay que reactivar la capacidad de ejecución de la organización/equipo de Make.
