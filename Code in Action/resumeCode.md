# Code in Action

---

<!-- Unidades y módulos se irán agregando a medida que avance el curso -->

---

## Unidad 1: ¿Qué es Claude Code?

---

### Módulo 1: ¿Qué es un asistente de codificación?

#### ¿Qué es y qué no es?

Un asistente de codificación no es un autocompletado glorificado. Es un sistema que usa un modelo de lenguaje para **razonar sobre problemas de programación** y resolverlos siguiendo el mismo proceso que usaría un desarrollador humano: entender el contexto, formular un plan y ejecutar acciones.

La diferencia con un LLM genérico es que el asistente puede **interactuar con el mundo exterior**: leer archivos, modificar código, ejecutar comandos. Un LLM sin herramientas solo puede procesar texto y devolver texto — si le pedís que lea un archivo, te dirá que no puede.

---

#### El proceso de resolución de tareas

Cuando el asistente recibe una tarea (por ejemplo, "corregí este error"), sigue tres etapas:

```text
1. Recopilar contexto
   └─ ¿A qué se refiere el error?
   └─ ¿Qué archivos están involucrados?
   └─ ¿Qué parte del código es relevante?

2. Formular un plan
   └─ ¿Cómo resolver el problema?
   └─ ¿Qué cambios hay que hacer?
   └─ ¿Cómo verificar que la solución funciona?

3. Tomar acciones
   └─ Actualizar archivos
   └─ Ejecutar comandos
   └─ Verificar resultados
```

Las etapas 1 y 3 requieren interacción con el exterior — ahí entra el uso de herramientas.

---

#### Cómo funciona el uso de herramientas

Los modelos de lenguaje solo generan texto. El "uso de herramientas" es un mecanismo que convierte ese texto en acciones reales:

```text
Vos:           "¿Qué código hay en main.go?"
                        ↓
Asistente:     agrega instrucciones al mensaje:
               "Si querés leer un archivo, respondé con 'ReadFile: nombre'"
                        ↓
Modelo:        responde → "ReadFile: main.go"
                        ↓
Asistente:     lee el archivo real del disco
                        ↓
Modelo:        recibe el contenido y genera la respuesta final
                        ↓
Vos:           ves la respuesta basada en el archivo real
```

El modelo nunca "lee" el archivo directamente. Genera texto con un formato especial, el asistente interpreta ese texto como una instrucción, ejecuta la acción y devuelve el resultado al modelo. Desde afuera parece magia; por dentro es un loop de texto bien orquestado.

---

#### Herramientas típicas de un asistente de codificación

| Herramienta | Qué hace | Ejemplo de uso |
| --- | --- | --- |
| `ReadFile` | Lee el contenido de un archivo | Entender qué hace una función |
| `WriteFile` / `Edit` | Crea o modifica archivos | Aplicar un fix |
| `RunCommand` | Ejecuta comandos en la terminal | Correr tests, compilar |
| `Search` / `Grep` | Busca patrones en el código | Encontrar dónde se usa una variable |
| `FetchURL` | Obtiene documentación externa | Consultar una API o librería |

---

#### Por qué el uso de herramientas de Claude importa

No todos los modelos usan herramientas con la misma habilidad. Claude (Opus, Sonnet, Haiku) está optimizado específicamente para esto, lo que se traduce en ventajas concretas para Claude Code:

| Ventaja | Descripción |
| --- | --- |
| **Tareas más difíciles** | Puede combinar múltiples herramientas para resolver problemas complejos, incluso con herramientas que no vio durante el entrenamiento |
| **Plataforma extensible** | Podés agregar herramientas propias y Claude se adapta para usarlas |
| **Mayor privacidad** | Navega el código sin necesidad de indexarlo — no envía todo el código a servidores externos |

El tercer punto es especialmente relevante para bases de código privadas: Claude lee solo lo que necesita para la tarea en curso, en lugar de requerir una indexación completa del repositorio.

---

#### Conclusión: qué hace que un asistente sea realmente potente

| Componente | Por qué importa |
| --- | --- |
| Modelo de lenguaje base | Razona sobre el problema y genera el plan |
| Sistema de herramientas | Conecta el razonamiento con el mundo real |
| Calidad del uso de herramientas | Determina qué tan bien se resuelven tareas complejas |

La combinación de los tres es lo que diferencia a Claude Code de un simple generador de texto con autocompletado.

---

## Unidad 2: Manos a la obra

---

### Módulo 1: Configuración de Claude Code

#### Instalación

La instalación varía según el sistema operativo. En todos los casos descarga y ejecuta un script de instalación oficial:

| Sistema | Comando |
| --- | --- |
| macOS / Linux / WSL | `curl -fsSL https://claude.ai/install.sh \| bash` |
| Windows PowerShell | `irm https://claude.ai/install.ps1 \| iex` |
| Windows CMD | `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd` |
| macOS (Homebrew) | `brew install --cask claude-code` |

Una vez instalado, ejecutá `claude` en tu terminal. La primera vez te pedirá:

1. Elegir un **tema de color** para la terminal
2. **Autenticarte** con tus credenciales de claude.ai

---

#### Problemas comunes de instalación

| Síntoma | Qué hacer |
| --- | --- |
| `claude: command not found` | Verificar que el directorio de instalación esté en el `PATH` del sistema |
| Error de red durante la instalación | Revisar firewall, proxy o VPN activos |
| Error de permisos | Ejecutar con los permisos adecuados (puede requerir `sudo` en Linux/macOS) |

Si ninguna de estas soluciones funciona, consultá la sección **"Solucionar problemas de instalación"** en la documentación oficial.

---

#### Uso con proveedores externos

Claude Code no solo funciona con la cuenta de claude.ai. También puede conectarse a modelos Claude alojados en:

| Proveedor | Cuándo usarlo |
| --- | --- |
| **Amazon Bedrock** | Entornos AWS o empresas con datos en infraestructura propia |
| **Google Cloud Vertex AI** | Entornos GCP o políticas de datos en Google Cloud |
| **Microsoft Foundry** | Entornos Azure o ecosistema Microsoft |

Cada proveedor requiere configuración adicional de credenciales. La documentación oficial tiene una sección específica de **"Configuración del proveedor externo"** para cada caso.

> Para la mayoría de los casos de uso individual y de aprendizaje, la autenticación estándar con claude.ai es suficiente.

---

### Módulo 2: Configuración del proyecto

#### El proyecto del curso: `uigen`

Para explorar Claude Code en contexto real, el curso usa un proyecto llamado **uigen** — una aplicación de generación de componentes de interfaz de usuario. No es obligatorio usarlo; podés seguir el resto del curso con tu propio código.

La app usa Claude a través de la API de Anthropic para generar componentes de UI. Si no configurás la clave API, igual funciona: genera código estático ficticio en su lugar.

---

#### Pasos de configuración

```text
1. Tener Node.js instalado localmente
2. Descomprimir uigen.zip en el directorio de trabajo
3. En el directorio del proyecto: npm run setup
   └─ instala dependencias
   └─ configura una base de datos SQLite local
4. (Opcional) Configurar la clave API de Anthropic
5. Iniciar el proyecto: npm run dev
```

---

#### Configuración de la clave API (opcional)

Si querés probar la generación real de UI con Claude en lugar del modo estático:

1. Obtené una clave API en `https://console.anthropic.com/`
2. Abrí el archivo `.env` en la raíz del proyecto
3. Reemplazá el texto `your-api-key-here` con tu clave

```bash
# .env
ANTHROPIC_API_KEY=your-api-key-here   # ← reemplazar esto
```

> Nunca commitees el archivo `.env` al repositorio. La clave API es una credencial privada — si la exponés en un repo público, cualquiera puede usarla a tu costo.

---

#### Resumen de comandos

| Comando | Qué hace |
| --- | --- |
| `npm run setup` | Instala dependencias y crea la base de datos SQLite |
| `npm run dev` | Inicia el servidor de desarrollo |

#### Decisión: proyecto propio vs. uigen

| Opción | Ventaja |
| --- | --- |
| **Usar uigen** | Proyecto conocido, los ejemplos del curso aplican directamente |
| **Usar tu propio código** | Aprendés en el contexto real de tu trabajo |

Ambas opciones son completamente válidas. Lo importante es tener un proyecto con archivos reales donde Claude Code pueda leer, editar y ejecutar comandos.

---

### Módulo 3: Añadiendo contexto

#### El problema del contexto

Un proyecto real puede tener cientos de archivos. Claude no necesita leerlos todos — necesita los **correctos**. Demasiado contexto irrelevante degrada el rendimiento: Claude pierde foco, tarda más y puede distraerse con información que no aplica a la tarea.

La gestión de contexto es la habilidad central para trabajar bien con Claude Code.

