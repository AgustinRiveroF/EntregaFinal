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
10. Logs y Errores alimentan el Dashboard Ejecutivo.

## Entregables
- `docs/arquitectura_sistema.pdf`
- `docs/manual_operativo_datos.pdf`
- `docs/matriz_costos.pdf`
- `docs/seguridad_resiliencia.pdf`
- `blueprints/` - Blueprints reales exportados de Make
- `schemas/` - JSON Schema de transferencia
- `prompts/` - Prompt estructurado y ejemplo de `cache_control`
- `screenshots/` - Evidencias de ejecución
- `tests/stress_test_plan.md`

## Base de datos
Airtable contiene las tablas **Contenido**, **Base de Conocimiento**, **Logs** y **Errores**, vinculadas para mantener trazabilidad.

## Human-in-the-loop
El flujo no permite la salida crítica mientras `Aprobado = false`. Solo `Aprobado = true` junto con `Estado = En revisión` habilita el paso final.

## Seguridad
- Minimización de datos.
- Sin API keys ni credenciales en este repositorio.
- Trigger filtrado para evitar loops.
- Validación de datos antes del modelo.
- Error Handling con registro operativo.
- Revisión humana antes de interactuar con el exterior.

## Dashboard
La interfaz de Airtable incluye total de piezas, piezas por estado, total de ejecuciones, tasa de errores, ejecuciones por estado y errores por severidad.

**URL pública del Dashboard:** `PENDIENTE_PUBLICAR_SHARED_VIEW`

## Demo
**Video demo (3 min):** `PENDIENTE_GRABAR_VIDEO`

## Estado de validación
- [x] RAG funcional
- [x] HITL bloqueado demostrado
- [x] HITL aprobado demostrado
- [x] Airtable relacional creado
- [x] Dashboard construido y publicado en Airtable
- [ ] Error Handler final conectado en Make
- [ ] 5 pruebas de estrés documentadas
- [ ] Enlace público de Shared View copiado
- [ ] Video demo de 3 minutos
