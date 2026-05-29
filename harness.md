# Harness — Arquitectura Personal de IA

> Documento vivo. Cada decisión de arquitectura se registra aquí con su razonamiento.
> Los principios operativos destilados viven en `~/.claude/CLAUDE.md`.
> Los agentes viven en `~/.claude/agents/`.

---

## La metáfora

El modelo de IA es el caballo — provee la fuerza computacional. El harness son las riendas
y los arneses: la arquitectura que dirige esa fuerza hacia resultados concretos y predecibles.

**Principio rector:** simplicidad sobre complejidad. Un harness sobrecargado es peor que
ninguno. Cada componente que se agrega debe justificar su existencia con claridad.

---

## Los tres pilares

```text
┌─────────────────────────────────────────────────────┐
│                    HARNESS                          │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │  PILAR 1     │  │  PILAR 2     │  │  PILAR 3  │  │
│  │  Contexto    │◄─┤  Orquesta-   ├─►│  Verifica-│  │
│  │  y Memoria   │  │  ción        │  │  ción     │  │
│  └──────┬───────┘  └──────┬───────┘  └─────┬─────┘  │
│         │                 │                │         │
│         └─────────────────┴────────────────┘         │
│              (los tres se retroalimentan)            │
└─────────────────────────────────────────────────────┘
```

Los pilares no son independientes: la verificación alimenta la memoria, la memoria
informa al orquestador, el orquestador determina qué verificar.

---

## Pilar 1 — Contexto y Memoria

### Estado actual: ✅ Base funcional

El sistema de memoria persistente en `~/.claude/projects/.../memory/` captura:

| Tipo | Qué guarda | Ejemplo |
|---|---|---|
| `user` | Perfil, rol, preferencias | "Estudiante de IA, prefiere prosa + tablas" |
| `feedback` | Correcciones y validaciones | "Incluir tablas de propiedades en código" |
| `project` | Contexto del trabajo actual | "Curso Anthropic, foco en arquitectura" |
| `reference` | Dónde encontrar información externa | "recurrencias.md como memoria del curso" |

### Lo que falta: ❌ Memoria de errores

Un agente dedicado a capturar errores con formato estructurado:

```text
Error memory entry:
- Tarea:      qué se intentaba hacer
- Agente:     cuál falló o estuvo involucrado
- Error:      qué pasó exactamente
- Solución:   qué lo resolvió
- Aprendizaje: qué debería hacerse diferente la próxima vez
- Contador:   N errores acumulados desde el último review
```

### Decisión: niveles de aprendizaje

| Nivel | Qué hace | Estado |
|---|---|---|
| **1** | Guarda error + solución en memoria para la próxima sesión | A implementar |
| **2** | Review automático al llegar a N errores → propone updates a agentes | A implementar (después del 1) |
| **3** | Los agentes se reescriben solos | ❌ Nunca — inestabilidad |

**Threshold de review (Nivel 2):** inicialmente N = 10 errores acumulados. Ajustable.
Cuando se alcanza, el error memory agent agrega una flag `REVIEW TRIGGERED` en su
archivo de memoria. En la próxima sesión, Claude la detecta y propone la revisión.
El usuario siempre aprueba los cambios — el sistema informa, no decide.

---

## Pilar 2 — Orquestación

### Estado actual: ✅ Parcial (meta-agentes, sin orquestador)

Tenemos agentes que **crean** herramientas, pero no un agente que **gestione** tareas:

| Agente | Rol | Tipo |
|---|---|---|
| `agent-architect` | Diseña y crea nuevos subagentes | Meta-agente (crea herramientas) |
| `skill-architect` | Diseña y crea nuevas skills | Meta-agente (crea herramientas) |
| `mcp-architect` | Investiga e instala servidores MCP | Meta-agente (crea herramientas) |

El routing actual es *pasivo*: las descripciones de los agentes activan el correcto
según keywords. No hay nadie que descomponga una tarea compleja y la secuencie.

### Lo que falta: ❌ Agente orquestador