---

#### El comando `/init`

El primer paso al abrir Claude Code en un proyecto nuevo es ejecutar `/init`. Claude analiza todo el código fuente y genera un resumen con:

- El propósito y la arquitectura general del proyecto
- Comandos importantes (tests, build, dev server)
- Archivos críticos y patrones de codificación

Ese resumen se guarda en un archivo `CLAUDE.md` en la raíz del proyecto. Claude te pedirá permiso para crear el archivo:

| Tecla | Efecto |
| --- | --- |
| `Enter` | Aprueba cada operación de escritura individualmente |
| `Shift + Tab` | Autoriza a Claude a escribir archivos libremente durante toda la sesión |

---

#### El archivo `CLAUDE.md`

`CLAUDE.md` es el mecanismo de contexto persistente de Claude Code. Se incluye **automáticamente en cada solicitud** que hacés — funciona como un system prompt permanente para tu proyecto.

Tiene dos propósitos:

1. **Guiar a Claude por el código:** arquitectura, comandos, convenciones de estilo
2. **Instrucciones personalizadas:** comportamientos específicos que querés que Claude siga

Si Claude agrega demasiados comentarios al código, podés corregirlo editando `CLAUDE.md`:

```markdown
Use comments sparingly. Only comment complex code.
```

Para abrir el archivo desde Claude Code: ejecutá `/memory`.

---

#### Las tres ubicaciones de `CLAUDE.md`

| Archivo | Dónde vive | Compartido | Para qué |
| --- | --- | --- | --- |
| `CLAUDE.md` | Raíz del proyecto | Sí (se commitea) | Contexto del proyecto compartido con todo el equipo |
| `CLAUDE.local.md` | Raíz del proyecto | No (en `.gitignore`) | Instrucciones personales, preferencias propias |
| `~/.claude/CLAUDE.md` | Directorio home | No | Instrucciones globales para todos tus proyectos |

El orden de prioridad es: global → proyecto → local. Las instrucciones más específicas sobreescriben a las más generales.

---

#### Menciones con `@`

Cuando necesitás que Claude examine archivos específicos en una conversación, usás `@` seguido de la ruta:

```text
How does the auth system work? @auth
```

Claude muestra una lista de archivos relacionados con ese nombre para que elijas, y luego incluye el contenido seleccionado en la conversación. Útil cuando sabés qué archivos son relevantes para tu pregunta.

---

#### Referencias `@` dentro de `CLAUDE.md`

También podés incluir referencias `@` en el propio `CLAUDE.md`. El contenido de esos archivos se carga automáticamente en cada sesión, sin que tengas que mencionarlos cada vez:

```markdown
<!-- CLAUDE.md -->
The database schema is defined in the @prisma/schema.prisma file.
Reference it anytime you need to understand the structure of data stored in the database.
```

Esto es especialmente útil para archivos que son transversales al proyecto: esquemas de base de datos, tipos globales, configuración de la API.

---

#### Importar desde otro archivo de instrucciones

Si tu repositorio ya tiene un `AGENTS.md` (para otra herramienta como Cursor o Copilot), no necesitás duplicar las instrucciones. Agregá esto en la primera línea de `CLAUDE.md`:

```markdown
@AGENTS.md

<!-- instrucciones específicas de Claude debajo -->
Use comments sparingly.
```

Claude carga primero el contenido de `AGENTS.md` y luego aplica las instrucciones propias que agregues.

---

#### Resumen: mecanismos de contexto

| Mecanismo | Cuándo aplica | Persistencia |
| --- | --- | --- |
| `/init` | Primera vez en un proyecto nuevo | Genera `CLAUDE.md` una sola vez |
| `CLAUDE.md` | Cada sesión automáticamente | Permanente (en el repo) |
| `CLAUDE.local.md` | Cada sesión automáticamente | Permanente (solo local) |
| `~/.claude/CLAUDE.md` | Cada sesión en cualquier proyecto | Permanente (global) |
| `@archivo` en el chat | Solo en esa solicitud | Puntual |
| `@archivo` en `CLAUDE.md` | Cada sesión automáticamente | Permanente |

---

### Módulo 4: Realizar cambios

#### Capturas de pantalla para comunicación visual

Cuando querés modificar algo visual en tu app, describir el problema con texto puede ser impreciso. Una captura de pantalla elimina la ambigüedad: Claude ve exactamente qué área de la UI querés cambiar.

Para pegar una imagen en Claude Code: **`Ctrl+V`** (funciona en macOS también — no `Cmd+V`).

Una vez pegada, podés pedir cambios concretos sobre lo que se ve en la imagen sin tener que describir dónde está el componente.

---

#### Modo de planificación (`/plan`)

Para tareas que afectan varios archivos o requieren entender una parte amplia del código antes de tocar nada, el **Modo de planificación** hace que Claude explore primero y actúe después.

**Cómo activarlo:**

| Método | Comando |
| --- | --- |
| Escribir en el chat | `/plan` |
| Atajo de teclado | `Shift + Tab` dos veces (o una vez si ya aceptás ediciones automáticamente) |

**Qué hace Claude en este modo:**

1. Lee más archivos del proyecto de lo habitual
2. Elabora un plan de implementación detallado
3. Te muestra exactamente lo que pretende hacer
4. Espera tu aprobación antes de ejecutar cualquier cambio

> **Truco:** al revisar el plan, presioná `Ctrl+G` para abrirlo en tu editor de texto. Podés modificar el plan directamente antes de aprobarlo — Claude verá la versión final que enviés.

---

#### Nivel de esfuerzo (`/effort`)

Por defecto, Claude razona antes de responder. Podés controlar cuánto tiempo dedica a ese razonamiento con `/effort`:

| Nivel | Velocidad | Costo | Ideal para |
| --- | --- | --- | --- |
| `low` | Rápido | Menor | Cambios simples, preguntas directas |
| *(default)* | Balanceado | Medio | La mayoría de las tareas |
| `max` | Más lento | Mayor | Problemas difíciles, debugging complejo |

Ejecutá `/effort` para ver tu nivel actual antes de ajustarlo.

**Para razonamiento profundo en una sola pregunta**, sin cambiar el nivel de la sesión, agregá la palabra clave `ultrathink` al mensaje:

```text
ultrathink — why is this recursive function causing a stack overflow?
```

Esto le indica a Claude que razone más en ese turno puntual, sin afectar el resto de la sesión.

Para ver el proceso de razonamiento de Claude en tiempo real: **`Ctrl+O`**.

---

#### Cuándo usar cada herramienta

Modo de planificación y nivel de esfuerzo resuelven tipos de complejidad diferentes:

| Herramienta | Tipo de complejidad | Ejemplo |
| --- | --- | --- |
| **Modo planificación** (`/plan`) | **Amplitud** — muchos archivos, varios pasos | Refactorizar un módulo entero, agregar una feature que cruza capas |
| **Esfuerzo alto** (`/effort max`) | **Profundidad** — lógica difícil, un solo problema | Debuggear un algoritmo, resolver un race condition |
| **`ultrathink`** | Profundidad puntual sin cambiar la sesión | Una pregunta difícil en medio de una tarea normal |
| **Ambos combinados** | Amplitud + profundidad | Arquitecturar e implementar algo complejo de cero |

> Ambas herramientas consumen tokens adicionales. Usalas cuando la complejidad lo justifica — no como modo por defecto.

---

### Módulo 5: Control de contexto

#### El problema que resuelve

En conversaciones largas, el contexto se acumula. Parte de ese contexto es valioso (la comprensión que Claude construyó sobre tu código); otra parte es ruido (un error que Claude intentó debuggear por tres mensajes y que ya no es relevante). Sin herramientas de control, Claude puede distraerse con contexto obsoleto o desenfocarse de la tarea actual.

Este módulo cubre las técnicas para mantener a Claude enfocado y productivo.

---

#### `Escape` — interrumpir y redirigir

Cuando Claude empieza a ir en la dirección equivocada, presioná `Escape` para detener la respuesta en curso. Luego podés redirigirlo hacia lo que realmente necesitás.

Ejemplo típico: le pedís a Claude que escriba tests para varias funciones y empieza a elaborar un plan exhaustivo para todas. Presionás `Escape` y le pedís que se enfoque solo en una función a la vez.

**Combinado con `/memory`:** si Claude comete el mismo error en múltiples conversaciones, el flujo es:

```text
1. Escape  →  detener la respuesta incorrecta
2. /memory →  abrir CLAUDE.md y agregar la corrección
3. Continuar la conversación con el comportamiento corregido
```

De esta forma el error queda documentado y Claude no lo repite en sesiones futuras.

---

#### `Escape` dos veces / `/rewind` — rebobinar la conversación

