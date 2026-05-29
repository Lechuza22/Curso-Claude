# Guía de Certificación — Claude Certified Architect: Foundations

> Fuente: Anthropic — Claude Certified Architect – Foundations Certification Exam Guide
> Traducido al español para estudio personal

---

## Introducción

La certificación **Claude Certified Architect – Foundations** valida que los profesionales pueden tomar decisiones informadas sobre tradeoffs al implementar soluciones reales con Claude. Este examen evalúa conocimiento fundacional sobre Claude Code, el Claude Agent SDK, la Claude API y el Model Context Protocol (MCP) — las tecnologías centrales para construir aplicaciones de nivel productivo con Claude.

Las preguntas del examen están basadas en escenarios realistas extraídos de casos de uso reales de clientes, incluyendo: construcción de sistemas agénticos para soporte al cliente, diseño de pipelines de investigación multi-agente, integración de Claude Code en flujos de trabajo CI/CD, construcción de herramientas de productividad para desarrolladores, y extracción de datos estructurados de documentos no estructurados. Los candidatos deben demostrar no solo conocimiento conceptual sino también juicio práctico sobre arquitectura, configuración y tradeoffs en despliegues productivos.

Esta guía describe el contenido del examen, lista los dominios y task statements evaluados, provee preguntas de muestra y recomienda estrategias de preparación. Usala junto con experiencia práctica para prepararte de forma efectiva.

---

## Descripción del Candidato Ideal

El candidato ideal para esta certificación es un arquitecto de soluciones que diseña e implementa aplicaciones productivas con Claude. Este candidato tiene experiencia práctica en:

- Construcción de aplicaciones agénticas usando el **Claude Agent SDK**, incluyendo orquestación multi-agente, delegación a subagentes, integración de tools y lifecycle hooks
- Configuración y personalización de **Claude Code** para flujos de trabajo de equipo usando archivos CLAUDE.md, Agent Skills, integraciones de servidores MCP y plan mode
- Diseño de interfaces de tools y recursos de **Model Context Protocol (MCP)** para integración con sistemas backend
- Ingeniería de prompts que producen **salida estructurada confiable**, aprovechando JSON schemas, few-shot examples y patrones de extracción
- Gestión efectiva del **context window** en documentos extensos, conversaciones multi-turno y handoffs multi-agente
- Integración de Claude en **pipelines CI/CD** para revisión automatizada de código, generación de tests y feedback en pull requests
- Toma de decisiones sólidas sobre **escalación y confiabilidad**, incluyendo manejo de errores, flujos de trabajo human-in-the-loop y patrones de auto-evaluación

El candidato típicamente tiene 6+ meses de experiencia práctica construyendo con las Claude APIs, el Agent SDK, Claude Code y MCP, comprendiendo tanto las capacidades como las limitaciones de los modelos de lenguaje de gran escala en entornos productivos.

---

## Contenido del Examen

### Tipos de Respuesta

Todas las preguntas del examen son de opción múltiple. Cada pregunta tiene una respuesta correcta y tres respuestas incorrectas (distractores).

Seleccioná la única respuesta que mejor completa el enunciado o responde la pregunta. Los distractores son opciones de respuesta que un candidato con conocimiento o experiencia incompleta podría elegir.

Las preguntas sin respuesta se puntúan como incorrectas; no hay penalidad por adivinar.

### Resultados del Examen

El examen tiene una designación de aprobado o reprobado. Se puntúa contra un estándar mínimo establecido por expertos en la materia.

Los resultados se reportan como un puntaje escalado de 100 a 1.000. El puntaje mínimo de aprobación es **720**. Los modelos de puntaje escalado ayudan a equiparar puntajes entre múltiples formas del examen que pueden tener niveles de dificultad ligeramente distintos.

---

## Outline de Contenidos

Este exam guide incluye pesos, dominios de contenido y task statements para el examen. El examen tiene los siguientes dominios de contenido y pesos:

| Dominio | Tema | Porcentaje del contenido evaluado |
|---------|------|-----------------------------------|
| Dominio 1 | Agentic Architecture & Orchestration | 27% |
| Dominio 2 | Tool Design & MCP Integration | 18% |
| Dominio 3 | Claude Code Configuration & Workflows | 20% |
| Dominio 4 | Prompt Engineering & Structured Output | 20% |
| Dominio 5 | Context Management & Reliability | 15% |

---

## Escenarios del Examen

El examen usa preguntas basadas en escenarios. Cada escenario presenta un contexto productivo realista que enmarca un conjunto de preguntas. Durante el examen, se presentarán 4 escenarios seleccionados al azar del conjunto completo de los 6 escenarios siguientes.

### Escenario 1: Agente de Resolución de Soporte al Cliente

Estás construyendo un agente de resolución de soporte al cliente usando el Claude Agent SDK. El agente maneja solicitudes de alta ambigüedad como devoluciones, disputas de facturación y problemas de cuenta. Tiene acceso a tus sistemas backend a través de tools de Model Context Protocol (MCP) personalizadas (`get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`). Tu objetivo es 80%+ de resolución en el primer contacto, sabiendo cuándo escalar.

**Dominios principales:** Agentic Architecture & Orchestration, Tool Design & MCP Integration, Context Management & Reliability

### Escenario 2: Generación de Código con Claude Code

Estás usando Claude Code para acelerar el desarrollo de software. Tu equipo lo usa para generación de código, refactoring, debugging y documentación. Necesitás integrarlo en tu flujo de trabajo de desarrollo con slash commands personalizados, configuraciones CLAUDE.md, y entender cuándo usar plan mode versus ejecución directa.

**Dominios principales:** Claude Code Configuration & Workflows, Context Management & Reliability

### Escenario 3: Sistema de Investigación Multi-Agente

Estás construyendo un sistema de investigación multi-agente usando el Claude Agent SDK. Un agente coordinador delega a subagentes especializados: uno busca en la web, uno analiza documentos, uno sintetiza hallazgos y uno genera reportes. El sistema investiga temas y produce reportes comprehensivos con citas.

**Dominios principales:** Agentic Architecture & Orchestration, Tool Design & MCP Integration, Context Management & Reliability

### Escenario 4: Productividad del Desarrollador con Claude

Estás construyendo herramientas de productividad para desarrolladores usando el Claude Agent SDK. El agente ayuda a los ingenieros a explorar codebases desconocidas, entender sistemas legacy, generar código boilerplate y automatizar tareas repetitivas. Usa las tools integradas (Read, Write, Bash, Grep, Glob) y se integra con servidores de Model Context Protocol (MCP).

**Dominios principales:** Tool Design & MCP Integration, Claude Code Configuration & Workflows, Agentic Architecture & Orchestration

### Escenario 5: Claude Code para Integración Continua

Estás integrando Claude Code en tu pipeline de Integración Continua/Despliegue Continuo (CI/CD). El sistema ejecuta revisiones de código automatizadas, genera casos de prueba y proporciona feedback sobre pull requests. Necesitás diseñar prompts que proporcionen feedback accionable y minimicen falsos positivos.

**Dominios principales:** Claude Code Configuration & Workflows, Prompt Engineering & Structured Output

### Escenario 6: Extracción de Datos Estructurados

Estás construyendo un sistema de extracción de datos estructurados usando Claude. El sistema extrae información de documentos no estructurados, valida la salida usando schemas de JavaScript Object Notation (JSON) y mantiene alta precisión. Debe manejar casos edge con elegancia e integrarse con sistemas downstream.

**Dominios principales:** Prompt Engineering & Structured Output, Context Management & Reliability

---

## Dominio 1: Agentic Architecture & Orchestration

### Task Statement 1.1: Diseñar e implementar agentic loops para la ejecución autónoma de tareas

**Conocimiento de:**
- El ciclo de vida del agentic loop: enviar solicitudes a Claude, inspeccionar `stop_reason` (`"tool_use"` vs `"end_turn"`), ejecutar las tools solicitadas y devolver resultados para la siguiente iteración
- Cómo los resultados de las tools se agregan al historial de conversación para que el modelo pueda razonar sobre la siguiente acción
- La distinción entre toma de decisiones guiada por el modelo (Claude razona qué tool llamar a continuación según el contexto) y árboles de decisión pre-configurados o secuencias fijas de tools

**Habilidades en:**
- Implementar el control de flujo del agentic loop que continúa cuando `stop_reason` es `"tool_use"` y termina cuando `stop_reason` es `"end_turn"`
- Agregar resultados de tools al contexto de conversación entre iteraciones para que el modelo pueda incorporar nueva información en su razonamiento
- Evitar anti-patrones como: parsear señales en lenguaje natural para determinar la terminación del loop, establecer límites de iteración arbitrarios como mecanismo primario de parada, o verificar el contenido de texto del asistente como indicador de completitud

---

### Task Statement 1.2: Orquestar sistemas multi-agente con patrones coordinador-subagente

**Conocimiento de:**
- Arquitectura hub-and-spoke donde un agente coordinador gestiona toda la comunicación inter-subagente, el manejo de errores y el enrutamiento de información
- Cómo los subagentes operan con contexto aislado — no heredan automáticamente el historial de conversación del coordinador
- El rol del coordinador en la descomposición de tareas, delegación, agregación de resultados, y la decisión de qué subagentes invocar según la complejidad de la consulta
- Riesgos de la descomposición de tareas demasiado estrecha por parte del coordinador, lo que lleva a cobertura incompleta de temas de investigación amplios

**Habilidades en:**
- Diseñar agentes coordinadores que analicen los requisitos de la consulta y seleccionen dinámicamente qué subagentes invocar, en lugar de enrutar siempre por el pipeline completo
- Particionar el alcance de la investigación entre subagentes para minimizar duplicación (por ejemplo, asignando subtemas o tipos de fuentes distintos a cada agente)
- Implementar loops de refinamiento iterativo donde el coordinador evalúa la salida de síntesis en busca de vacíos, re-delega a subagentes de búsqueda y análisis con consultas específicas, y re-invoca síntesis hasta que la cobertura sea suficiente
- Enrutar toda la comunicación de subagentes a través del coordinador para observabilidad, manejo de errores consistente y flujo de información controlado

---

### Task Statement 1.3: Configurar invocación de subagentes, paso de contexto y spawning

**Conocimiento de:**
- La tool `Task` como mecanismo para spawnear subagentes, y el requisito de que `allowedTools` debe incluir `"Task"` para que un coordinador pueda invocar subagentes
- Que el contexto del subagente debe ser proporcionado explícitamente en el prompt — los subagentes no heredan automáticamente el contexto del padre ni comparten memoria entre invocaciones
- La configuración `AgentDefinition` incluyendo descripciones, system prompts y restricciones de tools para cada tipo de subagente
- Gestión de sesiones basada en fork para explorar enfoques divergentes desde una base de análisis compartida