**Decisión de arquitectura:** un solo agente con dos fases, no dos agentes separados.
El motivo: el planner necesita conocer los agentes disponibles para armar un plan
coherente. Separarlo del dispatcher crea coordinación innecesaria y puntos de falla.

```text
ORCHESTRATOR — dos fases

FASE 1: PLAN
  ├── Recibe la tarea compleja
  ├── La descompone en subtareas ordenadas
  ├── Asigna un agente a cada subtarea
  ├── Estima qué modelo conviene para cada una
  ├── Muestra el plan completo al usuario
  └── → PAUSA OBLIGATORIA
        "¿Aprobás este plan o querés ajustar algo?"

FASE 2: DISPATCH (solo tras aprobación explícita)
  ├── Ejecuta subtarea 1 → delega al agente asignado
  ├── Recibe el resultado
  ├── Ejecuta subtarea 2 → siguiente agente
  ├── ...
  └── Resumen final de lo ejecutado + flag para verificación
```

La pausa entre fases es la decisión de diseño más importante: evita que el sistema
encadene agentes en la dirección equivocada. El usuario siempre aprueba el plan antes
de que empiece cualquier ejecución.

---

## Pilar 3 — Verificación

### Estado actual: ✅ Parcial (skills bajo demanda)

Las skills `verify`, `review` y `security-review` existen pero son manuales.
No hay verificación automática al finalizar una tarea.

### Lo que falta: ❌ Agente verificador

Un agente de verificación que se dispare al terminar el dispatch del orquestador:

```text
VERIFICATION AGENT

Entrada: resultado del orquestador + tipo de tarea
Proceso:
  ├── Tests automatizados según el tipo de tarea
  │     ├── Código: scripts de verificación, Playwright (UI)
  │     ├── Configuración: scripts init/smoke
  │     └── Documentación: estructura y links
  ├── Diagnóstico: ¿qué falló y por qué?
  └── Resultado:
        ├── ✅ OK → cierra la tarea
        └── ❌ Error → dispara error memory agent (no repite el loop ciegamente)
```

**Principio clave:** el error rompe el loop, no lo reinicia automáticamente.
El agente verifica, diagnostica y reporta — pero no intenta arreglarlo solo
indefinidamente. La solución pasa por el humano o por el Nivel 2 de memoria.

---

## Arquitectura completa del harness

```text
USUARIO
  │
  ▼
ORCHESTRATOR (~/.claude/agents/orchestrator.md)
  │
  ├── FASE 1: descompone y planifica
  │     └── → aprobación del usuario
  │
  └── FASE 2: dispatch
        ├── agent-architect    (si necesita crear un agente)
        ├── skill-architect    (si necesita crear una skill)
        ├── mcp-architect      (si necesita integración externa)
        ├── [agentes del proyecto]  (workers específicos)
        └── → VERIFICATION AGENT
                  │
                  ├── ✅ OK → cierra
                  └── ❌ Error → ERROR MEMORY AGENT
                                    └── contador++
                                    └── si contador == N → flag REVIEW TRIGGERED
```

---

## Jerarquía de archivos

```text
~/.claude/
├── CLAUDE.md                  ← principios operativos globales (siempre activos)
├── harness.md                 ← [futuro] versión condensada operativa
└── agents/
    ├── agent-architect.md     ✅ existe
    ├── skill-architect.md     ✅ existe
    ├── mcp-architect.md       ✅ existe
    ├── orchestrator.md        ❌ por crear
    ├── verification-agent.md  ❌ por crear
    └── error-memory-agent.md  ❌ por crear

[cada proyecto]/
├── .claude/
│   ├── CLAUDE.md              ← hereda global + overrides del proyecto
│   └── agents/                ← agentes específicos del proyecto (si necesita)
└── ...
```

---

## Decisiones de arquitectura registradas