Presionar `Escape` dos veces (o escribir `/rewind`) muestra todos los mensajes que enviaste y te permite volver a un punto anterior de la conversación para continuar desde ahí.

Útil cuando una rama de la conversación (por ejemplo, un debugging largo) ya no es relevante para la tarea siguiente, pero el contexto anterior sí lo es.

```text
Sin /rewind:
  [contexto inicial] → [debugging largo] → [nueva tarea]
  Claude arrastra el contexto del debugging a la nueva tarea

Con /rewind:
  [contexto inicial] → [nueva tarea]
  Claude retoma desde el contexto limpio
```

---

#### `/compact` — resumir sin perder lo aprendido

`/compact` condensa todo el historial de la conversación en un resumen, preservando lo que Claude aprendió sobre tu proyecto pero reduciendo el volumen de tokens.

**Cuándo usarlo:**

- Claude acumuló conocimiento valioso sobre la tarea actual
- Querés continuar con tareas relacionadas
- La conversación se extendió mucho pero no querés perder el contexto

Lo que se preserva: comprensión del código, decisiones tomadas, patrones identificados.
Lo que se comprime: el historial verbatim de mensajes.

---

#### `/clear` — empezar de cero

`/clear` inicia una conversación nueva con contexto en blanco. El historial anterior no se elimina — podés retomarlo con `/resume`.

**Cuándo usarlo:**

- Cambiás a una tarea completamente diferente
- El contexto acumulado podría confundir a Claude respecto a la nueva tarea
- Querés un punto de partida limpio

---

#### Resumen: qué hacer según la situación

| Situación | Herramienta | Por qué |
| --- | --- | --- |
| Claude va en la dirección equivocada | `Escape` | Detener y redirigir sin perder la sesión |
| Claude repite el mismo error | `Escape` + `/memory` | Corregir y dejar constancia permanente |
| Una rama de la conversación ya no es útil | `/rewind` | Volver al punto anterior sin perder el contexto previo |
| Conversación larga con contexto valioso | `/compact` | Comprimir sin perder lo aprendido |
| Nueva tarea sin relación con la anterior | `/clear` | Contexto en blanco para no confundir |

---

#### Referencia rápida de atajos

| Acción | Comando / Atajo |
| --- | --- |
| Interrumpir respuesta | `Escape` |
| Agregar instrucción permanente | `/memory` |
| Rebobinar conversación | `Escape` × 2 o `/rewind` |
| Resumir historial | `/compact` |
| Nueva conversación en blanco | `/clear` |
| Retomar conversación anterior | `/resume` |

---

### Módulo 6: Comandos personalizados

#### ¿Qué son y para qué sirven?

Claude Code incluye comandos integrados (`/plan`, `/clear`, `/memory`, etc.), pero también podés crear los tuyos propios. Los comandos personalizados son prompts reutilizables almacenados como archivos Markdown que se invocan con `/nombre`.

Son la forma de convertir un flujo de trabajo repetitivo en un comando de una línea — con las convenciones de tu proyecto ya incorporadas.

---

#### Estructura de carpetas

```text
tu-proyecto/
└── .claude/
    └── commands/
        ├── audit.md          → /audit
        ├── write_tests.md    → /write_tests
        └── deploy.md         → /deploy
```

Pasos para crear un comando:

1. Ubicá la carpeta `.claude/` en la raíz del proyecto
2. Creá el subdirectorio `commands/` dentro de ella
3. Creá un archivo `.md` con el nombre del comando

El nombre del archivo es el nombre del comando. Claude Code lo detecta automáticamente — no hace falta reiniciar.

---

#### Ejemplo: comando `/audit`

Un comando de auditoría de dependencias que siempre sigue los mismos tres pasos:

```markdown
<!-- .claude/commands/audit.md -->
Run npm audit to find vulnerable installed packages.
Run npm audit fix to apply updates.
Run tests to verify the updates haven't broken anything.
```

Al ejecutar `/audit`, Claude corre los tres pasos en orden sin que tengas que repetirlos manualmente ni recordar la secuencia correcta.

---

#### Comandos con argumentos (`$ARGUMENTS`)

Los comandos pueden recibir texto libre al momento de invocarse usando el marcador `$ARGUMENTS`. Eso los hace reutilizables para distintas entradas:

```markdown
<!-- .claude/commands/write_tests.md -->
Write comprehensive tests for: $ARGUMENTS

Testing conventions:
* Use Vitest with React Testing Library
* Place test files in a __tests__ directory in the same folder as the source file
* Name test files as [filename].test.ts(x)
* Use @/ prefix for imports

Coverage:
* Test happy paths
* Test edge cases
* Test error states
```

Invocación:

```text
/write_tests the use-auth.ts file in the hooks directory
```

`$ARGUMENTS` se reemplaza por todo el texto que escribís después del nombre del comando. No tiene que ser una ruta de archivo — puede ser cualquier descripción o instrucción adicional.

---

#### Dónde viven los comandos

| Ubicación | Alcance | Compartido |
| --- | --- | --- |
| `.claude/commands/` (proyecto) | Solo ese proyecto | Sí — se commitea al repo |
| `~/.claude/commands/` (global) | Todos los proyectos | No — solo en tu máquina |

Los comandos globales son útiles para flujos de trabajo personales que aplican a cualquier proyecto (por ejemplo, `/explain` para pedir explicaciones con tu formato preferido).

---

#### Comparación: comando personalizado vs. prompt manual

| Aspecto | Prompt manual | Comando personalizado |
| --- | --- | --- |
| Velocidad | Escribir todo cada vez | Una línea: `/audit` |
| Consistencia | Puede variar entre sesiones | Siempre los mismos pasos |
| Convenciones del proyecto | Hay que recordarlas y escribirlas | Incorporadas en el archivo |
| Reutilización entre sesiones | No | Sí — persiste en el repo |

---

#### Casos de uso típicos para comandos personalizados

- **Auditorías de seguridad:** `npm audit` + fix + tests en un solo comando
- **Generación de tests:** con las convenciones del equipo ya especificadas
- **Code review interno:** checklist de revisión personalizada
- **Deploy checklist:** pasos previos al deploy que siempre hay que verificar
- **Boilerplate:** generar componentes o módulos siguiendo la estructura del proyecto

---

### Módulo 7: Servidores MCP con Claude Code

#### ¿Qué agrega MCP a Claude Code?

Por defecto, Claude Code puede leer archivos, editar código y ejecutar comandos en tu máquina. Los servidores MCP extienden esas capacidades agregando herramientas completamente nuevas — acceso a un navegador, una base de datos, APIs externas, servicios en la nube.

La diferencia conceptual: las herramientas integradas de Claude Code operan sobre tu código fuente; las herramientas MCP operan sobre sistemas externos.

---

#### Instalar un servidor MCP

El comando se ejecuta en la terminal del sistema, **no dentro de Claude Code**:

```bash
claude mcp add <nombre> <comando-de-inicio>
```

Ejemplo con Playwright (control de navegador):

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

| Parte del comando | Qué hace |
| --- | --- |
| `claude mcp add` | Registra el servidor en la configuración de Claude Code |
| `playwright` | Nombre del servidor (se usa para los permisos) |
| `npx @playwright/mcp@latest` | Comando que Claude Code ejecuta para iniciar el servidor |

---

#### Gestión de permisos

La primera vez que Claude usa una herramienta de un servidor MCP, pide permiso. Si querés aprobar el servidor de forma permanente, editá `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": ["mcp__playwright"],
    "deny": []
  }
}
```

> El prefijo es `mcp__` (doble guion bajo) seguido del nombre del servidor. Esto preaprueba **todas** las herramientas de ese servidor — usalo con servidores de confianza.

---

#### Ejemplo práctico: mejora de prompts con visión real

El caso de uso del curso ilustra bien el salto cualitativo que da MCP. Sin Playwright, para mejorar un prompt de generación de UI tendrías que:

1. Generar un componente
2. Mirarlo vos en el navegador
3. Describir a Claude qué querés cambiar
4. Repetir

Con Playwright, Claude hace ese loop solo:

```text
"Navegá a localhost:3000, generá un componente básico, revisá
el estilo visual y actualizá el prompt en
@src/lib/prompts/generation.tsx para producir mejores
componentes en el futuro."
```

```text
Claude:
  1. Abre navegador → navega a localhost:3000
  2. Genera un componente de prueba
  3. Analiza el resultado visual (no solo el código)
  4. Actualiza el archivo de prompts con mejoras concretas
  5. Prueba el prompt actualizado con un nuevo componente
```

La diferencia clave: Claude ve el **resultado visual real**, no solo el código. Eso le permite tomar decisiones sobre estilo y diseño que no puede tomar leyendo solo texto.

---

#### El ecosistema MCP

