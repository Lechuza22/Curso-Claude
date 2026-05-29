# Dominio 3 — Claude Code Configuration & Workflows

> **Peso en el examen: 20%**
> Task statements: 3.1 al 3.6

---

## 3.1 Jerarquía de CLAUDE.md

### Niveles de configuración

```
~/.claude/CLAUDE.md          ← Usuario (NO se comparte por VCS)
    └── aplica solo a ese usuario en cualquier proyecto

.claude/CLAUDE.md            ← Proyecto (se comparte por VCS)
    └── aplica a todo el equipo cuando trabajan en este proyecto

src/api/CLAUDE.md            ← Directorio (se comparte por VCS)
    └── aplica solo cuando se trabaja en ese directorio
```

**Trampa frecuente del examen:** Si un nuevo miembro del equipo no recibe ciertas instrucciones, probablemente están en `~/.claude/CLAUDE.md` (nivel usuario) en lugar de en el nivel de proyecto.

### Sintaxis @import

Permite mantener CLAUDE.md modular y reutilizable:

```markdown
# CLAUDE.md del proyecto

@import .claude/rules/testing.md
@import .claude/rules/api-conventions.md
@import docs/architecture/decisions.md

## Estándares universales
- Siempre escribir tests para lógica de negocio
- Seguir los patrones del ADR-001 para decisiones arquitectónicas
```

Ventaja: los mantenedores de cada módulo pueden importar solo las reglas relevantes a su área.

### .claude/rules/ — organización por tema

En lugar de un CLAUDE.md monolítico, archivos enfocados:

```
.claude/rules/
├── testing.md          → convenciones de tests
├── api-conventions.md  → endpoints, respuestas, errores
├── deployment.md       → guías de deploy
└── security.md         → requisitos de seguridad
```

### Diagnóstico con /memory

El comando `/memory` muestra qué archivos de configuración están cargados en la sesión actual. Útil para depurar comportamiento inconsistente:

```
/memory
→ Loaded: ~/.claude/CLAUDE.md
→ Loaded: .claude/CLAUDE.md
→ Loaded: src/api/CLAUDE.md  (porque estás editando en src/api/)
→ Loaded: .claude/rules/testing.md (importado desde .claude/CLAUDE.md)
```

---

## 3.2 Slash Commands y Skills

### Diferencia clave

| Mecanismo | Propósito | Cuándo usarlo |
|-----------|-----------|---------------|
| CLAUDE.md | Estándares siempre cargados | Convenciones universales del proyecto |
| Slash commands | Flujos de trabajo específicos bajo demanda | `/review`, `/deploy-checklist`, `/generate-tests` |
| Skills | Flujos con configuración avanzada (fork, tools restringidas) | Análisis verbose, exploración exploratorias |

### Alcance de slash commands

```
.claude/commands/review.md        ← Proyecto (compartido por VCS, disponible para todo el equipo)
~/.claude/commands/my-review.md   ← Usuario (personal, no compartido)
```

### Skills en .claude/skills/

Las skills usan un archivo SKILL.md con frontmatter YAML:

```markdown
---
name: analyze-codebase
description: Análisis profundo de la arquitectura del codebase
context: fork
allowed-tools: Read, Grep, Glob
argument-hint: "Nombre del módulo a analizar (ej: src/api)"
---

Analizá el módulo especificado en profundidad:
1. Mapeá todas las dependencias externas
2. Identificá patrones arquitectónicos
3. Detectá code smells y deuda técnica
4. Generá un reporte estructurado
```

### Opciones de frontmatter SKILL.md

| Campo | Valores | Efecto |
|-------|---------|--------|
| `context: fork` | `fork` / (omitido) | Fork: la skill corre en subagente aislado, output no contamina la sesión |
| `allowed-tools` | Lista de tools | Restringe qué tools puede usar la skill |
| `argument-hint` | String descriptivo | Se muestra al usuario si invoca la skill sin argumentos |

### Cuándo usar context: fork

```
Skill de análisis de codebase → context: fork ✅
(Produce output verbose que llenaría el contexto principal)

Skill de lluvia de ideas → context: fork ✅
(Output exploratorio que no debe "contaminar" el contexto de trabajo)

Skill de generación de un archivo → NO fork ❌
(El resultado debe estar en el contexto principal)
```

---

## 3.3 Reglas con Scope de Ruta

### Problema que resuelven

Las convenciones de tests están **distribuidas** a lo largo del codebase:
- `src/components/Button.test.tsx`
- `src/api/users.test.ts`
- `tests/integration/auth.test.ts`

