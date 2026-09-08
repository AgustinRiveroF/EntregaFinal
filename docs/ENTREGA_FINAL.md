# Entrega Final - Ecosistema de Automatizacion IA Autonomo para Negocios

Caso de uso: pipeline de contenido con Airtable + Make + OpenAI + RAG + HITL + Slack.

## Entregables de rubrica

1. Mapa de arquitectura: `docs/Entrega_Final_CoderHouse_Ecosistema_IA.pdf`.
2. Manual operativo de datos: incluido en el PDF y respaldado por `schemas/`.
3. Matriz de costos: incluida en el PDF.
4. Seguridad y resiliencia: incluida en el PDF y validada en `docs/VALIDACION_RUNTIME.md`.
5. Dashboard de control: Airtable Interface `Centro de Comando HITL`.

## Enlaces

- Repositorio publico: https://github.com/AgustinRiveroF/EntregaFinal
- Shared View DB: https://airtable.com/app9d2rjDXsLqTRLC/shrpB1FBL0kJDgfNC
- Dashboard: https://airtable.com/app9d2rjDXsLqTRLC/shrvZ2RPRVhFF9rLU

## Flujo final

Escenario unico activo en Make: `FINAL - Pipeline IA RAG + HITL + Resiliencia`.

Rutas principales:
- Generacion RAG: Estado = Generando + Idea Semilla no vacia.
- Aprobacion HITL: Aprobado = true + Estado = En revision.
- Rechazo HITL: Estado = Rechazado + Rechazo procesado != true.
- Dato incompleto: Estado = Generando + Idea Semilla vacia + Dato incompleto registrado != true.

## Validacion

Se ejecutaron pruebas reales de generacion, aprobacion, rechazo, dato incompleto, error de publicacion y anti-loop. Ver `docs/VALIDACION_RUNTIME.md` y `tests/stress_test_plan.md`.