**Habilidades en:**
- Incluir hallazgos completos de agentes previos directamente en el prompt del subagente (por ejemplo, pasando resultados de búsqueda web y análisis de documentos al subagente de síntesis)
- Usar formatos de datos estructurados para separar contenido de metadatos (URLs de fuentes, nombres de documentos, números de página) al pasar contexto entre agentes para preservar la atribución
- Spawnear subagentes en paralelo emitiendo múltiples llamadas a la tool `Task` en una única respuesta del coordinador en lugar de en turnos separados
- Diseñar prompts de coordinador que especifiquen objetivos de investigación y criterios de calidad en lugar de instrucciones procedurales paso a paso, para habilitar la adaptabilidad del subagente

---

### Task Statement 1.4: Implementar flujos de trabajo multi-paso con patrones de enforcement y handoff

**Conocimiento de:**
- La diferencia entre enforcement programático (hooks, prerequisite gates) y guía basada en prompts para el ordenamiento del flujo de trabajo
- Cuando se requiere cumplimiento determinístico (por ejemplo, verificación de identidad antes de operaciones financieras), las instrucciones en el prompt tienen una tasa de fallo no nula
- Protocolos de handoff estructurados para escalación a mitad de proceso que incluyen detalles del cliente, análisis de causa raíz y acciones recomendadas

**Habilidades en:**
- Implementar prerequisitos programáticos que bloquean llamadas a tools downstream hasta que los pasos prerequisito se hayan completado (por ejemplo, bloqueando `process_refund` hasta que `get_customer` haya retornado un ID de cliente verificado)
- Descomponer solicitudes de clientes con múltiples problemas en ítems distintos, luego investigar cada uno en paralelo usando contexto compartido antes de sintetizar una resolución unificada
- Compilar resúmenes de handoff estructurados (ID de cliente, causa raíz, monto de reembolso, acción recomendada) al escalar a agentes humanos que no tienen acceso al transcript de la conversación

---

### Task Statement 1.5: Aplicar hooks del Agent SDK para intercepción de tool calls y normalización de datos

**Conocimiento de:**
- Patrones de hooks (por ejemplo, `PostToolUse`) que interceptan resultados de tools para transformarlos antes de que el modelo los procese
- Patrones de hooks que interceptan llamadas salientes a tools para hacer cumplir reglas de cumplimiento (por ejemplo, bloqueando reembolsos por encima de un umbral)
- La distinción entre usar hooks para garantías determinísticas versus depender de instrucciones en el prompt para cumplimiento probabilístico

**Habilidades en:**
- Implementar hooks `PostToolUse` para normalizar formatos de datos heterogéneos (timestamps Unix, ISO 8601, códigos de estado numéricos) de diferentes tools MCP antes de que el agente los procese
- Implementar hooks de intercepción de tool calls que bloqueen acciones que violan políticas (por ejemplo, reembolsos que excedan $500) y redirijan a flujos de trabajo alternativos (por ejemplo, escalación humana)
- Elegir hooks sobre enforcement basado en prompts cuando las reglas de negocio requieren cumplimiento garantizado

---

### Task Statement 1.6: Diseñar estrategias de descomposición de tareas para flujos de trabajo complejos

**Conocimiento de:**
- Cuándo usar pipelines secuenciales fijos (prompt chaining) versus descomposición adaptativa dinámica basada en hallazgos intermedios
- Patrones de prompt chaining que dividen revisiones en pasos secuenciales (por ejemplo, analizar cada archivo individualmente, luego ejecutar un pase de integración cross-file)
- El valor de planes de investigación adaptativos que generan subtareas basadas en lo que se descubre en cada paso

**Habilidades en:**
- Seleccionar patrones de descomposición de tareas apropiados para el flujo de trabajo: prompt chaining para revisiones multi-aspecto predecibles, descomposición dinámica para tareas de investigación de extremo abierto
- Dividir revisiones de código grandes en pases de análisis local por archivo más un pase de integración cross-file separado para evitar la dilución de atención
- Descomponer tareas de extremo abierto (por ejemplo, "agregar tests comprehensivos a un codebase legacy") mapeando primero la estructura, identificando áreas de alto impacto, luego creando un plan priorizado que se adapta a medida que se descubren dependencias

---

### Task Statement 1.7: Gestionar estado de sesión, reanudación y forking

**Conocimiento de:**
- Reanudación de sesiones nombradas usando `--resume <session-name>` para continuar una conversación previa específica
- `fork_session` para crear ramas independientes desde una base de análisis compartida para explorar enfoques divergentes
- La importancia de informar al agente sobre cambios en archivos previamente analizados al reanudar sesiones después de modificaciones de código
- Por qué iniciar una nueva sesión con un resumen estructurado es más confiable que reanudar con resultados de tools obsoletos

**Habilidades en:**
- Usar `--resume` con nombres de sesión para continuar sesiones de investigación nombradas a través de sesiones de trabajo
- Usar `fork_session` para crear ramas de exploración paralelas (por ejemplo, comparando dos estrategias de testing o enfoques de refactoring desde un análisis de codebase compartido)
- Elegir entre reanudación de sesión (cuando el contexto previo es mayormente válido) y comenzar desde cero con resúmenes inyectados (cuando los resultados previos de tools están obsoletos)
- Informar a una sesión reanudada sobre cambios específicos en archivos para re-análisis dirigido en lugar de requerir re-exploración completa

---

## Dominio 2: Tool Design & MCP Integration

### Task Statement 2.1: Diseñar interfaces de tools efectivas con descripciones claras y límites definidos

**Conocimiento de:**
- Las descripciones de tools como el mecanismo principal que los LLMs usan para la selección de tools; las descripciones mínimas llevan a selección poco confiable entre tools similares
- La importancia de incluir formatos de entrada, consultas de ejemplo, casos edge y explicaciones de límites en las descripciones de tools
- Cómo las descripciones de tools ambiguas o superpuestas causan mal enrutamiento (por ejemplo, `analyze_content` vs `analyze_document` con descripciones casi idénticas)
- El impacto del wording del system prompt en la selección de tools: instrucciones sensibles a palabras clave pueden crear asociaciones de tools no deseadas

**Habilidades en:**
- Escribir descripciones de tools que diferencien claramente el propósito de cada tool, entradas y salidas esperadas, y cuándo usarla versus alternativas similares
- Renombrar tools y actualizar descripciones para eliminar superposición funcional (por ejemplo, renombrando `analyze_content` a `extract_web_results` con una descripción específica para la web)
- Dividir tools genéricas en tools de propósito específico con contratos de entrada/salida definidos (por ejemplo, dividir un `analyze_document` genérico en `extract_data_points`, `summarize_content` y `verify_claim_against_source`)
- Revisar system prompts en busca de instrucciones sensibles a palabras clave que puedan anular descripciones de tools bien escritas

---

### Task Statement 2.2: Implementar respuestas de error estructuradas para tools MCP

**Conocimiento de:**
- El patrón del flag `isError` de MCP para comunicar fallos de tools de vuelta al agente
- La distinción entre errores transitorios (timeouts, indisponibilidad del servicio), errores de validación (entrada inválida), errores de negocio (violaciones de política) y errores de permisos
- Por qué las respuestas de error uniformes (un genérico "Operation failed") impiden que el agente tome decisiones de recuperación apropiadas
- La diferencia entre errores reintentables y no reintentables, y cómo devolver metadatos estructurados previene intentos de retry desperdiciados

**Habilidades en:**
- Devolver metadatos de error estructurados incluyendo `errorCategory` (transient/validation/permission), booleano `isRetryable` y descripciones legibles por humanos
- Incluir flags `retriable: false` y explicaciones amigables para el cliente en violaciones de reglas de negocio para que el agente pueda comunicarse apropiadamente
- Implementar recuperación de errores local dentro de subagentes para fallos transitorios, propagando al coordinador solo errores que no se pueden resolver localmente junto con resultados parciales y lo que se intentó
- Distinguir entre fallos de acceso (que necesitan decisiones de retry) y resultados vacíos válidos (que representan consultas exitosas sin coincidencias)

---

### Task Statement 2.3: Distribuir tools apropiadamente entre agentes y configurar tool choice

**Conocimiento de:**
- El principio de que darle a un agente acceso a demasiadas tools (por ejemplo, 18 en lugar de 4-5) degrada la confiabilidad de selección de tools al aumentar la complejidad de decisión
- Por qué los agentes con tools fuera de su especialización tienden a usarlas incorrectamente (por ejemplo, un agente de síntesis intentando búsquedas web)
- Acceso de tools con alcance: darle a los agentes solo las tools necesarias para su rol, con tools cross-rol limitadas para necesidades específicas de alta frecuencia
- Opciones de configuración de `tool_choice`: `"auto"`, `"any"` y selección forzada de tool (`{"type": "tool", "name": "..."}`)

**Habilidades en:**
- Restringir el conjunto de tools de cada subagente a las relevantes para su rol, previniendo el mal uso entre especializaciones
- Reemplazar tools genéricas con alternativas con restricciones (por ejemplo, reemplazando `fetch_url` con `load_document` que valida URLs de documentos)
- Proveer tools cross-rol con alcance para necesidades de alta frecuencia (por ejemplo, una tool `verify_fact` para el agente de síntesis) mientras se enrutan casos complejos a través del coordinador
- Usar selección forzada de `tool_choice` para asegurar que se llame a una tool específica primero (por ejemplo, forzando `extract_metadata` antes de tools de enriquecimiento), luego procesando pasos subsiguientes en turnos de seguimiento
- Establecer `tool_choice: "any"` para garantizar que el modelo llame a una tool en lugar de devolver texto conversacional

---

### Task Statement 2.4: Integrar servidores MCP en Claude Code y flujos de trabajo de agentes

**Conocimiento de:**
- Alcance de servidores MCP: nivel de proyecto (`.mcp.json`) para herramientas compartidas del equipo vs nivel de usuario (`~/.claude.json`) para servidores personales/experimentales
- Expansión de variables de entorno en `.mcp.json` (por ejemplo, `${GITHUB_TOKEN}`) para gestión de credenciales sin comprometer secretos
- Que las tools de todos los servidores MCP configurados se descubren al momento de la conexión y están disponibles simultáneamente para el agente
- Los recursos MCP como mecanismo para exponer catálogos de contenido (por ejemplo, resúmenes de issues, jerarquías de documentación, schemas de base de datos) para reducir llamadas exploratorias a tools