Un archivo CLAUDE.md en un subdirectorio solo aplica en ese directorio. Con reglas path-scoped, aplican en todos los archivos que coincidan con el patrón, independientemente de la ubicación.

### Sintaxis

```markdown
---
paths:
  - "**/*.test.tsx"
  - "**/*.test.ts"
  - "**/*.spec.ts"
---

# Convenciones de Tests

- Usar describe/it de Jest/Vitest
- Mockear dependencias externas con vi.mock()
- Cada test debe tener arrange/act/assert claramente diferenciados
- Nombres descriptivos: "should [do X] when [condition Y]"
```

```markdown
---
paths:
  - "terraform/**/*"
---

# Convenciones de Terraform

- Usar snake_case para nombres de recursos
- Agregar tags obligatorios: environment, team, cost_center
- No hardcodear secrets — usar variables o data sources
```

### Tabla de decisión: reglas path-scoped vs. CLAUDE.md en subdirectorio

| Situación | Usar |
|-----------|------|
| Convención aplica a archivos distribuidos en múltiples directorios | `.claude/rules/` con glob |
| Convención aplica solo a un directorio específico y sus hijos | CLAUDE.md en subdirectorio |
| Convención de tests en toda la app | `.claude/rules/testing.md` con `**/*.test.*` |
| Convención específica de un módulo | CLAUDE.md en ese módulo |

---

## 3.4 Plan Mode vs. Ejecución Directa

### Plan mode

Permite explorar y diseñar antes de comprometerse con cambios. Claude mapea el codebase, evalúa dependencias, propone el enfoque y espera aprobación antes de modificar archivos.

**Cuándo usar plan mode:**

| Criterio | Ejemplo |
|----------|---------|
| Cambios a gran escala (muchos archivos) | Migración de librería que afecta 45+ archivos |
| Múltiples enfoques válidos | Reestructurar monolito en microservicios |
| Decisiones arquitectónicas | Elegir entre REST vs. GraphQL |
| Implicaciones de infraestructura | Agregar caching que requiere Redis |
| Refactoring profundo | Cambiar patrón de estado en toda la app |

**Cuándo usar ejecución directa:**

| Criterio | Ejemplo |
|----------|---------|
| Bug fix en un único archivo | Stack trace claro → línea exacta |
| Cambio bien entendido y delimitado | Agregar validación de fecha a un campo |
| Sin dependencias que explorar | Cambiar un string, actualizar un valor |

### Combinar ambos

```
Plan mode → explorar e investigar el codebase
     ↓ plan aprobado por el desarrollador
Ejecución directa → implementar el enfoque planificado
```

### Subagente Explore

Para fases de descubrimiento verbose que llenarían el contexto principal:

```
Sin Explore:
[contexto principal] ← se llena con logs de exploración
                     ← menos espacio para la implementación

Con Explore:
Subagente Explore → investiga, devuelve resumen
[contexto principal] ← recibe solo el resumen compacto
                     ← context window preservado para la tarea real
```

---

## 3.5 Refinamiento Iterativo

### Técnicas por problema

| Problema | Técnica |
|----------|---------|
| Instrucciones en prosa dan resultados inconsistentes | Agregar 2-3 ejemplos concretos de input/output |
| Comportamiento esperado difícil de describir | Escribir tests primero, iterar compartiendo los fallos |
| Dominio desconocido con consideraciones ocultas | Patrón de entrevista: Claude pregunta antes de implementar |
| Múltiples issues que interactúan entre sí | Reportar todos juntos en un mensaje detallado |
| Issues independientes | Iterar secuencialmente, uno a la vez |

### Ejemplos de input/output (la técnica más efectiva)

```markdown
Transformá los datos de entrada al formato de salida.

Ejemplo 1:
Input:  {"fecha": "2024-01-15", "monto": "1.500,00"}
Output: {"date": "2024-01-15", "amount": 1500.00}

Ejemplo 2:
Input:  {"fecha": "15/01/2024", "monto": "$1,500.00"}
Output: {"date": "2024-01-15", "amount": 1500.00}

Ejemplo 3:
Input:  {"fecha": null, "monto": "N/A"}
Output: {"date": null, "amount": null}
```

### Patrón de entrevista

```
Usuario: "Implementá un sistema de caché para las queries de usuario"

Claude (modo entrevista): Antes de implementar, necesito entender:
1. ¿Qué TTL tienen los datos de usuario?
2. ¿Cómo se invalida el caché cuando se actualiza un perfil?
3. ¿Qué pasa si el caché y la DB están desincronizados?
4. ¿El caché debe ser compartido entre instancias o local a cada instancia?

→ Las respuestas revelan consideraciones que el desarrollador no anticipó
```