Playwright es el ejemplo del curso, pero el ecosistema es amplio:

| Categoría | Qué habilita |
| --- | --- |
| **Navegadores** (Playwright, Puppeteer) | Automatización web, testing visual, scraping |
| **Bases de datos** (PostgreSQL, SQLite, MongoDB) | Consultas directas, migraciones, análisis de datos |
| **APIs y servicios** (GitHub, Slack, Jira) | Crear issues, revisar PRs, enviar notificaciones |
| **Servicios en la nube** (AWS, GCP) | Deploy, gestión de infraestructura |
| **Herramientas de dev** (Docker, CI/CD) | Construir imágenes, correr pipelines |
| **Sistema de archivos extendido** | Operaciones fuera del directorio del proyecto |

---

#### Cuándo agregar un servidor MCP

| Señal | Servidor a considerar |
| --- | --- |
| Necesitás que Claude vea tu app corriendo | Playwright / Puppeteer |
| Claude tiene que consultar o modificar la base de datos | MCP de la DB que usás |
| Querés que Claude abra issues o PRs automáticamente | GitHub MCP |
| Claude necesita contexto de tickets o documentación | Jira / Notion / Confluence MCP |

La regla general: si le estás copiando y pegando output de un sistema externo a Claude manualmente, ese es un candidato para un servidor MCP.

---

### Módulo 8: Integración con GitHub

#### ¿Qué habilita la integración?

Claude Code puede integrarse directamente con GitHub Actions para operar como un miembro automatizado del equipo: responder menciones en issues y PRs, y revisar código automáticamente cada vez que se abre una pull request.

---

#### Configuración inicial

Todo empieza con un único comando dentro de Claude Code:

```text
/install-github-app
```

Claude guía el proceso completo:

1. Instala la app de Claude Code en el repositorio de GitHub
2. Agrega la clave API como secret de GitHub Actions
3. Genera automáticamente una PR con los archivos de workflow

Una vez fusionada esa PR, el directorio `.github/workflows/` tendrá dos archivos nuevos — uno por cada flujo de trabajo.

---

#### Los dos workflows predeterminados

##### Workflow de menciones (`@claude`)

Podés mencionar a `@claude` en cualquier issue o PR. Claude:

1. Analiza la solicitud y crea un plan de tareas
2. Ejecuta la tarea con acceso completo al código
3. Responde directamente en el thread del issue o PR

```text
Ejemplo en un issue:
  "Hey @claude, podés agregar tests para el componente LoginForm?"

Claude:
  → lee el código relevante
  → escribe los tests
  → responde con un resumen y el diff
```

##### Workflow de revisión automática de PRs

Cada vez que se abre una PR, Claude automáticamente:

- Revisa los cambios propuestos
- Analiza el impacto en el resto del código
- Publica un informe detallado en la PR

Sin intervención manual — Claude revisa cada PR como lo haría un reviewer del equipo.

---

#### Personalización de los workflows

Después de fusionar la PR inicial, podés editar los archivos de workflow para adaptarlos a tu proyecto. Las personalizaciones principales son tres:

**1. Setup del entorno** — pasos que corren antes de que Claude empiece:

```yaml
- name: Project Setup
  run: |
    npm run setup
    npm run dev:daemon
```

**2. Instrucciones personalizadas** — contexto específico del proyecto que Claude recibe:

```yaml
custom_instructions: |
  The project is already set up with all dependencies installed.
  The server is already running at localhost:3000. Logs from it
  are being written to logs.txt. If needed, you can query the
  db with the 'sqlite3' cli. If needed, use the mcp__playwright
  set of tools to launch a browser and interact with the app.
```

**3. Servidores MCP** — herramientas adicionales disponibles para Claude en el workflow:

```yaml
mcp_config: |
  {
    "mcpServers": {
      "playwright": {
        "command": "npx",
        "args": [
          "@playwright/mcp@latest",
          "--allowed-origins",
          "localhost:3000;cdn.tailwindcss.com;esm.sh"
        ]
      }
    }
  }
```

---

#### Permisos en GitHub Actions

A diferencia del desarrollo local — donde podés preaprobar un servidor completo con `mcp__playwright` — en GitHub Actions **cada herramienta debe listarse individualmente**:

```yaml
allowed_tools: "Bash(npm:*),Bash(sqlite3:*),mcp__playwright__browser_snapshot,mcp__playwright__browser_click,..."
```

| Contexto | Cómo se aprueban permisos |
| --- | --- |
| Local (`.claude/settings.local.json`) | Por servidor: `mcp__playwright` aprueba todas sus tools |
| GitHub Actions (`allowed_tools`) | Por herramienta individual: cada tool se lista explícitamente |

Esto es más restrictivo intencionalmente — en un entorno de CI automatizado, es importante controlar exactamente qué puede hacer Claude.

---

#### Resumen — Unidad 2

Esta unidad cubrió todo el flujo práctico de trabajo con Claude Code: desde la instalación hasta la integración con el repositorio remoto.

**Instalación y proyecto** (Módulos 1–2): Claude Code se instala con un script de una línea y se autentica con claude.ai. El proyecto del curso (`uigen`) es opcional — podés usar cualquier código propio.

**Contexto** (Módulo 3): el mecanismo central es `CLAUDE.md`, que actúa como system prompt permanente del proyecto. Existen tres ubicaciones (proyecto compartido, local, global) y dos formas de incluir archivos: `@archivo` en el chat (puntual) o en `CLAUDE.md` (permanente).

**Realizar cambios** (Módulo 4): capturas de pantalla para comunicación visual (`Ctrl+V`), modo planificación (`/plan`) para tareas amplias, y nivel de esfuerzo (`/effort`) para tareas profundas. `ultrathink` para profundidad puntual sin cambiar la sesión.

**Control de contexto** (Módulo 5): `Escape` para interrumpir, `/rewind` para rebobinar, `/compact` para comprimir sin perder contexto, `/clear` para empezar de cero.

**Comandos personalizados** (Módulo 6): archivos `.md` en `.claude/commands/` que se invocan con `/nombre`. Soportan `$ARGUMENTS` para recibir parámetros. Persisten en el repo y son compartibles con el equipo.

**MCP** (Módulo 7): servidores que extienden las capacidades de Claude más allá del código — navegadores, bases de datos, APIs. Se instalan con `claude mcp add` y se preaprueba con `mcp__nombre` en `settings.local.json`.

**GitHub** (Módulo 8): `/install-github-app` configura dos workflows automáticos — menciones con `@claude` y revisión de PRs. Los workflows son personalizables con setup de entorno, instrucciones y servidores MCP. En CI, los permisos son por herramienta individual.

---

### Mapa conceptual — Unidad 2

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 45, 'rankSpacing': 85}}}%%
flowchart LR
    U2((Unidad 2\nManos a la obra))

    U2 --> INST[Instalación\ny proyecto]
    INST --> INST1[claude mcp add\nscript de instalación]
    INST --> INST2[uigen o\nproyecto propio]

    U2 --> CTX[Contexto]
    CTX --> CTX1[/init → CLAUDE.md]
    CTX --> CTX2[3 ubicaciones\nproyecto · local · global]
    CTX --> CTX3[@archivo\npuntual o permanente]

    U2 --> CAMBIOS[Realizar\ncambios]
    CAMBIOS --> CAMBIOS1[Captura de pantalla\nCtrl+V]
    CAMBIOS --> CAMBIOS2[/plan\namplitud]
    CAMBIOS --> CAMBIOS3[/effort\nprofundidad]
    CAMBIOS --> CAMBIOS4[ultrathink\npuntual]

    U2 --> CTRL[Control\nde contexto]
    CTRL --> CTRL1[Escape\ninterrumpir]
    CTRL --> CTRL2[/rewind\nrebobinar]
    CTRL --> CTRL3[/compact\ncomprimir]
    CTRL --> CTRL4[/clear\nnueva sesión]

    U2 --> CMD[Comandos\npersonalizados]
    CMD --> CMD1[.claude/commands/\nnombre.md]
    CMD --> CMD2[$ARGUMENTS\nparámetros]
    CMD --> CMD3[global vs\nproyecto]

    U2 --> MCP[Servidores\nMCP]
    MCP --> MCP1[claude mcp add]
    MCP --> MCP2[mcp__nombre\npermisos local]
    MCP --> MCP3[Playwright · DB\nAPIs · Cloud]

    U2 --> GH[GitHub\nIntegración]
    GH --> GH1[/install-github-app]
    GH --> GH2[@claude en\nissues y PRs]
    GH --> GH3[Revisión\nautomática de PRs]
    GH --> GH4[Permisos por\nherramienta en CI]