**Habilidades en:**
- Configurar servidores MCP compartidos en `.mcp.json` con alcance de proyecto con expansión de variables de entorno para tokens de autenticación
- Configurar servidores MCP personales/experimentales en `~/.claude.json` con alcance de usuario
- Mejorar las descripciones de tools MCP para explicar capacidades y salidas en detalle, previniendo que el agente prefiera tools integradas (como Grep) sobre tools MCP más capaces
- Elegir servidores MCP de la comunidad existentes sobre implementaciones personalizadas para integraciones estándar (por ejemplo, Jira), reservando servidores personalizados para flujos de trabajo específicos del equipo
- Exponer catálogos de contenido como recursos MCP para darles a los agentes visibilidad del dato disponible sin requerir llamadas exploratorias a tools

---

### Task Statement 2.5: Seleccionar y aplicar tools integradas (Read, Write, Edit, Bash, Grep, Glob) efectivamente

**Conocimiento de:**
- **Grep** para búsqueda de contenido (buscando en el contenido de archivos patrones como nombres de funciones, mensajes de error o sentencias import)
- **Glob** para coincidencia de patrones en rutas de archivo (encontrando archivos por nombre o patrones de extensión)
- **Read/Write** para operaciones completas con archivos; **Edit** para modificaciones específicas usando coincidencia de texto único
- Cuando Edit falla por coincidencias de texto no únicas, usar Read + Write como fallback para modificaciones de archivos confiables

**Habilidades en:**
- Seleccionar Grep para buscar contenido de código a través de un codebase (por ejemplo, encontrar todos los llamadores de una función, localizar mensajes de error)
- Seleccionar Glob para encontrar archivos que coincidan con patrones de nombres (por ejemplo, `**/*.test.tsx`)
- Usar Read para cargar contenido completo del archivo seguido de Write cuando Edit no puede encontrar texto ancla único
- Construir comprensión del codebase de forma incremental: comenzando con Grep para encontrar puntos de entrada, luego usando Read para seguir imports y trazar flujos, en lugar de leer todos los archivos desde el principio
- Trazar el uso de funciones a través de módulos wrapper identificando primero todos los nombres exportados, luego buscando cada nombre a través del codebase

---

## Dominio 3: Claude Code Configuration & Workflows

### Task Statement 3.1: Configurar archivos CLAUDE.md con jerarquía apropiada, alcance y organización modular

**Conocimiento de:**
- La jerarquía de configuración de CLAUDE.md: nivel de usuario (`~/.claude/CLAUDE.md`), nivel de proyecto (`.claude/CLAUDE.md` o CLAUDE.md en la raíz), y nivel de directorio (archivos CLAUDE.md en subdirectorios)
- Que las configuraciones de nivel de usuario solo aplican a ese usuario — las instrucciones en `~/.claude/CLAUDE.md` no se comparten con compañeros de equipo mediante control de versiones
- La sintaxis `@import` para referenciar archivos externos y mantener CLAUDE.md modular (por ejemplo, importando archivos de estándares específicos relevantes para cada paquete)
- El directorio `.claude/rules/` para organizar archivos de reglas por tema como alternativa a un CLAUDE.md monolítico

**Habilidades en:**
- Diagnosticar problemas de jerarquía de configuración (por ejemplo, un nuevo miembro del equipo que no recibe instrucciones porque están en configuración de nivel usuario en lugar de nivel proyecto)
- Usar `@import` para incluir selectivamente archivos de estándares relevantes en el CLAUDE.md de cada paquete basándose en el conocimiento de dominio del mantenedor
- Dividir archivos CLAUDE.md grandes en archivos enfocados por tema en `.claude/rules/` (por ejemplo, `testing.md`, `api-conventions.md`, `deployment.md`)
- Usar el comando `/memory` para verificar qué archivos de memoria están cargados y diagnosticar comportamiento inconsistente entre sesiones

---

### Task Statement 3.2: Crear y configurar slash commands personalizados y skills

**Conocimiento de:**
- Comandos con alcance de proyecto en `.claude/commands/` (compartidos mediante control de versiones) vs comandos con alcance de usuario en `~/.claude/commands/` (personales)
- Skills en `.claude/skills/` con archivos SKILL.md que soportan configuración frontmatter incluyendo `context: fork`, `allowed-tools` y `argument-hint`
- La opción frontmatter `context: fork` para ejecutar skills en un contexto de sub-agente aislado, previniendo que las salidas de las skills contaminen la conversación principal
- Personalización de skills personales: crear variantes personales en `~/.claude/skills/` con nombres distintos para evitar afectar a los compañeros de equipo

**Habilidades en:**
- Crear slash commands con alcance de proyecto en `.claude/commands/` para disponibilidad en todo el equipo mediante control de versiones
- Usar `context: fork` para aislar skills que producen salida verbose (por ejemplo, análisis de codebase) o contexto exploratorio (por ejemplo, lluvia de ideas de alternativas) de la sesión principal
- Configurar `allowed-tools` en el frontmatter de la skill para restringir el acceso a tools durante la ejecución de la skill (por ejemplo, limitando a operaciones de escritura de archivos para prevenir acciones destructivas)
- Usar el frontmatter `argument-hint` para solicitar a los desarrolladores los parámetros requeridos cuando invocan la skill sin argumentos
- Elegir entre skills (invocación bajo demanda para flujos de trabajo específicos de tareas) y CLAUDE.md (estándares universales siempre cargados)

---

### Task Statement 3.3: Aplicar reglas específicas por ruta para carga condicional de convenciones

**Conocimiento de:**
- Archivos `.claude/rules/` con campos `paths` en frontmatter YAML que contienen patrones glob para activación condicional de reglas
- Cómo las reglas con alcance de ruta se cargan solo cuando se editan archivos coincidentes, reduciendo el contexto irrelevante y el uso de tokens
- La ventaja de las reglas con patrones glob sobre los archivos CLAUDE.md a nivel de directorio para convenciones que abarcan múltiples directorios (por ejemplo, archivos de test distribuidos a lo largo de un codebase)

**Habilidades en:**
- Crear archivos `.claude/rules/` con scope de ruta en frontmatter YAML (por ejemplo, `paths: ["terraform/**/*"]`) para que las reglas se carguen solo cuando se editan archivos coincidentes
- Usar patrones glob en reglas específicas por ruta para aplicar convenciones a archivos por tipo independientemente de la ubicación del directorio (por ejemplo, `**/*.test.tsx` para todos los archivos de test)
- Elegir reglas específicas por ruta sobre archivos CLAUDE.md en subdirectorios cuando las convenciones deben aplicarse a archivos distribuidos a lo largo del codebase

---

### Task Statement 3.4: Determinar cuándo usar plan mode versus ejecución directa

**Conocimiento de:**
- Plan mode está diseñado para tareas complejas que involucran cambios a gran escala, múltiples enfoques válidos, decisiones arquitectónicas y modificaciones multi-archivo
- La ejecución directa es apropiada para cambios simples y bien delimitados (por ejemplo, agregar una única verificación de validación a una función)
- Plan mode habilita exploración y diseño seguros del codebase antes de comprometerse con cambios, previniendo retrabajos costosos
- El subagente Explore para aislar la salida verbose de descubrimiento y devolver resúmenes para preservar el contexto de la conversación principal

**Habilidades en:**
- Seleccionar plan mode para tareas con implicaciones arquitectónicas (por ejemplo, reestructuración de microservicios, migraciones de librerías que afectan 45+ archivos, elección entre enfoques de integración con diferentes requisitos de infraestructura)
- Seleccionar ejecución directa para cambios bien entendidos con alcance claro (por ejemplo, un bug fix en un único archivo con un stack trace claro, agregar un condicional de validación de fecha)
- Usar el subagente Explore para fases de descubrimiento verbose para prevenir el agotamiento del context window durante tareas multi-fase
- Combinar plan mode para investigación con ejecución directa para implementación (por ejemplo, planificando una migración de librería, luego ejecutando el enfoque planificado)

---

### Task Statement 3.5: Aplicar técnicas de refinamiento iterativo para mejora progresiva

**Conocimiento de:**
- Ejemplos concretos de entrada/salida como la forma más efectiva de comunicar transformaciones esperadas cuando las descripciones en prosa se interpretan de forma inconsistente
- Iteración guiada por tests: escribir suites de tests primero, luego iterar compartiendo fallos de tests para guiar la mejora progresiva
- El patrón de entrevista: hacer que Claude haga preguntas para descubrir consideraciones que el desarrollador puede no haber anticipado antes de implementar
- Cuándo proporcionar todos los problemas en un único mensaje (problemas interactivos) versus arreglarlos secuencialmente (problemas independientes)

**Habilidades en:**
- Proporcionar 2-3 ejemplos concretos de entrada/salida para aclarar requisitos de transformación cuando las descripciones en lenguaje natural producen resultados inconsistentes
- Escribir suites de tests que cubran comportamiento esperado, casos edge y requisitos de rendimiento antes de la implementación, luego iterar compartiendo fallos de tests
- Usar el patrón de entrevista para descubrir consideraciones de diseño (por ejemplo, estrategias de invalidación de caché, modos de fallo) antes de implementar soluciones en dominios desconocidos
- Proporcionar casos de test específicos con entrada de ejemplo y salida esperada para corregir el manejo de casos edge (por ejemplo, valores null en scripts de migración)
- Abordar múltiples problemas interactivos en un único mensaje detallado cuando los fixes interactúan, versus iteración secuencial para problemas independientes

---

### Task Statement 3.6: Integrar Claude Code en pipelines CI/CD

**Conocimiento de:**
- El flag `-p` (o `--print`) para ejecutar Claude Code en modo no-interactivo en pipelines automatizados
- Los flags CLI `--output-format json` y `--json-schema` para hacer cumplir salida estructurada en contextos CI
- CLAUDE.md como mecanismo para proveer contexto del proyecto (estándares de testing, convenciones de fixtures, criterios de revisión) a Claude Code invocado por CI
- Aislamiento de contexto de sesión: por qué la misma sesión de Claude que generó el código es menos efectiva en revisar sus propios cambios en comparación con una instancia de revisión independiente

**Habilidades en:**
- Ejecutar Claude Code en CI con el flag `-p` para prevenir cuelgues por entrada interactiva
- Usar `--output-format json` con `--json-schema` para producir hallazgos estructurados parseables por máquinas para publicación automatizada como comentarios inline en PRs
- Incluir hallazgos de revisiones previas en el contexto al re-ejecutar revisiones después de nuevos commits, instruyendo a Claude a reportar solo issues nuevos o aún no resueltos para evitar comentarios duplicados
- Proveer archivos de test existentes en el contexto para que la generación de tests evite sugerir escenarios duplicados ya cubiertos por la suite de tests
- Documentar estándares de testing, criterios de tests valiosos y fixtures disponibles en CLAUDE.md para mejorar la calidad de generación de tests y reducir salida de tests de bajo valor

