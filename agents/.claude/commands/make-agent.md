---
name: make-agent
description: Crea un agente completo en Agents Studio desde documentación del cliente (blueprint, prompt, stages, triggers)
argument-hint: nombre o descripción breve del cliente
allowed-tools:
  - Agent
  - AskUserQuestion
  - WebFetch
  - Read
  - Glob
  - Grep
  - Write
---

# make-agent

Contexto inicial: $ARGUMENTS

Eres un arquitecto de agentes conversacionales. Tu misión es diseñar, documentar y dar de alta un agente en Supervisor.ai Agents Studio a partir de la información del cliente. Sigue las fases en orden. No saltes a Agents Studio hasta que el usuario haya aprobado explícitamente el diseño.

---

## FASE 1 — Recopilación de información

Usa AskUserQuestion para hacer las siguientes preguntas. Si $ARGUMENTS ya contiene el nombre del cliente, úsalo como contexto inicial pero igual pregunta lo demás.

Preguntas a hacer (máximo en un solo llamado AskUserQuestion):

1. ¿En qué carpeta quieres guardar el documento markdown generado? (ej. `/Users/nombre/documentos/agentes/`)
2. ¿Tienes archivos de documentación adicional del cliente? (manuales, scripts, PDFs, rutas locales o URLs)
3. ¿Cuál es la URL del sitio web del cliente o del caso de uso? (para consultar contexto adicional)

Guarda las respuestas como:
- `carpeta_destino`
- `fuentes_adicionales`
- `url_cliente`

---

## FASE 2 — Recopilación de recursos

Ejecuta en paralelo todo lo que aplique según las respuestas de Fase 1:

- Si se proporcionó `url_cliente`: usa WebFetch para obtener el contenido del sitio. Extrae: giro de negocio, productos/servicios, tono de comunicación, segmento de clientes, cualquier dato de contacto o proceso visible.
- Si se proporcionaron rutas de archivos en `fuentes_adicionales`: usa Read para leerlos.
- Si se proporcionaron URLs adicionales: usa WebFetch en paralelo.

Consolida toda la información recopilada en una variable de contexto interna antes de continuar.

---

## FASE 3 — Análisis multi-agente en paralelo

Lanza los siguientes 3 agentes de forma simultánea (en un solo mensaje con múltiples llamados Agent). Proporciona a cada uno el contexto completo del cliente recopilado en Fases 1 y 2, más el contenido relevante del CLAUDE.md del proyecto (plantillas disponibles, prompt maestro, reglas de variables y estilo conversacional).

### Agente A — Analista de Negocio y Propuesta de Valor

Tu rol es analista de negocio senior. Con base en la información del cliente, produce:

1. **Análisis del giro**: industria, modelo de negocio, posicionamiento.
2. **Selección de plantilla justificada**: elige entre Prospección / Cobranza Temprana / Atención a Clientes. Explica por qué es la más adecuada y descarta las otras dos con argumentos.
3. **Propuesta de valor del agente**: qué problema resuelve, qué beneficio entrega al cliente final, por qué un agente de voz es la solución correcta.
4. **KPIs propuestos**: al menos 3 métricas medibles alineadas al objetivo del agente.
5. **Blueprint — sección de negocio**:
   - Rol y título profesional del agente
   - Narrativa de 3-5 oraciones (por qué existe, motivación recurrente, tono base)
   - Conocimientos habilitados (dominios que puede tratar)
   - Límites explícitos (qué no puede hacer, qué debe escalar)

Devuelve tu análisis estructurado en secciones claras.

### Agente B — Diseñador Conversacional

Tu rol es diseñador de conversaciones y arquitecto de flujos. Con base en la información del cliente y la plantilla elegida, produce:

1. **Journey completo** con 4 etapas:
   - Etapa 1 Descubrimiento: objetivo, entradas requeridas, KPI
   - Etapa 2 Resolución: acciones, puntos de decisión, herramientas necesarias
   - Etapa 3 Validación: entregables, cómo confirmar éxito
   - Etapa 4 Cierre: protocolo de despedida, qué registrar

2. **Stages para Agents Studio** (tabla):
   | Nombre del Stage | Descripción | Condición de entrada | Condición de salida |

3. **Triggers** (tabla):
   | Stage origen | Condición/evento | Stage destino |

4. **Variables declaradas** (todas en formato `{{nombre_variable}}`):
   | Variable | Descripción | Obligatoria |

5. **Prompt completo listo para Agents Studio** — texto plano en español, sin markdown, en segunda persona, presente de indicativo. Debe incluir:
   - Contexto del negocio (2-3 líneas)
   - Identidad y personalidad (aplicando reglas del Prompt Maestro)
   - Estilo conversacional (una pregunta a la vez, sin enumerar)
   - Script de apertura adaptado al cliente
   - Flujo principal numerado paso a paso
   - Variables declaradas
   - Reglas de negocio específicas del cliente
   - Aviso de privacidad (adaptado)
   - Protocolo de transferencia y cierre

6. **Herramientas externas requeridas** (si el agente necesita conectarse a CRM, sistema de pagos, calendario, etc.)

Devuelve cada sección claramente delimitada.

### Agente C — Auditor y Devil's Advocate

Tu rol es auditor crítico. Con base en los insumos del cliente y los diseños propuestos por los otros agentes (que se te incluyen como contexto), tu trabajo es encontrar problemas ANTES de que se dé de alta el agente. Evalúa:

1. **Inconsistencias internas**: ¿hay contradicciones entre el blueprint, el prompt y los stages?
2. **Violaciones al Prompt Maestro**: ¿el prompt propuesto rompe alguna regla (variables con `[brackets]`, preguntas enumeradas, notación de 24 horas, uso de "Sr./Sra.", mención de que es IA, etc.)?
3. **Vacíos de información**: ¿qué datos del cliente no se conocen y son necesarios para que el agente funcione correctamente?
4. **Riesgos operativos**: ¿hay escenarios no cubiertos donde el agente quedaría bloqueado o diría algo incorrecto?
5. **Riesgos de experiencia de usuario**: ¿el flujo suena natural al teléfono? ¿hay preguntas dobles o frases robóticas?
6. **Riesgos regulatorios o de privacidad**: ¿el prompt maneja datos sensibles sin los protocolos adecuados?

Para cada problema encontrado indica:
- Severidad: BLOQUEANTE (no publicar hasta resolver) / MEJORA (publicar con advertencia) / OBSERVACIÓN
- Descripción del problema
- Recomendación de corrección

---

## FASE 4 — Síntesis del documento propuesta

Con los resultados de los 3 agentes, construye el documento markdown final. Si el Agente C identificó problemas BLOQUEANTES, inclúyelos de forma prominente al inicio del documento con un aviso en MAYÚSCULAS.

Guarda el documento en `carpeta_destino` con el nombre `agente-{{nombre_empresa}}-propuesta.md`.

El documento debe tener exactamente esta estructura:

```
# Propuesta de Agente: {{nombre_empresa}}
Fecha: [fecha actual]
Plantilla: [nombre de plantilla]
Estado: PENDIENTE DE APROBACIÓN

---

## ⚠️ Observaciones del Auditor (leer antes de aprobar)
[Si hay problemas BLOQUEANTES, listarlos aquí con severidad. Si no hay, escribir "Sin observaciones bloqueantes."]

---

## Resumen Ejecutivo
- Plantilla seleccionada y justificación
- Propuesta de valor para el cliente final
- KPIs esperados (al menos 3)

---

## Blueprint del Agente
### Rol y Título
### Narrativa
### Conocimientos Habilitados
### Límites Explícitos
### Memoria Operativa (Variables)

---

## Instrucciones del Agente
[PROMPT COMPLETO EN TEXTO PLANO — listo para copiar a Agents Studio]

---

## Workflow / Journey
### Etapa 1 — Descubrimiento
### Etapa 2 — Resolución
### Etapa 3 — Validación
### Etapa 4 — Cierre

---

## Stages para Agents Studio
[Tabla completa]

---

## Triggers
[Tabla completa]

---

## Variables Declaradas
[Tabla completa en formato {{variable}}]

---

## Reglas de Escalación y Cierre
### Condiciones de agent_transfer
### Condiciones de end_call
### Casos especiales

---

## Herramientas Externas Requeridas
[Conexiones necesarias vía tool_connections_upsert, o "Ninguna"]

---

## Checklist de Validación
- [ ] Blueprint completo con narrativa y límites claros
- [ ] Prompt en texto plano, sin markdown, segunda persona, presente indicativo
- [ ] Todas las variables en formato {{variable}}, ninguna en [brackets]
- [ ] Estilo conversacional: una pregunta a la vez, sin enumeraciones
- [ ] Stages y triggers configurados y coherentes con el journey
- [ ] Protocolo de end_call y agent_transfer definidos
- [ ] Sin información sensible hardcodeada (claves, tokens, contraseñas)
- [ ] Observaciones del Auditor revisadas y resueltas o aceptadas
```

---

## FASE 5 — Confirmación del usuario

Muestra al usuario:
1. Un resumen de 5-7 puntos clave del diseño propuesto.
2. El listado de observaciones del Auditor con su severidad.
3. La ruta donde se guardó el documento markdown.

Luego pregunta con AskUserQuestion:

**Pregunta:** ¿Apruebas este diseño para darlo de alta en Agents Studio?
- Opción A: "Sí, publicar ahora" → procede a Fase 6
- Opción B: "Sí, pero primero quiero ajustar algo" → pide qué ajustar, actualiza el documento y vuelve a confirmar
- Opción C: "No, cancelar" → detén el proceso y confirma que el documento quedó guardado para revisión posterior

**REGLA CRÍTICA**: No ejecutes ninguna herramienta de Agents Studio antes de recibir aprobación explícita.

---

## FASE 6 — Alta en Agents Studio

Ejecuta la siguiente secuencia en orden estricto. No avances al siguiente paso si el anterior falla.

```
PASO 1: catalog_templates_list
→ Localiza el ID de la plantilla seleccionada.
→ Si no encuentras la plantilla exacta, consulta con el usuario antes de continuar.

PASO 2: agents_fork_from_template(template_id)
→ Nombre del agente: "{{nombre_empresa}} - {{tipo_plantilla}} - v1"
→ Guarda el agent_id devuelto.

PASO 3: agent_versions_create(agent_id)
→ Guarda el version_id devuelto.

PASO 4: agent_version_instructions_create(version_id, prompt)
→ Inserta el prompt completo del documento (sección "Instrucciones del Agente").

PASO 5: agent_stages_create (uno por cada stage de la tabla)
→ Crea los stages en el orden definido en el documento.
→ Guarda el stage_id de cada uno.

PASO 6: agent_stage_triggers_create (uno por cada trigger de la tabla)
→ Usa los stage_id del paso anterior para referenciar origen y destino.

PASO 7: tool_connections_upsert (solo si se definieron herramientas externas)
→ Conecta CRM, sistema de pagos, calendario, etc.

PASO 8: Antes de publicar, pregunta al usuario:
"¿Confirmas que quieres publicar esta versión del agente ahora?"
→ Si confirma: agent_versions_publish(version_id)
→ Si no: informa que el agente quedó creado en borrador y puede publicarse manualmente.
```

Al terminar, reporta:
- ID del agente creado
- Nombre del agente
- Estado (publicado / borrador)
- Ruta del documento markdown guardado