```

---

## Unidad 3: Hooks y SDK

---

### Módulo 1: Presentando los hooks

#### ¿Qué son los hooks?

Los hooks son comandos que se ejecutan automáticamente **antes o después** de que Claude ejecute una herramienta. Se insertan en el flujo normal de Claude Code sin que Claude lo decida — son disparados por el sistema, no por el modelo.

```text
Flujo sin hooks:
  Tu pregunta → Claude → decide usar una tool → tool se ejecuta → resultado

Flujo con hooks:
  Tu pregunta → Claude → decide usar una tool
    → [PreToolUse hook corre]
    → tool se ejecuta
    → [PostToolUse hook corre]
    → resultado
```

---

#### Los dos tipos principales

| Tipo | Cuándo corre | Puede bloquear la tool | Puede dar feedback a Claude |
| --- | --- | --- | --- |
| `PreToolUse` | Antes de ejecutar la tool | Sí | Sí |
| `PostToolUse` | Después de ejecutar la tool | No (ya ocurrió) | Sí |

`PreToolUse` es el guardián: puede detener la operación y enviar un mensaje de error a Claude antes de que haga algo. `PostToolUse` es el seguimiento: no puede deshacer lo que pasó, pero puede ejecutar acciones adicionales y retroalimentar a Claude.

---

#### Dónde se configuran

Los hooks viven en los archivos de configuración de Claude Code. Tres ubicaciones posibles:

| Archivo | Alcance | Compartido |
| --- | --- | --- |
| `~/.claude/settings.json` | Global — todos los proyectos | No |
| `.claude/settings.json` | Proyecto | Sí (se commitea) |
| `.claude/settings.local.json` | Proyecto personal | No |

Podés escribirlos manualmente en esos archivos o usar el comando `/hooks` dentro de Claude Code para una interfaz guiada.

---

#### Estructura de configuración

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "node /home/hooks/read_hook.js"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node /home/hooks/edit_hook.js"
          }
        ]
      }
    ]
  }
}
```

| Campo | Descripción |
| --- | --- |
| `matcher` | Nombre de la tool (o patrón con `\|`) que dispara el hook |
| `type` | Siempre `"command"` por ahora |
| `command` | El comando del sistema que se ejecuta |

El `matcher` soporta el operador `|` para apuntar a múltiples tools: `"Write|Edit|MultiEdit"`.

---

#### Qué recibe el hook y qué puede devolver

Cuando Claude Code dispara un hook, el proceso recibe por `stdin` un objeto JSON con los detalles de la llamada a la tool — qué tool es, con qué argumentos, etc. El hook puede:

**En `PreToolUse`:**

- Salir con código `0` → la tool procede normalmente
- Salir con código distinto de `0` + mensaje en stdout → Claude recibe el mensaje como error y la tool **no se ejecuta**

**En `PostToolUse`:**

- Escribir en stdout → Claude recibe ese texto como feedback adicional sobre lo que acaba de hacer

---

#### Casos de uso típicos para hooks

| Caso | Tipo de hook | Qué hace |
| --- | --- | --- |
| Formatear código después de editar | `PostToolUse` (Write/Edit) | Corre Prettier, ESLint fix, etc. |
| Ejecutar tests al modificar archivos | `PostToolUse` (Write/Edit) | Corre el suite de tests afectado |
| Bloquear acceso a archivos sensibles | `PreToolUse` (Read) | Verifica la ruta y rechaza si está en lista negra |
| Verificar tipo antes de guardar | `PostToolUse` (Write) | Corre `tsc --noEmit` y reporta errores a Claude |
| Registrar actividad de Claude | `PostToolUse` (cualquiera) | Escribe en un log qué archivos tocó Claude |
| Validar convenciones de nombres | `PreToolUse` (Write) | Rechaza si el nombre del archivo no sigue la convención |

La diferencia con los comandos personalizados: los comandos los ejecutás vos explícitamente; los hooks corren solos cada vez que Claude usa la tool correspondiente.

---

### Módulo 2: Implementar un hook

#### El objetivo

Crear un hook `PreToolUse` que bloquee la herramienta `Read` cuando intenta abrir el archivo `.env`. Es el ejemplo más concreto del patrón de control de acceso: interceptar antes de que ocurra la lectura.

---

#### Paso 1: Registrar el hook en la configuración

En `.claude/settings.local.json` (no se commitea — es configuración personal):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          { "type": "command", "command": "node $PWD/hooks/read_hook.js" }
        ]
      }
    ]
  }
}
```

`$PWD` resuelve al directorio actual — así el path al script es relativo al directorio donde corrés Claude Code, no absoluto.

---

#### Paso 2: Escribir el script del hook

Crear el archivo `hooks/read_hook.js`:

```javascript
process.stdin.setEncoding("utf8");
let input = "";
process.stdin.on("data", (d) => (input += d));
process.stdin.on("end", () => {
  const toolArgs = JSON.parse(input);
  const readPath = toolArgs.tool_input?.file_path || "";

  if (readPath.includes(".env")) {
    console.error("You cannot read the .env file");
    process.exit(2);    // código distinto de 0 → bloquea la tool
  }

  process.exit(0);      // código 0 → la tool procede normalmente
});
```

El flujo del script:

```text
stdin recibe JSON con la llamada a la tool
  ↓
JSON.parse → toolArgs
  ↓
toolArgs.tool_input.file_path  →  la ruta que Claude quiere leer
  ↓
¿contiene ".env"?
  ├─ Sí → console.error(mensaje) + process.exit(2)  →  Claude ve el mensaje, Read no corre
  └─ No → process.exit(0)                           →  Read corre normalmente
```

---

#### La estructura del JSON de entrada

Cada tool envía una estructura diferente. Para `Read`:

```json
{
  "tool_name": "Read",
  "tool_input": {
    "file_path": "/home/user/project/.env"
  }
}
```

Por eso el hook accede a `toolArgs.tool_input?.file_path` — ese es el campo específico que usa la herramienta `Read`.

---

#### Por qué este hook solo cubre `Read`

Cada tool envía campos distintos en `tool_input`. Un hook que chequea `file_path` solo funciona para `Read`:

| Tool | Campo relevante | Ejemplo |
| --- | --- | --- |
| `Read` | `tool_input.file_path` | `"/project/.env"` |
| `Grep` | `tool_input.path` | `"/project/"` (directorio, no archivo) |
| `Bash` | `tool_input.command` | `"cat .env"` o `"grep API_KEY .env"` |

Un `grep` de todo el proyecto para `API_KEY` o un `cat .env` en Bash pasan sin ser detectados por este hook, porque ninguno usa `file_path`.

---

#### Protección completa: hook + `permissions.deny`

Para bloquear el acceso a `.env` desde cualquier herramienta, combiná el hook con una regla de permisos en `settings.local.json`:

```json
{
  "permissions": {
    "deny": ["Read(**/.env)"]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          { "type": "command", "command": "node $PWD/hooks/read_hook.js" }
        ]
      }
    ]
  }
}
```

| Mecanismo | Qué bloquea | Cómo |
| --- | --- | --- |
| Hook `PreToolUse` en `Read` | Solo la tool `Read` | Lógica en el script, mensaje personalizado a Claude |
| `permissions.deny` | Todas las tools uniformemente | Regla declarativa, Claude no puede ignorarla |
| Ambos combinados | Cobertura completa | Hook para mensaje claro + deny como barrera de seguridad |

> La regla `permissions.deny` es más robusta que el hook porque se aplica independientemente de la tool que intente acceder al archivo. El hook es útil cuando además querés dar un mensaje específico a Claude explicando por qué no puede acceder.

---

### Módulo 3: Definiendo hooks

#### Los cuatro pasos para construir un hook

Todo hook se define siguiendo la misma secuencia:

```text
1. ¿PreToolUse o PostToolUse?
   └─ ¿Necesitás bloquear algo antes de que ocurra?  → PreToolUse
   └─ ¿Querés reaccionar después de que ya ocurrió?  → PostToolUse

2. ¿Qué tools querés monitorizar?
   └─ Especificás el matcher: "Read", "Write|Edit", "Bash", etc.

3. Escribir el comando que recibirá la llamada
   └─ Lee JSON desde stdin, parsea tool_name y tool_input

4. Comunicar la decisión a Claude Code
   └─ exit(0) → permitir
   └─ exit(2) → bloquear (solo PreToolUse), mensaje en stderr