---

## Dominio 4: Prompt Engineering & Structured Output

### Task Statement 4.1: Diseñar prompts con criterios explícitos para mejorar la precisión y reducir falsos positivos

**Conocimiento de:**
- La importancia de criterios explícitos sobre instrucciones vagas (por ejemplo, "marcar comentarios solo cuando el comportamiento afirmado contradice el comportamiento real del código" vs "verificar que los comentarios sean precisos")
- Cómo las instrucciones generales como "ser conservador" o "reportar solo hallazgos de alta confianza" no logran mejorar la precisión en comparación con criterios categóricos específicos
- El impacto de las tasas de falsos positivos en la confianza del desarrollador: las categorías de alto índice de falsos positivos socavan la confianza en categorías precisas

**Habilidades en:**
- Escribir criterios de revisión específicos que definan qué issues reportar (bugs, seguridad) versus cuáles omitir (estilo menor, patrones locales) en lugar de depender de filtrado basado en confianza
- Deshabilitar temporalmente categorías de alto índice de falsos positivos para restaurar la confianza del desarrollador mientras se mejoran los prompts para esas categorías
- Definir criterios de severidad explícitos con ejemplos de código concretos para cada nivel de severidad para lograr clasificación consistente

---

### Task Statement 4.2: Aplicar few-shot prompting para mejorar la consistencia y calidad de la salida

**Conocimiento de:**
- Los few-shot examples como la técnica más efectiva para lograr salida consistentemente formateada y accionable cuando las instrucciones detalladas solas producen resultados inconsistentes
- El rol de los few-shot examples en demostrar el manejo de casos ambiguos (por ejemplo, selección de tools para solicitudes ambiguas, brechas de cobertura de tests a nivel de rama)
- Cómo los few-shot examples habilitan al modelo a generalizar el juicio a patrones novedosos en lugar de coincidir solo con casos pre-especificados
- La efectividad de los few-shot examples para reducir alucinaciones en tareas de extracción (por ejemplo, manejo de medidas informales, estructuras de documentos variadas)

**Habilidades en:**
- Crear 2-4 few-shot examples dirigidos para escenarios ambiguos que muestren el razonamiento de por qué se eligió una acción sobre alternativas plausibles
- Incluir few-shot examples que demuestren el formato de salida deseado específico (ubicación, issue, severidad, fix sugerido) para lograr consistencia
- Proveer few-shot examples que distingan patrones de código aceptables de issues genuinos para reducir falsos positivos mientras se habilita la generalización
- Usar few-shot examples para demostrar el manejo correcto de estructuras de documentos variadas (citas inline vs bibliografías, secciones de metodología vs detalles embebidos)
- Agregar few-shot examples que muestren extracción correcta de documentos con formatos variados para abordar la extracción vacía/null de campos requeridos

---

### Task Statement 4.3: Hacer cumplir salida estructurada usando tool use y JSON schemas

**Conocimiento de:**
- El uso de tools (`tool_use`) con JSON schemas como el enfoque más confiable para salida estructurada garantizada con cumplimiento de schema, eliminando errores de sintaxis JSON
- La distinción entre `tool_choice: "auto"` (el modelo puede devolver texto en lugar de llamar a una tool), `"any"` (el modelo debe llamar a una tool pero puede elegir cuál), y selección forzada de tool (el modelo debe llamar a una tool específica nombrada)
- Que los JSON schemas estrictos mediante tool use eliminan errores de sintaxis pero no previenen errores semánticos (por ejemplo, líneas de ítems que no suman el total, valores en campos incorrectos)
- Consideraciones de diseño de schema: campos requeridos vs opcionales, patrones enum con `"other"` + string de detalle para categorías extensibles

**Habilidades en:**
- Definir tools de extracción con JSON schemas como parámetros de entrada y extraer datos estructurados de la respuesta `tool_use`
- Establecer `tool_choice: "any"` para garantizar salida estructurada cuando existen múltiples schemas de extracción y el tipo de documento es desconocido
- Forzar una tool específica con `tool_choice: {"type": "tool", "name": "extract_metadata"}` para asegurar que una extracción particular se ejecute antes de los pasos de enriquecimiento
- Diseñar campos de schema como opcionales (nullable) cuando los documentos fuente pueden no contener la información, previniendo que el modelo fabrique valores para satisfacer campos requeridos
- Agregar valores enum como `"unclear"` para casos ambiguos y campos `"other"` + detalle para categorización extensible
- Incluir reglas de normalización de formato en los prompts junto con schemas de salida estrictos para manejar formateo fuente inconsistente

---

### Task Statement 4.4: Implementar loops de validación, retry y feedback para calidad de extracción

**Conocimiento de:**
- Retry-con-feedback-de-error: agregar errores de validación específicos al prompt en el retry para guiar al modelo hacia la corrección
- Los límites del retry: los retries son inefectivos cuando la información requerida simplemente está ausente del documento fuente (vs errores de formato o estructura)
- Diseño de feedback loop: rastrear qué constructos de código desencadenan hallazgos (campo `detected_pattern`) para habilitar análisis sistemático de patrones de descarte
- La diferencia entre errores de validación semántica (los valores no suman, placement incorrecto de campos) y errores de sintaxis de schema (eliminados por tool use)

**Habilidades en:**
- Implementar solicitudes de seguimiento que incluyan el documento original, la extracción fallida y errores de validación específicos para auto-corrección del modelo
- Identificar cuándo los retries serán inefectivos (por ejemplo, la información existe solo en un documento externo no provisto) versus cuándo tendrán éxito (desajustes de formato, errores estructurales de salida)
- Agregar campos `detected_pattern` a los hallazgos estructurados para habilitar el análisis de patrones de falsos positivos cuando los desarrolladores descartan hallazgos
- Diseñar flujos de validación de auto-corrección: extraer `"calculated_total"` junto con `"stated_total"` para marcar discrepancias, agregar booleanos `"conflict_detected"` para datos fuente inconsistentes

---

### Task Statement 4.5: Diseñar estrategias eficientes de procesamiento en lote

**Conocimiento de:**
- La Message Batches API: 50% de ahorro en costo, ventana de procesamiento de hasta 24 horas, sin SLA de latencia garantizada
- El procesamiento en lote es apropiado para cargas de trabajo no bloqueantes y tolerantes a latencia (reportes nocturnos, auditorías semanales, generación de tests nocturna) e inapropiado para flujos de trabajo bloqueantes (verificaciones pre-merge)
- La batch API no soporta llamadas a tools multi-turno dentro de una única solicitud (no puede ejecutar tools a mitad de solicitud y devolver resultados)
- Campos `custom_id` para correlacionar pares de solicitud/respuesta en lote

**Habilidades en:**
- Hacer coincidir el enfoque de API con los requisitos de latencia del flujo de trabajo: API síncrona para verificaciones bloqueantes pre-merge, batch API para análisis nocturno/semanal
- Calcular la frecuencia de envío de lotes basándose en restricciones de SLA (por ejemplo, ventanas de 4 horas para garantizar un SLA de 30 horas con procesamiento de lotes de 24 horas)
- Manejar fallos en lotes: reenviar solo documentos fallidos (identificados por `custom_id`) con modificaciones apropiadas (por ejemplo, dividir documentos que excedieron los límites de contexto en chunks)
- Usar refinamiento de prompts en un conjunto de muestra antes de procesar grandes volúmenes en lotes para maximizar las tasas de éxito en el primer pase y reducir costos de reenvío iterativo

---

### Task Statement 4.6: Diseñar arquitecturas de revisión multi-instancia y multi-pase

**Conocimiento de:**
- Limitaciones de la auto-revisión: un modelo retiene el contexto de razonamiento de la generación, haciéndolo menos propenso a cuestionar sus propias decisiones en la misma sesión
- Las instancias de revisión independientes (sin contexto de razonamiento previo) son más efectivas para detectar issues sutiles que las instrucciones de auto-revisión o el pensamiento extendido
- Revisión multi-pase: dividir revisiones grandes en pases de análisis local por archivo más pases de integración cross-file para evitar la dilución de atención y hallazgos contradictorios

**Habilidades en:**
- Usar una segunda instancia independiente de Claude para revisar código generado sin el contexto de razonamiento del generador
- Dividir revisiones multi-archivo grandes en pases enfocados por archivo para issues locales más pases de integración separados para análisis de flujo de datos cross-file
- Ejecutar pases de verificación donde el modelo auto-reporta confianza junto con cada hallazgo para habilitar enrutamiento calibrado de revisiones

---

## Dominio 5: Context Management & Reliability

### Task Statement 5.1: Gestionar el contexto de conversación para preservar información crítica en interacciones largas

**Conocimiento de:**
- Riesgos de la summarización progresiva: condensar valores numéricos, porcentajes, fechas y expectativas declaradas por el cliente en resúmenes vagos
- El efecto "lost in the middle": los modelos procesan de forma confiable la información al principio y al final de entradas largas pero pueden omitir hallazgos de las secciones del medio
- Cómo los resultados de tools se acumulan en el contexto y consumen tokens desproporcionadamente a su relevancia (por ejemplo, 40+ campos por lookup de orden cuando solo 5 son relevantes)
- La importancia de pasar el historial de conversación completo en solicitudes API subsiguientes para mantener coherencia conversacional

**Habilidades en:**
- Extraer hechos transaccionales (montos, fechas, números de orden, estados) en un bloque de "hechos del caso" persistente incluido en cada prompt, fuera del historial resumido
- Extraer y persistir datos de issues estructurados (IDs de orden, montos, estados) en una capa de contexto separada para sesiones multi-issue
- Recortar salidas de tools verbose a solo los campos relevantes antes de que se acumulen en el contexto (por ejemplo, manteniendo solo los campos relevantes para devoluciones de los lookups de órdenes)
- Colocar resúmenes de hallazgos clave al principio de las entradas agregadas y organizar resultados detallados con encabezados de sección explícitos para mitigar efectos de posición
- Requerir que los subagentes incluyan metadatos (fechas, ubicaciones de fuentes, contexto metodológico) en las salidas estructuradas para soportar síntesis downstream precisa
- Modificar agentes upstream para devolver datos estructurados (hechos clave, citas, puntajes de relevancia) en lugar de cadenas de contenido verbose y razonamiento cuando los agentes downstream tienen presupuestos de contexto limitados

