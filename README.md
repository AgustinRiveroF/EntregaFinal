# Ecosistema de Automatización IA Autónomo para Negocios

Proyecto final de **AI Automation — CoderHouse**. Implementa un pipeline de contenido con **Airtable + Make + OpenAI + RAG + HITL + Slack**, concentrado en **un único escenario de Make** para reducir consumo operativo y mantener toda la lógica auditable en un solo flujo.

## Arquitectura final

**Escenario único:** `FINAL - Pipeline IA RAG + HITL + Resiliencia`

El trigger `Airtable / Watch Records` observa la tabla `Contenido` y un Router separa cuatro rutas controladas:

1. **Generación RAG** — `Estado = Generando` + `Idea Semilla` no vacía.
   - consulta `Base de Conocimiento`;
   - agrega `Tema + Contenido` como contexto privado;
   - GPT-5 nano genera el borrador sin recurrir a información externa;
   - actualiza el registro a `En revisión` y fuerza `Aprobado = false`;
   - notifica por Slack y registra trazabilidad.
2. **Validación de entrada** — `Estado = Generando` + `Idea Semilla` vacía + `Dato incompleto registrado != true`.
   - bloquea el consumo de IA;
   - registra el incidente en `Errores`;
   - alerta por Slack;
   - marca `Dato incompleto registrado = true` para evitar reprocesamientos.
3. **Aprobación HITL** — `Aprobado = true` + `Estado = En revisión`.
   - copia `Borrador IA` a `Resultado final`;
   - cambia a `Publicado`;
   - notifica y registra log de publicación.
4. **Rechazo HITL** — `Estado = Rechazado` + `Rechazo procesado != true`.
   - no produce ninguna salida externa;
   - notifica el rechazo;
   - registra trazabilidad;
   - marca `Rechazo procesado = true` para evitar reprocesamientos.

## Human-in-the-loop

La IA **nunca publica por sí sola**. Al terminar la generación el registro queda en:

`Estado = En revisión` + `Aprobado = false`

Solo la acción humana de marcar `Aprobado = true`, manteniendo `Estado = En revisión`, habilita la ruta final a `Publicado`.

## RAG privado

La tabla `Base de Conocimiento` contiene información atómica como tono de marca, producto, público objetivo, restricciones y CTA. Make recupera esos registros y los agrega antes de llamar al modelo. El prompt instruye al modelo a utilizar exclusivamente ese contexto y la `Idea Semilla`.

## Resiliencia

Las dos operaciones críticas tienen Error Handler:

- **Generación IA**: registra `OPENAI_RUNTIME_ERROR`, alerta por Slack y usa `Retry/Break` con **3 intentos automáticos** y 1 minuto entre intentos.
- **Publicación**: registra `AIRTABLE_UPDATE_ERROR`, alerta por Slack y usa `Retry/Break` con **3 intentos automáticos** y 1 minuto entre intentos.

Ningún error crítico se interpreta como publicación exitosa.

## Airtable

Base: `Pipeline de Contenido IA`

Tablas principales:
- `Contenido`
- `Base de Conocimiento`
- `Logs`
- `Errores`

Estados del ciclo de vida:

`Generando → En revisión → Publicado`

También se contempla `Rechazado` como salida humana sin publicación.

Campos internos anti-loop:
- `Rechazo procesado`
- `Dato incompleto registrado`

**Shared View:** https://airtable.com/app9d2rjDXsLqTRLC/shrpB1FBL0kJDgfNC

## Dashboard Ejecutivo

La interfaz pública `Centro de Comando HITL` permite supervisar piezas, estados, ejecuciones, errores y severidad.

**Dashboard:** https://airtable.com/app9d2rjDXsLqTRLC/shrvZ2RPRVhFF9rLU

## Seguridad y buenas prácticas

- No se almacenan API keys ni credenciales en GitHub.
- Validación de datos antes de consumir IA.
- Rutas filtradas para evitar procesamiento fuera del estado esperado.
- Contexto RAG privado y controlado.
- HITL obligatorio antes de la salida final.
- Tabla independiente de errores y logs operativos.
- Alertas por Slack para revisión, rechazo e incidentes.
- Prompt con restricciones explícitas contra invención de datos.
- Protección anti-loop para eventos que modifican campos vinculados en Airtable.
- Diseño optimizado para el plan gratuito de Make: un único escenario activo.

## Validación runtime realizada

Se ejecutó una batería real de pruebas sobre el escenario final:

- **Generación RAG:** PASS — 7 módulos, 0 errores, borrador creado y detenido en `En revisión`.
- **Aprobación HITL:** PASS — 4 módulos, 0 errores, `Publicado` + `Resultado final` + Slack + Log.
- **Rechazo HITL:** PASS — Slack + Log, sin publicación.
- **Dato incompleto:** PASS — Error + Slack, sin consumo de IA.
- **Error de publicación simulado:** PASS — falló el módulo de publicación, el Error Handler creó el registro de error y alertó por Slack.
- **Anti-loop:** PASS — al modificar registros ya procesados, el trigger los detectó pero las rutas quedaron bloqueadas; la ejecución consumió solo el trigger.

El detalle se encuentra en `docs/VALIDACION_RUNTIME.md` y `tests/stress_test_plan.md`.

## Estructura del repositorio

- `blueprints/` — snapshot técnico sanitizado del escenario final.
- `docs/` — arquitectura final y evidencia de validación runtime.
- `prompts/` — prompt de generación.
- `schemas/` — contratos JSON de transferencia, contexto, logs y errores.
- `tests/` — plan y resultados de validación funcional y de resiliencia.

## Estado

Arquitectura consolidada a un único escenario para la entrega final. Los escenarios históricos separados quedan fuera de la arquitectura final y no deben permanecer activos.
