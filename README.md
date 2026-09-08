# Ecosistema de Automatización IA Autónomo para Negocios

Proyecto final de **AI Automation — CoderHouse**: pipeline de contenido con **Make + Airtable + OpenAI + RAG + HITL + Slack** en un único escenario auditable.

## Entregables principales

- **PDF final (5 criterios de rúbrica):** [`docs/Entrega_Final_CoderHouse_Ecosistema_IA.pdf`](docs/Entrega_Final_CoderHouse_Ecosistema_IA.pdf)
- **Índice de entrega:** [`docs/ENTREGA_FINAL.md`](docs/ENTREGA_FINAL.md)
- **Blueprint nativo exportado de Make:** [`blueprints/FINAL - Pipeline IA RAG + HITL + Resiliencia.blueprint.json`](blueprints/FINAL%20-%20Pipeline%20IA%20RAG%20%2B%20HITL%20%2B%20Resiliencia.blueprint.json)
- **Schemas JSON:** [`schemas/`](schemas/)
- **Validación runtime:** [`docs/VALIDACION_RUNTIME.md`](docs/VALIDACION_RUNTIME.md)
- **Pruebas de estrés:** [`tests/stress_test_plan.md`](tests/stress_test_plan.md)
- **Evidencias:** [`evidencias/`](evidencias/)
- **Guion video demo 3 min:** [`docs/VIDEO_DEMO_3_MIN.md`](docs/VIDEO_DEMO_3_MIN.md)

## Enlaces públicos obligatorios

- **Airtable Shared View:** https://airtable.com/app9d2rjDXsLqTRLC/shrpB1FBL0kJDgfNC
- **Dashboard Ejecutivo:** https://airtable.com/app9d2rjDXsLqTRLC/shrvZ2RPRVhFF9rLU

## Arquitectura final

Escenario único de Make: `FINAL - Pipeline IA RAG + HITL + Resiliencia`.

El trigger `Airtable / Watch Records` observa `Contenido` y un Router separa cuatro rutas:

1. **Generación RAG** — `Estado = Generando` + `Idea Semilla` no vacía.
   - busca hasta 20 registros en `Base de Conocimiento`;
   - agrega `Tema + Contenido`;
   - genera con **GPT-5 nano** usando prompt dinámico;
   - límite de salida configurado en **700 tokens** y máximo 220 palabras por prompt;
   - actualiza a `En revisión` y fuerza `Aprobado = false`;
   - avisa por Slack, guarda el `Slack Thread TS` y registra Log.
2. **Aprobación HITL** — `Aprobado = true` + `Estado = En revisión`.
   - copia `Borrador IA` a `Resultado final`;
   - cambia a `Publicado`;
   - responde en el **mismo hilo de Slack** mediante `thread_ts`;
   - registra publicación exitosa.
3. **Rechazo HITL** — `Estado = Rechazado` + `Rechazo procesado != true`.
   - no publica;
   - notifica y registra Log;
   - marca `Rechazo procesado = true`.
4. **Dato incompleto** — `Estado = Generando` + `Idea Semilla` vacía + `Dato incompleto registrado != true`.
   - bloquea el consumo de IA;
   - registra `MISSING_IDEA_SEED`;
   - alerta por Slack;
   - marca `Dato incompleto registrado = true`.

## Human-in-the-loop

La IA **nunca publica por sí sola**. Después de generar queda:

`Estado = En revisión` + `Aprobado = false`

Solo una aprobación humana permite la ruta crítica a `Publicado`.

## Resiliencia y seguridad

- Error Handler de IA → Airtable `Errores` + Slack + `Break/Retry` de 3 intentos.
- Error Handler de publicación → Airtable `Errores` + Slack + `Break/Retry` de 3 intentos.
- Protección anti-loop con `Rechazo procesado` y `Dato incompleto registrado`.
- Variables dinámicas desde Airtable/Make; sin API Keys ni secretos en GitHub.
- RAG limitado al contexto privado recibido.
- Slack organizado con `thread_ts` para revisión → publicación.

## Airtable

Tablas vinculadas:
- `Contenido`
- `Base de Conocimiento`
- `Logs`
- `Errores`

El esquema completo y los contratos JSON están explicados en el PDF final.

## Dashboard Ejecutivo

El panel público `Centro de Comando HITL` incluye:
- Total piezas
- Contenido por estado
- Total ejecuciones
- Tasa de errores
- Ejecuciones por estado
- Errores registrados
- Errores por severidad

## Validación runtime

Se comprobaron con ejecuciones reales:
- Generación RAG ✅
- GPT-5 nano + límite de salida ✅
- HITL bloqueado antes de aprobación ✅
- Aprobación → `Publicado` + `Resultado final` ✅
- Slack Thread ID / `thread_ts` ✅
- Rechazo sin publicación ✅
- Dato incompleto sin IA ✅
- Error Handler de publicación ✅
- Retry/Break ✅
- Anti-loop ✅

La prueba final de hilo ejecutó generación con 8 operaciones y 0 errores, y la aprobación con 4 operaciones y 0 errores. El detalle y IDs de ejecución están en `docs/VALIDACION_RUNTIME.md`.

## Optimización de costos

La matriz comparativa de modelos, estrategia Batch y estimaciones de costo se encuentra en `docs/Entrega_Final_CoderHouse_Ecosistema_IA.pdf`. Para este caso se seleccionó **GPT-5 nano** por su relación costo/suficiencia para generación breve con RAG.

## Estado

**Escenario final activo, conexiones OK y 0 ejecuciones incompletas.**