---

### Task Statement 5.2: Diseñar patrones efectivos de escalación y resolución de ambigüedad

**Conocimiento de:**
- Triggers apropiados de escalación: solicitudes del cliente de un humano, excepciones/brechas de política (no solo casos complejos), e incapacidad de hacer progreso significativo
- La distinción entre escalar inmediatamente cuando un cliente lo solicita explícitamente versus ofrecer resolver cuando el issue es sencillo
- Por qué la escalación basada en sentimiento y los puntajes de confianza auto-reportados son proxies poco confiables para la complejidad real del caso
- Cómo múltiples coincidencias de clientes requieren aclaración (solicitando identificadores adicionales) en lugar de selección heurística

**Habilidades en:**
- Agregar criterios de escalación explícitos con few-shot examples al system prompt demostrando cuándo escalar versus resolver autónomamente
- Honrar las solicitudes explícitas de los clientes de agentes humanos inmediatamente sin intentar investigación primero
- Reconocer la frustración mientras se ofrece resolución cuando el issue está dentro de la capacidad del agente, escalando solo si el cliente reitera su preferencia
- Escalar cuando la política es ambigua o no contempla la solicitud específica del cliente (por ejemplo, igualación de precios de competidores cuando la política solo aborda ajustes en el propio sitio)
- Instruir al agente a solicitar identificadores adicionales cuando los resultados de tools devuelven múltiples coincidencias, en lugar de seleccionar basándose en heurísticas

---

### Task Statement 5.3: Implementar estrategias de propagación de errores en sistemas multi-agente

**Conocimiento de:**
- El contexto de error estructurado (tipo de fallo, consulta intentada, resultados parciales, enfoques alternativos) como habilitador de decisiones de recuperación inteligentes del coordinador
- La distinción entre fallos de acceso (timeouts que necesitan decisiones de retry) y resultados vacíos válidos (consultas exitosas sin coincidencias)
- Por qué los estados de error genéricos ("search unavailable") ocultan contexto valioso al coordinador
- Por qué suprimir errores silenciosamente (devolver resultados vacíos como éxito) o terminar flujos de trabajo completos ante fallos únicos son ambos anti-patrones

**Habilidades en:**
- Devolver contexto de error estructurado incluyendo tipo de fallo, lo que se intentó, resultados parciales y alternativas potenciales para habilitar la recuperación del coordinador
- Distinguir fallos de acceso de resultados vacíos válidos en el reporte de errores para que el coordinador pueda tomar decisiones apropiadas
- Hacer que los subagentes implementen recuperación local para fallos transitorios y solo propaguen errores que no pueden resolver, incluyendo lo que se intentó y resultados parciales
- Estructurar la salida de síntesis con anotaciones de cobertura indicando qué hallazgos están bien respaldados versus qué áreas de temas tienen brechas debido a fuentes no disponibles

---

### Task Statement 5.4: Gestionar el contexto efectivamente en la exploración de codebases grandes

**Conocimiento de:**
- Degradación del contexto en sesiones extendidas: los modelos empiezan a dar respuestas inconsistentes y referencian "patrones típicos" en lugar de clases específicas descubiertas antes
- El rol de los archivos scratchpad para persistir hallazgos clave a través de límites de contexto
- Delegación a subagentes para aislar la salida verbose de exploración mientras el agente principal coordina la comprensión de alto nivel
- Persistencia de estado estructurada para recuperación de crashes: cada agente exporta estado a una ubicación conocida, y el coordinador carga un manifiesto al reanudar

**Habilidades en:**
- Spawnear subagentes para investigar preguntas específicas (por ejemplo, "encontrar todos los archivos de test", "trazar dependencias del flujo de reembolso") mientras el agente principal preserva la coordinación de alto nivel
- Hacer que los agentes mantengan archivos scratchpad registrando hallazgos clave, referenciándolos para preguntas subsiguientes para contrarrestar la degradación del contexto
- Resumir hallazgos clave de una fase de exploración antes de spawnear sub-agentes para la siguiente fase, inyectando resúmenes en el contexto inicial
- Diseñar recuperación de crashes usando exportaciones de estado de agentes estructuradas (manifiestos) que el coordinador carga al reanudar e inyecta en los prompts de los agentes
- Usar `/compact` para reducir el uso de contexto durante sesiones de exploración extendidas cuando el contexto se llena con salida verbose de descubrimiento

---

### Task Statement 5.5: Diseñar flujos de trabajo de revisión humana y calibración de confianza

**Conocimiento de:**
- El riesgo de que las métricas de precisión agregadas (por ejemplo, 97% general) puedan enmascarar rendimiento pobre en tipos de documentos o campos específicos
- Muestreo aleatorio estratificado para medir tasas de error en extracciones de alta confianza y detectar patrones de error novedosos
- Puntajes de confianza a nivel de campo calibrados usando conjuntos de validación etiquetados para enrutar la atención de revisión
- La importancia de validar la precisión por tipo de documento y segmento de campo antes de automatizar extracciones de alta confianza

**Habilidades en:**
- Implementar muestreo aleatorio estratificado de extracciones de alta confianza para medición continua de tasa de error y detección de patrones novedosos
- Analizar precisión por tipo de documento y campo para verificar rendimiento consistente en todos los segmentos antes de reducir la revisión humana
- Hacer que los modelos generen puntajes de confianza a nivel de campo, luego calibrar umbrales de revisión usando conjuntos de validación etiquetados
- Enrutar extracciones con baja confianza del modelo o documentos fuente ambiguos/contradictorios a revisión humana, priorizando la capacidad limitada de los revisores

---

### Task Statement 5.6: Preservar la procedencia de información y manejar incertidumbre en síntesis multi-fuente

**Conocimiento de:**
- Cómo la atribución de fuentes se pierde durante los pasos de summarización cuando los hallazgos se comprimen sin preservar los mapeos reclamo-fuente
- La importancia de los mapeos estructurados reclamo-fuente que el agente de síntesis debe preservar y fusionar al combinar hallazgos
- Cómo manejar estadísticas contradictorias de fuentes creíbles: anotar conflictos con atribución de fuente en lugar de seleccionar arbitrariamente un valor
- Datos temporales: requerir fechas de publicación/recolección en salidas estructuradas para prevenir que las diferencias temporales se malinterpreten como contradicciones

**Habilidades en:**
- Requerir que los subagentes generen mapeos estructurados reclamo-fuente (URLs de fuentes, nombres de documentos, extractos relevantes) que los agentes downstream preserven a través de la síntesis
- Estructurar reportes con secciones explícitas que distingan hallazgos bien establecidos de los cuestionados, preservando las caracterizaciones originales de las fuentes y el contexto metodológico
- Completar el análisis de documentos con valores conflictivos incluidos y explícitamente anotados, dejando que el coordinador decida cómo reconciliarlos antes de pasar a síntesis
- Requerir que los subagentes incluyan fechas de publicación o recolección de datos en salidas estructuradas para habilitar la interpretación temporal correcta
- Renderizar distintos tipos de contenido apropiadamente en las salidas de síntesis — datos financieros como tablas, noticias como prosa, hallazgos técnicos como listas estructuradas — en lugar de convertir todo a un formato uniforme

---

## Preguntas de Muestra

Las siguientes preguntas de muestra ilustran el formato y el nivel de dificultad del examen. Están extraídas del test de práctica e incluyen explicaciones para facilitar el aprendizaje.

### Escenario: Agente de Resolución de Soporte al Cliente

---

#### Pregunta 1

> Los datos de producción muestran que en el 12% de los casos, tu agente omite `get_customer` completamente y llama a `lookup_order` usando solo el nombre declarado por el cliente, lo que ocasionalmente lleva a cuentas mal identificadas y reembolsos incorrectos. ¿Qué cambio abordaría de manera más efectiva este problema de confiabilidad?

- A) Agregar un prerequisito programático que bloquee las llamadas a `lookup_order` y `process_refund` hasta que `get_customer` haya retornado un ID de cliente verificado.
- B) Mejorar el system prompt para declarar que la verificación del cliente mediante `get_customer` es obligatoria antes de cualquier operación de órdenes.
- C) Agregar few-shot examples que muestren al agente siempre llamando a `get_customer` primero, incluso cuando los clientes proporcionan detalles de la orden.
- D) Implementar un clasificador de enrutamiento que analice cada solicitud y habilite solo el subconjunto de tools apropiado para ese tipo de solicitud.

**Respuesta Correcta: A**

Cuando se requiere una secuencia específica de tools para lógica de negocio crítica (como verificar la identidad del cliente antes de procesar reembolsos), el enforcement programático proporciona garantías determinísticas que los enfoques basados en prompts no pueden ofrecer. Las opciones B y C dependen del cumplimiento probabilístico del LLM, que es insuficiente cuando los errores tienen consecuencias financieras. La opción D aborda la disponibilidad de tools en lugar del ordenamiento de tools, que no es el problema real.

---

#### Pregunta 2

> Los logs de producción muestran que el agente frecuentemente llama a `get_customer` cuando los usuarios preguntan sobre órdenes (por ejemplo, "revisar mi orden #12345"), en lugar de llamar a `lookup_order`. Ambas tools tienen descripciones mínimas ("Retrieves customer information" / "Retrieves order details") y aceptan formatos de identificador similares. ¿Cuál es el primer paso más efectivo para mejorar la confiabilidad en la selección de tools?

- A) Agregar few-shot examples al system prompt demostrando patrones correctos de selección de tools, con 5-8 ejemplos que muestren consultas relacionadas con órdenes enrutadas a `lookup_order`.
- B) Expandir la descripción de cada tool para incluir los formatos de entrada que maneja, consultas de ejemplo, casos edge y límites que explican cuándo usarla versus tools similares.
- C) Implementar una capa de enrutamiento que parsee la entrada del usuario antes de cada turno y pre-seleccione la tool apropiada basándose en palabras clave detectadas y patrones de identificador.
- D) Consolidar ambas tools en una única tool `lookup_entity` que acepte cualquier identificador y determine internamente qué backend consultar.

**Respuesta Correcta: B**

Las descripciones de tools son el mecanismo principal que los LLMs usan para la selección de tools. Cuando las descripciones son mínimas, los modelos carecen del contexto para diferenciar entre tools similares. La opción B aborda directamente esta causa raíz con un fix de bajo esfuerzo y alto impacto. Los few-shot examples (A) agregan overhead de tokens sin solucionar el problema subyacente. Una capa de enrutamiento (C) es sobre-ingeniería y evita la comprensión del lenguaje natural del LLM. Consolidar tools (D) es una elección arquitectónica válida pero requiere más esfuerzo del que justifica un "primer paso" cuando el problema inmediato es descripciones inadecuadas.

