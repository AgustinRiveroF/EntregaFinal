# Ecosistema de Automatización IA Autónomo para Negocios

Proyecto final de **AI Automation — CoderHouse**. Implementa un pipeline de contenido con **Airtable + Make + OpenAI + RAG + HITL + Slack**, concentrado en **un único escenario de Make** para reducir consumo operativo y mantener toda la lógica auditable en un solo flujo.

## Arquitectura final

**Escenario único:** `FINAL - Pipeline IA RAG + HITL + Resiliencia`

El trigger `Airtable / Watch Records` observa la tabla `Contenido` y un Router separa rutas mutuamente controladas:

1. **Generación RAG** — `Estado = Generando` + `Idea Semilla` no vacía.
   - consulta `Base de Conocimiento`;
   - agrega `Tema + Contenido` como contexto privado;
   - GPT-5 nano genera el borrador sin recurrir a información externa;
   - actualiza el registro a `En revisión` y fuerza `Aprobado = false`;
   - notifica por Slack y registra trazabilidad.
2. **Validación de entrada** — `Estado = Generando` + `Idea Semilla` vacía.
   - bloquea el consumo de IA;
   - registra el incidente en `Errores`;
   - alerta por Slack.
3. **Aprobación HITL** — `Aprobado = true` + `Estado = En revisión`.
   - copia `Borrador IA` a `Resultado final`;
   - cambia a `Publicado`;
   - notifica y registra log de publicación.
4. **Rechazo HITL** — `Estado = Rechazado`.
   - no produce ninguna salida externa;
   - notifica el rechazo y registra trazabilidad.
5. **Resiliencia**.
   - errores del nodo de IA y de publicación se registran en la tabla `Errores`;
   - las rutas críticas emiten alertas operativas en Slack;
   - ningún fallo se interpreta como publicación exitosa.

## Human-in-the-loop

La IA **nunca publica por sí sola**. Al terminar la generación el registro queda en:

`Estado = En revisión` + `Aprobado = false`

Solo la acción humana de marcar `Aprobado = true`, manteniendo `Estado = En revisión`, habilita la ruta final a `Publicado`.

## RAG privado

La tabla `Base de Conocimiento` contiene información atómica como tono de marca, producto, público objetivo, restricciones y CTA. Make recupera esos registros y los agrega antes de llamar al modelo. El prompt instruye al modelo a utilizar exclusivamente ese contexto y la `Idea Semilla`.

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
- Diseño optimizado para el plan gratuito de Make: un único escenario activo.

## Estructura del repositorio

- `blueprints/` — snapshot técnico del escenario final.
- `prompts/` — prompt de generación y material complementario.
- `schemas/` — contratos JSON de transferencia, contexto, logs y errores.
- `tests/` — plan de validación funcional y de resiliencia.

## Criterios funcionales de validación

- Idea válida → generación con RAG → `En revisión`.
- Sin aprobación humana → no existe publicación.
- Aprobación humana → `Publicado` + `Resultado final`.
- Rechazo humano → log de rechazo y cero publicación.
- Idea vacía → bloqueo previo al modelo.
- Fallo de IA/publicación → registro de error + alerta operativa.

## Evidencia recomendada para la entrega

1. Captura completa del escenario único con Router y rutas.
2. Registro en Airtable en `En revisión` con `Aprobado = false`.
3. Evidencia de que la ruta de publicación no avanza sin aprobación.
4. Registro aprobado terminando en `Publicado` y `Resultado final`.
5. Registro rechazado sin salida externa.
6. Validación de `Idea Semilla` vacía sin consumo de IA.
7. Tabla de Logs y Errores + Dashboard Ejecutivo.

## Estado

Arquitectura consolidada a un único escenario para la entrega final. Los escenarios históricos separados quedan fuera de la arquitectura final y no deben permanecer activos.