```

---

#### Herramientas integradas disponibles para monitorizar

Claude Code tiene un conjunto de tools integradas que podés usar como matcher en los hooks:

| Tool | Qué hace |
| --- | --- |
| `Read` | Lee el contenido de un archivo |
| `Write` | Crea o sobreescribe un archivo |
| `Edit` | Edita partes de un archivo existente |
| `MultiEdit` | Edita múltiples partes de un archivo en una sola operación |
| `Bash` | Ejecuta comandos de shell |
| `Grep` | Busca patrones en archivos o directorios |
| `Glob` | Lista archivos por patrón |
| `TodoWrite` | Escribe o actualiza la lista de tareas interna |
| `WebFetch` | Descarga contenido de una URL |
| `WebSearch` | Realiza búsquedas web |

> Si agregás servidores MCP, sus herramientas también estarán disponibles para monitorizar. Podés pedirle a Claude directamente que liste todas las tools disponibles en la sesión actual.

---

#### La estructura completa del JSON de entrada

Cuando Claude Code dispara un hook, el proceso recibe este JSON por stdin:

```json
{
  "session_id": "2d6a1e4d-6...",
  "transcript_path": "/Users/sg/...",
  "hook_event_name": "PreToolUse",
  "tool_name": "Read",
  "tool_input": {
    "file_path": "/code/queries/.env"
  }
}
```

| Campo | Descripción |
| --- | --- |
| `session_id` | Identificador único de la sesión de Claude Code |
| `transcript_path` | Ruta al archivo de transcripción de la sesión |
| `hook_event_name` | `"PreToolUse"` o `"PostToolUse"` |
| `tool_name` | Nombre de la tool que Claude quiere usar |
| `tool_input` | Objeto con los argumentos específicos de esa tool |

`tool_input` varía según la tool — por eso cada hook que chequea un campo específico solo funciona para la tool que envía ese campo.

---

#### Códigos de salida y flujo de control

El script comunica su decisión a Claude Code únicamente a través del código de salida:

| Código de salida | Efecto | Cuándo aplica |
| --- | --- | --- |
| `0` | La tool procede normalmente | PreToolUse y PostToolUse |
| `2` | La tool es bloqueada | Solo PreToolUse |
| Cualquier otro | Comportamiento indefinido — evitar | — |

Cuando el script sale con código `2`:

- Lo que escribió en `stderr` → Claude lo recibe como mensaje explicativo
- Lo que escribió en `stdout` → Claude también lo puede recibir como feedback
- La tool **no se ejecuta**

```javascript
// Patrón estándar de un hook PreToolUse
process.stdin.setEncoding("utf8");
let input = "";
process.stdin.on("data", (d) => (input += d));
process.stdin.on("end", () => {
  const { tool_name, tool_input } = JSON.parse(input);

  // lógica de decisión
  if (/* condición de bloqueo */) {
    console.error("Mensaje que verá Claude");
    process.exit(2);
  }

  process.exit(0);
});
```

---

#### Monitorizar múltiples tools para el mismo objetivo

Si un comportamiento que querés controlar puede ocurrir a través de varias tools, necesitás un matcher y lógica separada para cada una:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [{ "type": "command", "command": "node $PWD/hooks/read_hook.js" }]
      },
      {
        "matcher": "Grep",
        "hooks": [{ "type": "command", "command": "node $PWD/hooks/grep_hook.js" }]
      },
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "node $PWD/hooks/bash_hook.js" }]
      }
    ]
  }
}
```

Cada script inspecciona el campo relevante de `tool_input` para su propia tool. No podés usar un solo script para todas porque los campos son diferentes en cada una.

> Para bloqueo uniforme sin escribir un hook por tool, usá `permissions.deny` — se aplica a todas las tools sin necesidad de lógica personalizada.

---

### Módulo 4: Trampas alrededor de los hooks

#### El problema: rutas absolutas vs. compartir configuración

La documentación de Claude Code recomienda usar **rutas absolutas** en los comandos de los hooks, no rutas relativas. La razón es de seguridad: las rutas relativas son vulnerables a ataques de interceptación de rutas (*path hijacking*) y de inserción de binarios maliciosos.

```json
// ❌ Ruta relativa — vulnerable
{ "command": "node hooks/read_hook.js" }

// ✓ Ruta absoluta — recomendada
{ "command": "node /Users/jerom/projects/uigen/hooks/read_hook.js" }
```

El problema: la ruta absoluta en tu máquina es distinta a la de cualquier otro miembro del equipo. Si commiteas `settings.json` con rutas absolutas, el archivo no funciona en otra máquina sin editarlo manualmente.

---

#### La solución del proyecto: `settings.example.json` + script de init

El proyecto `uigen` resuelve la tensión entre seguridad y portabilidad con este flujo:

```text
settings.example.json          ← se commitea al repo
  │  (contiene $PWD como placeholder)
  │
  └─ npm run setup
       │
       └─ scripts/init-claude.js
            │  reemplaza cada $PWD por la ruta absoluta real de la máquina
            │
            └─ .claude/settings.local.json   ← no se commitea
                 (contiene rutas absolutas correctas para esta máquina)
```

El `settings.example.json` usa `$PWD` como marcador de posición:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          { "type": "command", "command": "node $PWD/hooks/read_hook.js" }
        ]
      }
    ]
  }
}
```

Cuando corrés `npm run setup`, el script `init-claude.js` reemplaza `$PWD` por la ruta absoluta del proyecto en tu máquina y guarda el resultado como `settings.local.json`.

---

#### Por qué hay dos archivos de settings en `.claude/`

Después de correr `npm run setup`, el directorio `.claude/` tiene:

| Archivo | Origen | Commiteado | Contiene |
| --- | --- | --- | --- |
| `settings.json` | Creado manualmente o por el equipo | Sí | Configuración compartida (sin hooks con rutas) |
| `settings.local.json` | Generado por `npm run setup` | No (en `.gitignore`) | Hooks con rutas absolutas para esta máquina |

Claude Code carga ambos archivos. `settings.local.json` sobrescribe o complementa `settings.json` para las configuraciones que solapan.

---

#### Resumen del patrón

| Problema | Solución |
| --- | --- |
| Rutas absolutas = no portables | Usar `$PWD` como placeholder en el archivo de ejemplo |
| Rutas relativas = vulnerables | El script de init las reemplaza por absolutas en cada máquina |
| `settings.local.json` no se puede compartir | `settings.example.json` se commitea y sirve de plantilla |

> Este patrón — archivo de ejemplo con placeholders + script de setup que genera el archivo local — es una buena práctica para cualquier configuración que contenga rutas absolutas o valores específicos de la máquina.

---

### Módulo 5: Hooks muy útiles

#### Por qué los hooks son especialmente valiosos en proyectos grandes

Claude trabaja en turnos: ve el archivo que modificó, pero no siempre vuelve a escanear todo el proyecto para detectar efectos secundarios. En proyectos grandes eso genera dos problemas recurrentes:

1. **Cambios en firmas de funciones** → los call sites en otros archivos quedan desactualizados
2. **Código duplicado** → Claude escribe una nueva consulta en lugar de reutilizar una existente

Los hooks que se describen en este módulo atacan exactamente esos dos problemas.

---

#### Hook 1: verificación de tipos TypeScript (`PostToolUse`)

**El problema:** Claude modifica la firma de `schema.ts` pero no actualiza el call site en `main.ts`. Los errores de tipo existen, pero Claude no los ve a menos que vuelva a leer todos los archivos afectados.

**La solución:** un `PostToolUse` que corre `tsc --noEmit` después de cada edición y le reporta los errores directamente a Claude:

```text
Claude edita un archivo
  ↓
PostToolUse hook dispara
  ↓
tsc --noEmit corre en todo el proyecto
  ↓
¿Errores de tipo?
  ├─ No → hook termina silenciosamente, Claude continúa
  └─ Sí → hook escribe los errores en stdout
           Claude los recibe como feedback
           Claude corrige los archivos afectados
```

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          { "type": "command", "command": "node $PWD/hooks/typecheck_hook.js" }
        ]
      }
    ]
  }
}
```

El script corre `tsc --noEmit`, captura la salida y la escribe en stdout si hay errores. Claude la recibe como contexto adicional y procede a corregir los archivos que fallaron.

> Este patrón aplica a cualquier lenguaje tipado: TypeScript, Go, Rust, Python con mypy, etc. Para lenguajes no tipados, el equivalente es correr el suite de tests automáticamente.

---

#### Hook 2: detección de consultas duplicadas (`PostToolUse`)

**El problema:** en proyectos con muchas consultas a la base de datos, Claude a veces escribe una nueva consulta en lugar de reutilizar `getPendingOrders()` que ya existe. Ocurre especialmente en tareas de varios pasos donde la query es solo un componente de algo más grande.

**La solución:** un hook que inicia una **segunda instancia de Claude** para revisar el cambio:

```text
Claude modifica un archivo en ./queries/
  ↓
PostToolUse hook dispara
  ↓
Hook lanza una segunda instancia de Claude Code (vía SDK de agente)
  ↓
Segunda instancia recibe:
  - el diff del cambio
  - la lista de todas las funciones existentes en ./queries/
  ↓
¿Hay duplicación?
  ├─ No → segunda instancia termina silenciosamente
  └─ Sí → segunda instancia escribe feedback describiendo la función existente
           Claude original recibe ese feedback
           Claude original refactoriza para usar la función existente
```