---

#### Pregunta 3

> Tu agente alcanza 55% de resolución en el primer contacto, muy por debajo del objetivo del 80%. Los logs muestran que escala casos sencillos (reemplazos estándar por daños con evidencia fotográfica) mientras intenta manejar autónomamente situaciones complejas que requieren excepciones de política. ¿Cuál es la forma más efectiva de mejorar la calibración de escalación?

- A) Agregar criterios de escalación explícitos a tu system prompt con few-shot examples que demuestren cuándo escalar versus resolver autónomamente.
- B) Hacer que el agente auto-reporte un puntaje de confianza (1-10) antes de cada respuesta y enrutar automáticamente solicitudes a humanos cuando la confianza cae por debajo de un umbral.
- C) Desplegar un modelo clasificador separado entrenado en tickets históricos para predecir qué solicitudes necesitan escalación antes de que el agente principal comience a procesar.
- D) Implementar análisis de sentimiento para detectar niveles de frustración del cliente y escalar automáticamente cuando el sentimiento negativo excede un umbral.

**Respuesta Correcta: A**

Agregar criterios de escalación explícitos con few-shot examples aborda directamente la causa raíz: límites de decisión poco claros. Esta es la respuesta proporcionada antes de agregar infraestructura. La opción B falla porque la confianza auto-reportada del LLM está mal calibrada — el agente ya tiene incorrectamente confianza en casos difíciles. La opción C es sobre-ingeniería, requiriendo datos etiquetados e infraestructura de ML cuando la optimización de prompts aún no se ha intentado. La opción D resuelve un problema completamente diferente; el sentimiento no se correlaciona con la complejidad del caso, que es el problema real.

---

### Escenario: Generación de Código con Claude Code

---

#### Pregunta 4

> Querés crear un slash command personalizado `/review` que ejecute el checklist estándar de revisión de código de tu equipo. Este comando debe estar disponible para todos los desarrolladores cuando clonan o hacen pull del repositorio. ¿Dónde deberías crear este archivo de comando?

- A) En el directorio `.claude/commands/` en el repositorio del proyecto
- B) En `~/.claude/commands/` en el directorio home de cada desarrollador
- C) En el archivo CLAUDE.md en la raíz del proyecto
- D) En un archivo `.claude/config.json` con un array de commands

**Respuesta Correcta: A**

Los slash commands personalizados con alcance de proyecto deben almacenarse en el directorio `.claude/commands/` dentro del repositorio. Estos comandos están bajo control de versiones y están automáticamente disponibles para todos los desarrolladores cuando clonan o hacen pull del repositorio. La opción B (`~/.claude/commands/`) es para comandos personales que no se comparten mediante control de versiones. La opción C (CLAUDE.md) es para instrucciones y contexto del proyecto, no para definiciones de comandos. La opción D describe un mecanismo de configuración que no existe en Claude Code.

---

#### Pregunta 5

> Te asignaron reestructurar la aplicación monolítica del equipo en microservicios. Esto involucrará cambios en docenas de archivos y requiere decisiones sobre límites de servicio y dependencias de módulos. ¿Qué enfoque deberías tomar?

- A) Entrar en plan mode para explorar el codebase, entender dependencias y diseñar un enfoque de implementación antes de hacer cambios.
- B) Comenzar con ejecución directa y hacer cambios incrementalmente, dejando que la implementación revele los límites naturales del servicio.
- C) Usar ejecución directa con instrucciones comprehensivas por adelantado detallando exactamente cómo debe estructurarse cada servicio.
- D) Comenzar en modo de ejecución directa y cambiar a plan mode solo si encontrás complejidad inesperada durante la implementación.

**Respuesta Correcta: A**

Plan mode está diseñado para tareas complejas que involucran cambios a gran escala, múltiples enfoques válidos y decisiones arquitectónicas — exactamente lo que requiere la reestructuración de monolito a microservicios. Habilita exploración y diseño seguros del codebase antes de comprometerse con cambios. La opción B arriesga retrabajos costosos cuando se descubren dependencias tardíamente. La opción C asume que ya conocés la estructura correcta sin explorar el código. La opción D ignora que la complejidad ya está declarada en los requisitos, no es algo que pueda emerger más tarde.

---

#### Pregunta 6

> Tu codebase tiene áreas distintas con diferentes convenciones de código: los componentes React usan estilo funcional con hooks, los handlers de API usan async/await con manejo de errores específico, y los modelos de base de datos siguen un patrón repository. Los archivos de test están distribuidos a lo largo del codebase junto al código que prueban (por ejemplo, `Button.test.tsx` junto a `Button.tsx`), y querés que todos los tests sigan las mismas convenciones independientemente de la ubicación. ¿Cuál es la forma más mantenible de asegurar que Claude aplique automáticamente las convenciones correctas al generar código?

- A) Crear archivos de reglas en `.claude/rules/` con frontmatter YAML especificando patrones glob para aplicar condicionalmente convenciones basadas en rutas de archivo
- B) Consolidar todas las convenciones en el archivo CLAUDE.md raíz bajo encabezados para cada área, confiando en que Claude infiera qué sección aplica
- C) Crear skills en `.claude/skills/` para cada tipo de código que incluyan las convenciones relevantes en sus archivos SKILL.md
- D) Colocar un archivo CLAUDE.md separado en cada subdirectorio que contenga las convenciones específicas de esa área

**Respuesta Correcta: A**

La opción A es correcta porque `.claude/rules/` con patrones glob (por ejemplo, `**/*.test.tsx`) permite que las convenciones se apliquen automáticamente basadas en rutas de archivo independientemente de la ubicación del directorio — esencial para archivos de test distribuidos a lo largo del codebase. La opción B depende de la inferencia en lugar de coincidencia explícita, haciéndola poco confiable. La opción C requiere invocación manual de skills o depende de que Claude elija cargarlas, contradiciendo la necesidad de aplicación "automática" determinística basada en rutas de archivo. La opción D no puede manejar fácilmente archivos distribuidos en muchos directorios ya que los archivos CLAUDE.md están ligados al directorio.

---

### Escenario: Sistema de Investigación Multi-Agente

---

#### Pregunta 7

> Después de ejecutar el sistema sobre el tema "impacto de la IA en las industrias creativas", observás que cada subagente se completa exitosamente: el agente de búsqueda web encuentra artículos relevantes, el agente de análisis de documentos resume papers correctamente, y el agente de síntesis produce salida coherente. Sin embargo, los reportes finales cubren solo las artes visuales, sin mencionar en absoluto música, escritura y producción cinematográfica. Al examinar los logs del coordinador, ves que descompuso el tema en tres subtareas: "IA en creación de arte digital", "IA en diseño gráfico" y "IA en fotografía". ¿Cuál es la causa raíz más probable?

- A) El agente de síntesis carece de instrucciones para identificar brechas de cobertura en los hallazgos que recibe de otros agentes.
- B) La descomposición de tareas del agente coordinador es demasiado estrecha, resultando en asignaciones de subagentes que no cubren todos los dominios relevantes del tema.
- C) Las consultas del agente de búsqueda web no son suficientemente comprehensivas y necesitan expandirse para cubrir más sectores de la industria creativa.
- D) El agente de análisis de documentos está filtrando fuentes relacionadas con industrias creativas no visuales debido a criterios de relevancia demasiado restrictivos.

**Respuesta Correcta: B**

Los logs del coordinador revelan la causa raíz directamente: descompuso "industrias creativas" solo en subtareas de artes visuales (arte digital, diseño gráfico, fotografía), omitiendo completamente música, escritura y cine. Los subagentes ejecutaron sus tareas asignadas correctamente — el problema es lo que se les asignó. Las opciones A, C y D culpan incorrectamente a agentes downstream que están funcionando correctamente dentro de su alcance asignado.

---

#### Pregunta 8

> El subagente de búsqueda web se agota por timeout mientras investiga un tema complejo. Necesitás diseñar cómo esta información de fallo fluye de vuelta al agente coordinador. ¿Qué enfoque de propagación de errores mejor habilita la recuperación inteligente?

- A) Devolver contexto de error estructurado al coordinador incluyendo el tipo de fallo, la consulta intentada, cualquier resultado parcial y enfoques alternativos potenciales.
- B) Implementar lógica de retry automática con backoff exponencial dentro del subagente, devolviendo un estado genérico "search unavailable" solo después de que todos los retries se agoten.
- C) Capturar el timeout dentro del subagente y devolver un conjunto de resultados vacío marcado como exitoso.
- D) Propagar la excepción de timeout directamente a un handler de nivel superior que termina todo el flujo de trabajo de investigación.

**Respuesta Correcta: A**

El contexto de error estructurado le da al coordinador la información que necesita para tomar decisiones de recuperación inteligentes — ya sea reintentar con una consulta modificada, probar un enfoque alternativo, o proceder con resultados parciales. El estado genérico de la opción B oculta contexto valioso del coordinador, impidiendo decisiones informadas. La opción C suprime el error marcando el fallo como éxito, lo que previene cualquier recuperación y arriesga salidas de investigación incompletas. La opción D termina innecesariamente todo el flujo de trabajo cuando las estrategias de recuperación podrían tener éxito.

---

#### Pregunta 9

> Durante las pruebas, observás que el agente de síntesis frecuentemente necesita verificar afirmaciones específicas mientras combina hallazgos. Actualmente, cuando se necesita verificación, el agente de síntesis devuelve el control al coordinador, que invoca al agente de búsqueda web, luego re-invoca síntesis con los resultados. Esto agrega 2-3 viajes de ida y vuelta por tarea y aumenta la latencia en un 40%. Tu evaluación muestra que el 85% de estas verificaciones son verificaciones de hechos simples (fechas, nombres, estadísticas) mientras el 15% requiere investigación más profunda. ¿Cuál es el enfoque más efectivo para reducir el overhead mientras se mantiene la confiabilidad del sistema?

- A) Darle al agente de síntesis una tool `verify_fact` con alcance para búsquedas simples, mientras las verificaciones complejas continúan delegando al agente de búsqueda web a través del coordinador.
- B) Hacer que el agente de síntesis acumule todas las necesidades de verificación y las devuelva como un lote al coordinador al final de su pase, que luego las envía todas al agente de búsqueda web de una vez.
- C) Darle al agente de síntesis acceso a todas las tools de búsqueda web para que pueda manejar cualquier necesidad de verificación directamente sin viajes de ida y vuelta a través del coordinador.
- D) Hacer que el agente de búsqueda web almacene en caché proactivamente contexto adicional alrededor de cada fuente durante la investigación inicial, anticipando lo que el agente de síntesis podría necesitar verificar.