| # | Decisión | Alternativa descartada | Por qué |
|---|---|---|---|
| 1 | Un orquestador con dos fases | Dos agentes separados (planner + dispatcher) | El planner necesita conocer los agentes disponibles; separarlos crea coordinación innecesaria |
| 2 | Nivel 1 de memoria primero | Implementar los tres niveles a la vez | Complejidad progresiva; necesitamos datos reales antes de diseñar el Nivel 2 |
| 3 | Threshold N para review (Nivel 2) | Review manual sin trigger | Sistematiza la cadencia; N es ajustable con experiencia |
| 4 | Nivel 3 nunca | Agentes que se reescriben solos | Inestabilidad impredecible; el humano aprueba los cambios siempre |
| 5 | Error rompe el loop, no lo reinicia | Retry automático infinito | Evita loops ciegos; el diagnóstico pasa al humano o al review |

---

## Roadmap — qué construir y en qué orden

```text
FASE A — Fundamentos (ahora)
  ├── [✅] agent-architect
  ├── [✅] skill-architect
  ├── [✅] mcp-architect
  ├── [✅] sistema de memoria (user, feedback, project, reference)
  └── [✅] orchestrator.md

FASE B — Cierre del loop
  ├── [✅] error-memory-agent.md
  └── [✅] verification-agent.md

FASE C — Nivel 2 de aprendizaje
  └── [✅] error-review-agent.md — lee log, identifica patrones, propone updates

FASE D — Harness por proyecto
  ├── [✅] ~/.claude/CLAUDE.md — principios operativos globales (siempre activos)
  └── [✅] ~/.claude/templates/project-claude.md — template para proyectos nuevos
```

---

## Sesión 2026-05-26 — Lo que construimos

Partimos de cero y llegamos a un harness funcional en una sola sesión.

**El punto de partida:** tres meta-agentes existentes (`agent-architect`, `skill-architect`,
`mcp-architect`) y un sistema de memoria pasiva. Sin orquestación, sin verificación,
sin ciclo de aprendizaje.

**Lo que se definió conceptualmente:**
- La metáfora del arnés: el modelo es el caballo, la arquitectura son las riendas
- Los tres pilares (memoria, orquestación, verificación) y cómo se retroalimentan
- El principio rector: el humano siempre aprueba antes de que el sistema ejecute o modifique

**Lo que se construyó:**

| Archivo | Fase | Qué hace |
|---|---|---|
| `orchestrator.md` | A | Agente principal. Descompone tareas complejas, presenta un plan, espera aprobación y luego despacha a los agentes correctos |
| `error-memory-agent.md` | B | Registra errores con formato estructurado. Lleva un contador y agrega `REVIEW TRIGGERED` al llegar a N=10 |
| `verification-agent.md` | B | Verifica que una tarea realmente funcionó. En fallo: detiene el loop, diagnostica y reporta al error-memory-agent |
| `error-review-agent.md` | C | Lee el log de errores acumulados, identifica patrones y propone cambios concretos a los agentes afectados. Nunca modifica sin aprobación |
| `~/.claude/CLAUDE.md` | D | Principios operativos globales. Se carga automáticamente en cualquier proyecto |
| `templates/project-claude.md` | D | Template para iniciar un proyecto nuevo con el harness ya configurado |

**Las decisiones clave que no son obvias:**
- El orquestador es un solo agente con dos fases, no dos agentes separados
- Los errores rompen el loop, no lo reinician automáticamente
- El Nivel 3 (agentes que se reescriben solos) se descartó por inestabilidad
- El threshold N=10 es ajustable — es un punto de partida, no una constante

**El loop completo, de principio a fin:**
```text
Tarea → Orquestador → plan → aprobación → dispatch
                                               ↓
                                     Verification agent
                                      ✅ OK → cierra
                                      ❌ falla → error-memory-agent (N++)
                                                   N=10 → REVIEW TRIGGERED
                                                              ↓
                                                   error-review-agent
                                                   patrones → propuestas → aprobás → aplica
```

---

## Changelog

| Fecha | Cambio |
|---|---|
| 2026-05-26 | Harness completo — 4 fases, 7 archivos creados, loop de automejora funcional |