Este es el patrón de **revisión entre instancias**: una instancia de Claude revisa el trabajo de otra. Lo habilita el **SDK de agente de Claude**, que permite lanzar instancias de Claude programáticamente desde un script.

---

#### Comparación de los dos hooks

| Hook | Tipo | Cuándo dispara | Costo | Velocidad |
| --- | --- | --- | --- | --- |
| TypeScript typecheck | `PostToolUse` | Cualquier Write/Edit | Bajo (solo compilador) | Rápido (~1–2 seg) |
| Detección de duplicados | `PostToolUse` | Edit en `./queries/` | Alto (nueva llamada a la API) | Más lento (varios segundos) |

El hook de TypeScript es liviano y se puede activar en cada edición sin pensar. El de duplicados tiene un costo real de API — por eso la recomendación es apuntarlo solo a directorios críticos donde la consistencia importa más.

---

#### Principios generalizables

Estos dos ejemplos ilustran patrones que podés aplicar a cualquier proyecto:

| Principio | Ejemplo concreto |
| --- | --- |
| Usar la salida del compilador/linter como feedback | `tsc --noEmit`, `eslint`, `mypy`, `cargo check` |
| Revisión entre instancias de IA | Hook que lanza Claude para revisar el trabajo de Claude |
| Monitorizar solo directorios de alto valor | `./queries/`, `./schema/`, `./api/` en lugar de todo el proyecto |
| Equilibrar beneficio vs. costo de API | Hooks livianos en cualquier edit; hooks pesados solo donde más importa |

La regla práctica: identificá los errores que Claude comete de forma recurrente en tu proyecto específico, y creá un hook que los detecte automáticamente.

---

### Módulo 6: Más tipos de hooks

#### Más allá de PreToolUse y PostToolUse

El curso se enfocó en los dos tipos más comunes, pero el sistema de hooks tiene siete tipos en total, cada uno disparado por un momento distinto del ciclo de vida de Claude Code:

| Tipo de hook | Cuándo se ejecuta |
| --- | --- |
| `PreToolUse` | Antes de que Claude ejecute una tool |
| `PostToolUse` | Después de que Claude ejecuta una tool |
| `Notification` | Cuando Claude necesita permiso para una tool, o tras 60 segundos de inactividad |
| `Stop` | Cuando Claude termina de responder |
| `SubagentStop` | Cuando un subagente (aparece como "Tarea" en la UI) finaliza |
| `PreCompact` | Antes de una operación de compactación (manual o automática) |
| `UserPromptSubmit` | Cuando el usuario envía una solicitud, antes de que Claude la procese |
| `SessionStart` | Al iniciar o reanudar una sesión |
| `SessionEnd` | Cuando finaliza la sesión |

Casos de uso para los menos obvios:

- **`Stop`:** notificar al usuario (Slack, sonido, notificación del sistema) que Claude terminó una tarea larga
- **`UserPromptSubmit`:** validar o enriquecer el prompt antes de enviarlo a Claude
- **`SessionStart`:** cargar contexto inicial, verificar el estado del entorno
- **`PreCompact`:** guardar un snapshot del contexto antes de que se comprima

---

#### El desafío: el JSON de entrada varía según el tipo

La estructura del JSON que recibe el script cambia completamente según el tipo de hook — y también según la tool, en el caso de `PreToolUse`/`PostToolUse`.

**Ejemplo: `PostToolUse` con `TodoWrite`**

```json
{
  "session_id": "9ecf22fa-...",
  "transcript_path": "<path>",
  "hook_event_name": "PostToolUse",
  "tool_name": "TodoWrite",
  "tool_input": {
    "todos": [{ "content": "write a readme", "status": "pending", "id": "1" }]
  },
  "tool_response": {
    "oldTodos": [],
    "newTodos": [{ "content": "write a readme", "status": "pending", "id": "1" }]
  }
}
```

**Ejemplo: `Stop`**

```json
{
  "session_id": "af9f50b6-...",
  "transcript_path": "<path>",
  "hook_event_name": "Stop",
  "stop_hook_active": false
}
```

Los campos comunes a todos los hooks son `session_id`, `transcript_path` y `hook_event_name`. Todo lo demás depende del tipo y, en el caso de tools, de cuál tool específica fue invocada.

---

#### Campos comunes vs. campos específicos

| Campo | Presente en | Descripción |
| --- | --- | --- |
| `session_id` | Todos | Identificador de la sesión |
| `transcript_path` | Todos | Ruta al archivo de transcripción |
| `hook_event_name` | Todos | Tipo de hook que disparó |
| `tool_name` | Pre/PostToolUse | Nombre de la tool invocada |
| `tool_input` | Pre/PostToolUse | Argumentos de la tool (estructura varía por tool) |
| `tool_response` | PostToolUse | Resultado de la ejecución de la tool |
| `stop_hook_active` | Stop | Si ya hay otro Stop hook activo |

---

#### Técnica: hook de logging para explorar la estructura

Antes de escribir la lógica de un hook nuevo, usá este hook auxiliar para inspeccionar exactamente qué JSON recibe tu script:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "jq . > post-log.json"
          }
        ]
      }
    ]
  }
}
```

`jq .` formatea el JSON de stdin con indentación y lo guarda en `post-log.json`. Después de que Claude use cualquier tool, abrís ese archivo y ves exactamente la estructura que tu hook recibiría.

El flujo de desarrollo recomendado para un hook nuevo:

```text
1. Agregá el hook de logging con matcher "*"
2. Pedile a Claude que use la tool que querés monitorizar
3. Abrí post-log.json → inspeccioná la estructura
4. Escribí tu hook real sabiendo exactamente qué campos esperar
5. Remové el hook de logging
```

> El matcher `"*"` captura todas las tools — útil para explorar, pero no lo dejes activo en producción porque escribe un archivo en cada llamada a cualquier tool.

---

### Módulo 7: El SDK de Claude Code

#### ¿Qué es el SDK de agente?

El SDK de agente te permite ejecutar Claude Code **programáticamente** desde tus propias aplicaciones y scripts. Obtenés el mismo ciclo de agente de la CLI (leer archivos, editar código, usar tools) pero bajo tu control: podés invocar a Claude desde otro programa, desde un hook, o como parte de un pipeline automatizado.

Disponible para **TypeScript** y **Python**.

---

#### Instalación del SDK

```bash
mkdir sdk-demo
cd sdk-demo
npm init -y
npm install @anthropic-ai/claude-agent-sdk
```

> El paquete correcto es `@anthropic-ai/claude-agent-sdk`. El paquete `@anthropic-ai/claude-code` es la CLI en sí y **no se puede importar** como librería.

---

#### Ejemplo mínimo

```javascript
// index.mjs
import { query } from "@anthropic-ai/claude-agent-sdk";

const prompt = "List the files in the current directory";

for await (const message of query({ prompt })) {
  console.log(JSON.stringify(message, null, 2));
}
```

```bash
node index.mjs
```

`query()` devuelve un **async iterable**: cada iteración produce un mensaje del ciclo de conversación — llamadas a tools, resultados de tools, texto de Claude — en el mismo formato que verías en la CLI.

---

#### Qué recibís en el stream

```text
Claude recibe el prompt
  ↓
Mensaje: { type: "assistant", content: "I'll list the files..." }
  ↓
Mensaje: { type: "tool_use", name: "Bash", input: { command: "ls" } }
  ↓
Mensaje: { type: "tool_result", content: "index.mjs\npackage.json\n..." }
  ↓
Mensaje: { type: "assistant", content: "The directory contains..." }
  ↓