**Respuesta Correcta: A**

La opción A aplica el principio de mínimo privilegio dándole al agente de síntesis solo lo que necesita para el caso común del 85% (verificación de hechos simples) mientras preserva el patrón de coordinación existente para casos complejos. El enfoque de batching de la opción B crea dependencias bloqueantes ya que los pasos de síntesis pueden depender de hechos verificados anteriores. La opción C sobre-provisiona al agente de síntesis, violando la separación de responsabilidades. La opción D depende de caching especulativo que no puede predecir de manera confiable lo que el agente de síntesis necesitará verificar.

---

### Escenario: Claude Code para Integración Continua

---

#### Pregunta 10

> Tu script de pipeline ejecuta `claude "Analyze this pull request for security issues"` pero el job se cuelga indefinidamente. Los logs indican que Claude Code está esperando entrada interactiva. ¿Cuál es el enfoque correcto para ejecutar Claude Code en un pipeline automatizado?

- A) Agregar el flag `-p`: `claude -p "Analyze this pull request for security issues"`
- B) Establecer la variable de entorno `CLAUDE_HEADLESS=true` antes de ejecutar el comando
- C) Redirigir stdin desde `/dev/null`: `claude "Analyze this pull request for security issues" < /dev/null`
- D) Agregar el flag `--batch`: `claude --batch "Analyze this pull request for security issues"`

**Respuesta Correcta: A**

El flag `-p` (o `--print`) es la forma documentada de ejecutar Claude Code en modo no-interactivo. Procesa el prompt, envía el resultado a stdout y sale sin esperar entrada del usuario — exactamente lo que los pipelines CI/CD requieren. Las otras opciones referencian características inexistentes (la variable de entorno `CLAUDE_HEADLESS`, el flag `--batch`) o usan workarounds Unix que no abordan apropiadamente la sintaxis de comandos de Claude Code.

---

#### Pregunta 11

> Tu equipo quiere reducir los costos de API para análisis automatizado. Actualmente, las llamadas en tiempo real de Claude potencian dos flujos de trabajo: (1) una verificación bloqueante pre-merge que debe completarse antes de que los desarrolladores puedan hacer merge, y (2) un reporte de deuda técnica generado de noche para revisión a la mañana siguiente. Tu gerente propone cambiar ambos a la Message Batches API por su ahorro del 50% en costos. ¿Cómo deberías evaluar esta propuesta?

- A) Usar procesamiento en lote solo para los reportes de deuda técnica; mantener llamadas en tiempo real para las verificaciones pre-merge.
- B) Cambiar ambos flujos de trabajo a procesamiento en lote con polling de estado para verificar la completitud.
- C) Mantener llamadas en tiempo real para ambos flujos de trabajo para evitar problemas de ordenamiento en resultados de lote.
- D) Cambiar ambos a procesamiento en lote con un fallback de timeout a tiempo real si los lotes tardan demasiado.

**Respuesta Correcta: A**

La Message Batches API ofrece 50% de ahorro en costos pero tiene tiempos de procesamiento de hasta 24 horas sin SLA de latencia garantizada. Esto la hace inadecuada para verificaciones bloqueantes pre-merge donde los desarrolladores esperan resultados, pero ideal para trabajos nocturnos como reportes de deuda técnica. La opción B es incorrecta porque depender de completitud "frecuentemente más rápida" no es aceptable para flujos de trabajo bloqueantes. La opción C refleja un malentendido — los resultados en lote pueden correlacionarse usando campos `custom_id`. La opción D agrega complejidad innecesaria cuando la solución más simple es hacer coincidir cada API con su caso de uso apropiado.

---

#### Pregunta 12

> Un pull request modifica 14 archivos en el módulo de seguimiento de acciones. Tu revisión de un solo pase analizando todos los archivos juntos produce resultados inconsistentes: feedback detallado para algunos archivos pero comentarios superficiales para otros, bugs obvios pasados por alto, y feedback contradictorio — marcando un patrón como problemático en un archivo mientras aprueba código idéntico en otro archivo del mismo PR. ¿Cómo deberías reestructurar la revisión?

- A) Dividir en pases enfocados: analizar cada archivo individualmente para issues locales, luego ejecutar un pase separado enfocado en integración examinando el flujo de datos cross-file.
- B) Requerir que los desarrolladores dividan PRs grandes en envíos más pequeños de 3-4 archivos antes de que se ejecute la revisión automatizada.
- C) Cambiar a un modelo de nivel superior con un context window más grande para darle a los 14 archivos atención adecuada en un solo pase.
- D) Ejecutar tres pases de revisión independientes en el PR completo y marcar solo los issues que aparecen en al menos dos de los tres pases.

**Respuesta Correcta: A**

Dividir las revisiones en pases enfocados aborda directamente la causa raíz: dilución de atención al procesar muchos archivos a la vez. El análisis archivo por archivo asegura profundidad consistente, mientras un pase de integración separado detecta issues cross-file. La opción B transfiere la carga a los desarrolladores sin mejorar el sistema. La opción C malentiende que los context windows más grandes no resuelven problemas de calidad de atención. La opción D suprimiría la detección de bugs reales al requerir consenso en issues que pueden detectarse solo de forma intermitente.

---

## Estrategias de Preparación

### Ejercicios de Preparación

Completá estos ejercicios prácticos para construir familiaridad práctica con los temas cubiertos en el examen. Cada ejercicio está diseñado para reforzar conocimiento en uno o más dominios del examen.

#### Ejercicio 1: Construir un Agente Multi-Tool con Lógica de Escalación

**Objetivo:** Practicar el diseño de un agentic loop con integración de tools, manejo de errores estructurado y patrones de escalación.

**Pasos:**
1. Definir 3-4 tools MCP con descripciones detalladas que claramente diferencien el propósito de cada tool, las entradas esperadas y las condiciones de límite. Incluir al menos dos tools con funcionalidad similar que requieran descripción cuidadosa para evitar confusión en la selección.
2. Implementar un agentic loop que verifique `stop_reason` para determinar si continuar la ejecución de tools o presentar la respuesta final. Manejar correctamente los stop_reason `"tool_use"` y `"end_turn"`.
3. Agregar respuestas de error estructuradas a tus tools: incluir `errorCategory` (transient/validation/permission), booleano `isRetryable` y descripciones legibles por humanos. Probar que el agente maneja cada tipo de error apropiadamente (reintentando errores transitorios, explicando errores de negocio al usuario).
4. Implementar un hook programático que intercepte llamadas a tools para hacer cumplir una regla de negocio (por ejemplo, bloqueando operaciones por encima de un monto umbral), redirigiendo a un flujo de trabajo de escalación cuando se active.
5. Probar con mensajes de múltiples problemas (por ejemplo, solicitudes que involucren múltiples issues) y verificar que el agente descompone la solicitud, maneja cada problema y sintetiza una respuesta unificada.

**Dominios reforzados:** Dominio 1 (Agentic Architecture & Orchestration), Dominio 2 (Tool Design & MCP Integration), Dominio 5 (Context Management & Reliability)

---

#### Ejercicio 2: Configurar Claude Code para un Flujo de Trabajo de Desarrollo en Equipo

**Objetivo:** Practicar la configuración de jerarquías CLAUDE.md, slash commands personalizados, reglas específicas por ruta e integración de servidores MCP para un proyecto multi-desarrollador.

**Pasos:**
1. Crear un CLAUDE.md a nivel de proyecto con estándares de código universales y convenciones de testing. Verificar que las instrucciones colocadas a nivel de proyecto se apliquen consistentemente en todos los miembros del equipo.
2. Crear archivos `.claude/rules/` con patrones glob en frontmatter YAML para diferentes áreas de código (por ejemplo, `paths: ["src/api/**/*"]` para convenciones de API, `paths: ["**/*.test.*"]` para convenciones de testing). Probar que las reglas se cargan solo cuando se editan archivos coincidentes.
3. Crear una skill con alcance de proyecto en `.claude/skills/` con `context: fork` y restricciones de `allowed-tools`. Verificar que la skill se ejecuta en aislamiento sin contaminar el contexto de conversación principal.
4. Configurar un servidor MCP en `.mcp.json` con expansión de variables de entorno para credenciales. Agregar un servidor MCP experimental personal en `~/.claude.json` y verificar que ambos están disponibles simultáneamente.
5. Probar plan mode versus ejecución directa en tareas de complejidad variable: un bug fix en un único archivo, una migración de librería multi-archivo, y una nueva feature con múltiples enfoques de implementación válidos. Observar cuándo plan mode proporciona valor.

**Dominios reforzados:** Dominio 3 (Claude Code Configuration & Workflows), Dominio 2 (Tool Design & MCP Integration)

---

#### Ejercicio 3: Construir un Pipeline de Extracción de Datos Estructurados

**Objetivo:** Practicar el diseño de JSON schemas, usar `tool_use` para salida estructurada, implementar loops de validación-retry y diseñar estrategias de procesamiento en lote.

**Pasos:**
1. Definir una tool de extracción con un JSON schema que contenga campos requeridos y opcionales, un enum con un patrón `"other"` + string de detalle, y campos nullable para información que puede no existir en los documentos fuente. Procesar documentos donde algunos campos están ausentes y verificar que el modelo devuelve null en lugar de fabricar valores.
2. Implementar un loop de validación-retry: cuando la validación Pydantic o de JSON schema falla, enviar una solicitud de seguimiento incluyendo el documento, la extracción fallida y el error de validación específico. Rastrear qué errores son resolubles mediante retry (desajustes de formato) versus cuáles no (información ausente de la fuente).
3. Agregar few-shot examples que demuestren extracción de documentos con formatos variados (por ejemplo, citas inline vs bibliografías, descripciones narrativas vs tablas estructuradas) y verificar el manejo mejorado de variedad estructural.
4. Diseñar una estrategia de procesamiento en lote: enviar un lote de 100 documentos usando la Message Batches API, manejar fallos por `custom_id`, reenviar documentos fallidos con modificaciones (por ejemplo, dividir en chunks los documentos que excedieron los límites de contexto), y calcular el tiempo total de procesamiento relativo a las restricciones de SLA.
5. Implementar una estrategia de enrutamiento de revisión humana: hacer que el modelo genere puntajes de confianza a nivel de campo, enrutar extracciones de baja confianza a revisión humana, y analizar precisión por tipo de documento y campo para verificar rendimiento consistente.

**Dominios reforzados:** Dominio 4 (Prompt Engineering & Structured Output), Dominio 5 (Context Management & Reliability)