### Issues paralelos vs. secuenciales

```python
# Issues que INTERACTÚAN → reportar juntos
"""
Hay tres problemas que interactúan:
1. La función validate() no maneja null (causa el error en 2)
2. El error de validate() rompe el flujo de process() (causa el problema 3)  
3. process() no tiene try/catch, así que el error llega hasta la UI

Arreglalos como un conjunto coherente.
"""

# Issues INDEPENDIENTES → iterar secuencialmente
# Primero: "Arreglá la validación de email en register()"
# Luego: "Ahora arreglá el manejo de timeout en fetch()"
```

---

## 3.6 Claude Code en CI/CD

### Flags esenciales para CI

| Flag | Propósito |
|------|-----------|
| `-p` / `--print` | Modo no-interactivo: procesa, imprime a stdout, sale |
| `--output-format json` | Output en JSON parseable por máquinas |
| `--json-schema <schema>` | Fuerza que el output siga un schema JSON |

```bash
# En el pipeline CI
claude -p "Review this PR for security issues" \
  --output-format json \
  --json-schema ./schemas/review-output.json \
  < diff.txt
```

Si no se usa `-p`, Claude Code espera input interactivo → el job se cuelga indefinidamente.

### CLAUDE.md en CI

El archivo CLAUDE.md provee contexto a Claude Code cuando es invocado por CI:

```markdown
# CLAUDE.md (incluye sección para CI)

## Para revisiones de código automatizadas

Criterios de revisión:
- Reportar solo bugs confirmados y vulnerabilidades de seguridad
- Ignorar problemas de estilo que no afectan el comportamiento
- Para cada hallazgo: archivo, línea, descripción, severidad (HIGH/MEDIUM/LOW)

## Tests disponibles
Los fixtures están en `tests/fixtures/`. No sugerir escenarios de test ya cubiertos.
```

### Evitar comentarios duplicados en PRs

```bash
# Incluir revisiones anteriores en el contexto
claude -p "Review only NEW issues not in the previous review.
Previous review findings: $(cat previous_review.json)
Current diff: $(cat current.diff)"
```

### Aislamiento de sesión para revisión

Una misma sesión que generó el código es **menos efectiva** para revisarlo después:
- Retiene el razonamiento que llevó a las decisiones
- Es menos propensa a cuestionar sus propias elecciones

Para revisión efectiva → usar una **instancia independiente** sin contexto del código generado.

---

## Mapa Conceptual del Dominio 3

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 50, 'rankSpacing': 70}}}%%
flowchart LR
    D3((Dominio 3\nClaude Code\nConfig))
    
    D3 --> HIER[Jerarquía CLAUDE.md]
    HIER --> USER[~/.claude/ → usuario]
    HIER --> PROJ[.claude/ → proyecto/VCS]
    HIER --> DIR[Subdirectorio → local]
    HIER --> IMP[@import modular]
    
    D3 --> CMD[Comandos & Skills]
    CMD --> SCOPE[.claude/commands/ compartido\n~/.claude/commands/ personal]
    CMD --> FORK[context: fork\naislamiento]
    CMD --> AT[allowed-tools\nrestricción]
    
    D3 --> RULES[Path Rules]
    RULES --> GLOB[Patrones glob en frontmatter]
    RULES --> DIST[Archivos distribuidos]
    
    D3 --> MODE[Plan vs. Directo]
    MODE --> PLAN[Plan: arquitectónico\nmulti-archivo]
    MODE --> DIRECT[Directo: bug fix\nalcance claro]
    
    D3 --> CI[CI/CD]
    CI --> P[-p modo no-interactivo]
    CI --> JSON[--output-format json]
```

---

## Preguntas Clave para Repasar

1. Un nuevo dev del equipo no ve las instrucciones de CLAUDE.md. ¿Cuál es la causa más probable?
2. ¿Cuál es la diferencia entre `.claude/commands/` y `~/.claude/commands/`?
3. ¿Para qué sirve `context: fork` en una skill?
4. ¿Cuándo usarías reglas con path glob en `.claude/rules/` en lugar de un CLAUDE.md en un subdirectorio?
5. ¿Qué flag hace que Claude Code no se cuelgue en un pipeline CI?
6. ¿Por qué una sesión que generó código es menos efectiva para revisarlo?
7. ¿Para qué se usa `/memory` en Claude Code?