Mensaje: { type: "result", ... }   ← fin del ciclo
```

Cada mensaje es un objeto JSON con `type` como campo discriminador.

---

#### Restricción de herramientas

Por defecto el SDK tiene acceso al conjunto completo de tools. Para limitarlo:

```javascript
for await (const message of query({
  prompt,
  options: { allowedTools: ["Read", "Glob"] },
})) {
  // solo puede leer archivos y listar por patrón
}
```

Es el equivalente del flag `--allowedTools` de la CLI. Útil cuando querés que un script automatizado tenga acceso mínimo — principio de mínimo privilegio aplicado al SDK.

---

#### Comparación: CLI vs. SDK

| Aspecto | CLI (`claude`) | SDK (`query()`) |
| --- | --- | --- |
| Uso | Interactivo, terminal | Programático, desde código |
| Control del flujo | El usuario | Tu aplicación |
| Integración | Manual | Automatizada (hooks, pipelines, apps) |
| Acceso a mensajes | Solo la respuesta final visible | Todos los eventos del ciclo |
| Restricción de tools | `--allowedTools` en el CLI | `options.allowedTools` en el código |

---

#### Casos de uso del SDK

- **Hooks de revisión entre instancias** (como el del Módulo 5): lanzar una segunda instancia de Claude para revisar el trabajo de la primera
- **Pipelines de CI/CD**: análisis de código, generación de changelogs, revisión automática de PRs
- **Scripts de automatización**: tareas repetitivas que involucran lectura y edición de archivos con inteligencia
- **Apps que embeben Claude Code**: incorporar las capacidades del agente en una herramienta propia

---

#### Capacidades del SDK

El SDK soporta todas las features de la CLI:

| Feature | Disponible en SDK |
| --- | --- |
| System prompt personalizado | ✓ |
| Servidores MCP | ✓ |
| Hooks | ✓ |
| Subagentes | ✓ |
| Reanudación de sesión | ✓ |
| Restricción de tools | ✓ (`allowedTools`) |

---

### Resumen — Unidad 3

Esta unidad introdujo dos mecanismos de extensión avanzados: los hooks y el SDK de agente.

**Hooks** (Módulos 1–6): comandos que se insertan automáticamente antes o después de que Claude use una tool. `PreToolUse` puede bloquear operaciones; `PostToolUse` ejecuta seguimiento y da feedback. La configuración vive en `settings.json` (compartido) o `settings.local.json` (personal). Los scripts reciben JSON por stdin y comunican su decisión con el código de salida (`0` = permitir, `2` = bloquear). Para protección completa, combinar hooks con `permissions.deny`. Para compartir hooks con rutas absolutas, usar el patrón `settings.example.json` con `$PWD` + script de init. Más allá de Pre/PostToolUse, existen otros 5 tipos de hooks para distintos momentos del ciclo de vida de Claude Code.

**SDK de agente** (Módulo 7): permite ejecutar Claude Code programáticamente. La función `query()` devuelve un async iterable con todos los mensajes del ciclo — no solo la respuesta final. Es lo que hace posible el patrón de revisión entre instancias visto en el Módulo 5.

| Mecanismo | Dónde vive | Para qué |
| --- | --- | --- |
| Hook `PreToolUse` | `settings.json` / `.local.json` | Bloquear o validar antes de actuar |
| Hook `PostToolUse` | `settings.json` / `.local.json` | Seguimiento y feedback después de actuar |
| Otros hooks | `settings.json` / `.local.json` | Reaccionar a momentos del ciclo de sesión |
| SDK (`query()`) | Código propio | Controlar Claude programáticamente |

---

### Mapa conceptual — Unidad 3

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 45, 'rankSpacing': 85}}}%%
flowchart LR
    U3((Unidad 3\nHooks y SDK))

    U3 --> HOOKS[Hooks]
    HOOKS --> H1[PreToolUse\nbloquea antes]
    HOOKS --> H2[PostToolUse\nfeedback después]
    HOOKS --> H3[Otros tipos\nStop · Session\nNotification · etc.]
    HOOKS --> HCONF[Configuración]
    HCONF --> HCONF1[settings.json\ncompartido]
    HCONF --> HCONF2[settings.local.json\npersonal · rutas absolutas]
    HCONF --> HCONF3[settings.example.json\n+ $PWD → portabilidad]
    HOOKS --> HLOGIC[Lógica del script]
    HLOGIC --> HLOGIC1[stdin → JSON\ntool_name + tool_input]
    HLOGIC --> HLOGIC2[exit 0 → permitir\nexit 2 → bloquear]
    HLOGIC --> HLOGIC3[jq . para\nexplorar estructura]

    U3 --> UTIL[Hooks útiles]
    UTIL --> UTIL1[tsc --noEmit\ntipo de errores]
    UTIL --> UTIL2[Detección\nde duplicados\nvía SDK]

    U3 --> SDK[SDK de agente]
    SDK --> SDK1[@anthropic-ai/\nclaude-agent-sdk]
    SDK --> SDK2[query prompt\nasync iterable]
    SDK --> SDK3[allowedTools\nmínimo privilegio]
    SDK --> SDK4[Todos los eventos\ndel ciclo]
```

---

## Resumen General del Curso

---

### Unidad 1 — ¿Qué es Claude Code?

Un asistente de codificación es un LLM combinado con un sistema de herramientas. El modelo solo genera texto, pero el sistema interpreta ese texto como instrucciones para leer archivos, editar código y ejecutar comandos. Claude destaca por su capacidad de uso de herramientas, lo que se traduce en mayor seguridad, extensibilidad y robustez en tareas complejas.

---

### Unidad 2 — Manos a la obra

El flujo de trabajo práctico con Claude Code gira en torno a tres ejes:

**Contexto:** `CLAUDE.md` es el system prompt permanente del proyecto. Hay tres ubicaciones (global, proyecto, local) y dos formas de referenciar archivos (`@` puntual en el chat, `@` permanente en `CLAUDE.md`).

**Herramientas de trabajo:** `/plan` para tareas amplias, `/effort` para tareas profundas, `ultrathink` para profundidad puntual. Capturas de pantalla con `Ctrl+V` para comunicar lo visual.

**Gestión de sesión:** `Escape` para interrumpir, `/rewind` para rebobinar, `/compact` para comprimir, `/clear` para empezar de cero. Comandos personalizados en `.claude/commands/` para automatizar flujos repetitivos. MCP para conectar herramientas externas. GitHub Actions para integración con el repositorio.

---

### Unidad 3 — Hooks y SDK

Los hooks extienden Claude Code con automatización propia. `PreToolUse` intercepta antes de que una tool corra; `PostToolUse` reacciona después. Los scripts reciben JSON por stdin y se comunican con códigos de salida. El patrón `settings.example.json` + script de init resuelve la tensión entre rutas absolutas seguras y archivos portables.

El SDK de agente (`@anthropic-ai/claude-agent-sdk`) permite controlar Claude Code programáticamente desde cualquier aplicación, con acceso a todos los eventos del ciclo y restricción de herramientas.

---

### Tabla de referencia rápida

| Necesito... | Herramienta |
| --- | --- |
| Dar contexto permanente al proyecto | `CLAUDE.md` + `/init` |
| Incluir un archivo en cada sesión | `@archivo` en `CLAUDE.md` |
| Planificar antes de implementar | `/plan` |
| Razonamiento profundo en una tarea | `/effort max` o `ultrathink` |
| Limpiar el contexto de la sesión | `/compact` o `/clear` |
| Automatizar un flujo repetitivo | Comando personalizado en `.claude/commands/` |
| Conectar Claude a un sistema externo | Servidor MCP |
| Integrar con el repo de GitHub | `/install-github-app` |
| Ejecutar algo automáticamente al usar una tool | Hook Pre/PostToolUse |
| Bloquear acceso a archivos sensibles | Hook `PreToolUse` + `permissions.deny` |
| Controlar Claude desde código propio | SDK `@anthropic-ai/claude-agent-sdk` |

---

## Mapa conceptual general

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 40, 'rankSpacing': 80}}}%%
flowchart LR
    ROOT((Code in\nAction))

    ROOT --> U1[Unidad 1\n¿Qué es\nClaude Code?]
    U1 --> U1A[LLM + herramientas]
    U1 --> U1B[Tool use\ncomo diferenciador]

    ROOT --> U2[Unidad 2\nManos a\nla obra]
    U2 --> U2A[Contexto\nCLAUDE.md]
    U2A --> U2A1[Global · proyecto · local]
    U2A --> U2A2[@archivo puntual\no permanente]
    U2 --> U2B[Herramientas\nde trabajo]
    U2B --> U2B1[/plan amplitud]
    U2B --> U2B2[/effort profundidad]
    U2B --> U2B3[ultrathink puntual]
    U2 --> U2C[Gestión\nde sesión]
    U2C --> U2C1[Escape · /rewind]
    U2C --> U2C2[/compact · /clear]
    U2 --> U2D[Extensiones]
    U2D --> U2D1[Comandos\npersonalizados]
    U2D --> U2D2[Servidores\nMCP]
    U2D --> U2D3[GitHub\nActions]

    ROOT --> U3[Unidad 3\nHooks y SDK]
    U3 --> U3A[Hooks]
    U3A --> U3A1[PreToolUse\nbloquear]
    U3A --> U3A2[PostToolUse\nfeedback]
    U3A --> U3A3[7 tipos\nen total]
    U3A --> U3A4[exit 0 / exit 2]
    U3 --> U3B[SDK de agente]
    U3B --> U3B1[query\nasync iterable]
    U3B --> U3B2[allowedTools]
    U3B --> U3B3[Revisión entre\ninstancias]
```