---

#### Ejercicio 4: Diseñar y Depurar un Pipeline de Investigación Multi-Agente

**Objetivo:** Practicar la orquestación de subagentes, gestión del paso de contexto, implementación de propagación de errores y manejo de síntesis con rastreo de procedencia.

**Pasos:**
1. Construir un agente coordinador que delegue a al menos dos subagentes (por ejemplo, búsqueda web y análisis de documentos). Asegurarse de que `allowedTools` del coordinador incluya `"Task"` y que cada subagente reciba sus hallazgos de investigación directamente en su prompt en lugar de depender de herencia automática de contexto.
2. Implementar ejecución paralela de subagentes haciendo que el coordinador emita múltiples llamadas a la tool `Task` en una única respuesta. Medir la mejora de latencia comparada con la ejecución secuencial.
3. Diseñar salida estructurada para subagentes que separe contenido de metadatos: cada hallazgo debe incluir una afirmación, extracto de evidencia, URL de fuente/nombre de documento y fecha de publicación. Verificar que el subagente de síntesis preserve la atribución de fuentes al combinar hallazgos.
4. Implementar propagación de errores: simular un timeout de subagente y verificar que el coordinador recibe contexto de error estructurado (tipo de fallo, consulta intentada, resultados parciales). Probar que el coordinador puede proceder con resultados parciales y anotar la salida final con brechas de cobertura.
5. Probar con datos de fuentes conflictivos (por ejemplo, dos fuentes creíbles con estadísticas diferentes) y verificar que la salida de síntesis preserva ambos valores con atribución de fuente en lugar de seleccionar arbitrariamente uno, y estructura el reporte para distinguir hallazgos bien establecidos de los cuestionados.

**Dominios reforzados:** Dominio 1 (Agentic Architecture & Orchestration), Dominio 2 (Tool Design & MCP Integration), Dominio 5 (Context Management & Reliability)

---

## Apéndice

### Tecnologías y Conceptos

La siguiente lista contiene tecnologías y conceptos que pueden aparecer en el examen:

- **Claude Agent SDK** — Definiciones de agentes, agentic loops, manejo de `stop_reason`, hooks (`PostToolUse`, intercepción de tool calls), spawning de subagentes mediante la tool `Task`, configuración de `allowedTools`
- **Model Context Protocol (MCP)** — Servidores MCP, tools MCP, recursos MCP, flag `isError`, descripciones de tools, distribución de tools, configuración de `.mcp.json`, expansión de variables de entorno
- **Claude Code** — Jerarquía de configuración CLAUDE.md (usuario/proyecto/directorio), `.claude/rules/` con path-scoping en frontmatter YAML, `.claude/commands/` para slash commands, `.claude/skills/` con frontmatter SKILL.md (`context: fork`, `allowed-tools`, `argument-hint`), plan mode, ejecución directa, comando `/memory`, `/compact`, `--resume`, `fork_session`, subagente Explore
- **Claude Code CLI** — Flag `-p` / `--print` para modo no-interactivo, `--output-format json`, `--json-schema` para salida estructurada en CI
- **Claude API** — `tool_use` con JSON schemas, opciones de `tool_choice` (`"auto"`, `"any"`, selección forzada de tool), valores de `stop_reason` (`"tool_use"`, `"end_turn"`), `max_tokens`, system prompts
- **Message Batches API** — 50% de ahorro en costos, ventana de procesamiento de hasta 24 horas, `custom_id` para correlación solicitud/respuesta, polling para completitud, sin soporte de tool calling multi-turno
- **JSON Schema** — Campos requeridos vs opcionales, tipos enum, campos nullable, patrones `"other"` + string de detalle, modo estricto para eliminación de errores de sintaxis
- **Pydantic** — Validación de schema, errores de validación semántica, loops de validación-retry
- **Tools integradas** — Read, Write, Edit, Bash, Grep, Glob — sus propósitos y criterios de selección
- **Few-shot prompting** — Ejemplos dirigidos para escenarios ambiguos, demostración de formato, generalización a patrones novedosos
- **Prompt chaining** — Descomposición secuencial de tareas en pases enfocados
- **Gestión del context window** — Presupuestos de tokens, summarización progresiva, efectos "lost in the middle", extracción de contexto, archivos scratchpad
- **Gestión de sesiones** — Reanudación de sesiones, `fork_session`, sesiones nombradas, aislamiento de contexto de sesión
- **Puntaje de confianza** — Confianza a nivel de campo, calibración con conjuntos de validación etiquetados, muestreo estratificado para medición de tasa de error

---

### Temas En Alcance

Los siguientes temas son evaluados explícitamente en el examen:

- **Implementación de agentic loop** — Control de flujo basado en `stop_reason`, manejo de resultados de tools, condiciones de terminación del loop
- **Orquestación multi-agente** — Patrones coordinador-subagente, descomposición de tareas, ejecución paralela de subagentes, loops de refinamiento iterativo
- **Gestión de contexto de subagentes** — Paso explícito de contexto, persistencia estructurada de estado, recuperación de crashes usando manifiestos
- **Diseño de interfaces de tools** — Escribir descripciones efectivas de tools, dividir vs consolidar tools, naming de tools para reducir ambigüedad
- **Diseño de tools y recursos MCP** — Recursos para catálogos de contenido, tools para acciones, calidad de descripción para adopción
- **Configuración de servidores MCP** — Alcance de proyecto vs usuario, expansión de variables de entorno, acceso simultáneo a múltiples servidores
- **Manejo y propagación de errores** — Respuestas de error estructuradas, errores transitorios vs de negocio vs de permisos, recuperación local antes de escalación
- **Toma de decisiones de escalación** — Criterios explícitos, honrar preferencias del cliente, identificación de brechas de política
- **Configuración de CLAUDE.md** — Jerarquía (usuario/proyecto/directorio), patrones `@import`, `.claude/rules/` con patrones glob
- **Comandos y skills personalizados** — Alcance de proyecto vs usuario, `context: fork`, frontmatter `allowed-tools`, `argument-hint`
- **Plan mode vs ejecución directa** — Evaluación de complejidad, decisiones arquitectónicas, cambios en un único archivo
- **Refinamiento iterativo** — Ejemplos de entrada/salida, iteración guiada por tests, patrón de entrevista, resolución de issues secuencial vs paralela
- **Salida estructurada mediante tool_use** — Diseño de schema, configuración de `tool_choice`, campos nullable para prevenir alucinación
- **Few-shot prompting** — Targeting de escenarios ambiguos, consistencia de formato, reducción de falsos positivos
- **Procesamiento en lote** — Adecuación de la Message Batches API, evaluación de tolerancia a latencia, manejo de fallos por `custom_id`
- **Optimización del context window** — Recortar salidas de tools verbose, extracción de hechos estructurados, ordenamiento de entradas consciente de la posición
- **Flujos de trabajo de revisión humana** — Calibración de confianza, muestreo estratificado, segmentación de precisión por tipo de documento y campo
- **Procedencia de información** — Mapeos reclamo-fuente, manejo de datos temporales, anotación de conflictos, reporte de brechas de cobertura

---

### Temas Fuera de Alcance

Los siguientes temas relacionados **NO** aparecerán en el examen:

- Fine-tuning de modelos Claude o entrenamiento de modelos personalizados
- Autenticación de la Claude API, facturación o gestión de cuentas
- Implementación detallada de lenguajes de programación o frameworks específicos (más allá de lo necesario para configuración de tools y schemas)
- Despliegue u hosting de servidores MCP (infraestructura, redes, orquestación de contenedores)
- Arquitectura interna de Claude, proceso de entrenamiento o pesos del modelo
- Constitutional AI, RLHF o metodologías de entrenamiento de seguridad
- Modelos de embedding o detalles de implementación de bases de datos vectoriales
- Computer use (automatización de navegador, interacción con escritorio)
- Capacidades de análisis de visión/imágenes
- Implementación de la Streaming API o server-sent events
- Rate limiting, cuotas o cálculos de precios de API
- OAuth, rotación de API keys o detalles de protocolos de autenticación
- Configuraciones específicas de proveedores cloud (AWS, GCP, Azure)
- Benchmarking de rendimiento o métricas de comparación de modelos
- Detalles de implementación de prompt caching (más allá de saber que existe)
- Algoritmos de conteo de tokens o especificidades de tokenización

---

## Recomendaciones de Preparación para el Examen

Para prepararte para este examen de certificación:

1. **Construir un agente con el Claude Agent SDK** — Implementar un agentic loop completo con llamadas a tools, manejo de errores y gestión de sesiones. Practicar el spawning de subagentes y el paso de contexto entre ellos.

2. **Configurar Claude Code para un proyecto real** — Establecer CLAUDE.md con una jerarquía de configuración, crear reglas específicas por ruta en `.claude/rules/`, construir skills personalizadas con opciones frontmatter (`context: fork`, `allowed-tools`), e integrar al menos un servidor MCP.

3. **Diseñar y probar tools MCP** — Escribir descripciones de tools que claramente diferencien tools similares. Implementar respuestas de error estructuradas con categorías de error y flags retryable. Probar la confiabilidad de selección de tools con solicitudes ambiguas.

4. **Construir un pipeline de extracción de datos estructurados** — Usar `tool_use` con JSON schemas, implementar loops de validación-retry, diseñar schemas con campos opcionales/nullable, y practicar el procesamiento en lote con la Message Batches API.

5. **Practicar técnicas de prompt engineering** — Escribir few-shot examples para escenarios ambiguos. Definir criterios de revisión explícitos para reducir falsos positivos. Diseñar arquitecturas de revisión multi-pase para revisiones de código grandes.

6. **Estudiar patrones de gestión de contexto** — Practicar la extracción de hechos estructurados de salidas verbose de tools, implementar archivos scratchpad para sesiones largas, y diseñar delegación a subagentes para gestionar límites de contexto.

7. **Revisar patrones de escalación y human-in-the-loop** — Entender cuándo escalar (brechas de política, solicitudes del cliente, incapacidad de progresar) versus resolver autónomamente. Practicar el diseño de flujos de trabajo de revisión humana con enrutamiento basado en confianza.

8. **Completar el Examen de Práctica** — Antes de presentar el examen real, completar el examen de práctica (el enlace se proporcionará por separado). El examen de práctica cubre los mismos escenarios y formato de pregunta que el examen real y muestra explicaciones después de cada respuesta para ayudar a reforzar tu comprensión.

---

*Versión 0.1 — Última actualización: 10 de febrero de 2025*

*Anthropic, PBC — Need to Know (NTK)*
