# Comandos y Atajos de Claude Code

Claude Code es una CLI (interfaz de línea de comandos) para interactuar con Claude directamente desde la terminal, con capacidad de leer archivos, ejecutar código, buscar en el repositorio y más.

---

## 1. Comandos de inicio (CLI)

Comandos que se usan desde la terminal del sistema operativo para arrancar o configurar Claude Code.

| Comando | Descripción |
| --- | --- |
| `claude` | Inicia una sesión interactiva REPL (modo conversación) |
| `claude "pregunta"` | Ejecuta una sola consulta sin entrar al REPL |
| `claude -p "prompt"` | Modo headless: responde y sale (no interactivo) |
| `claude --continue` | Reanuda la última conversación guardada |
| `claude --resume` | Muestra un selector para reanudar una conversación anterior |
| `claude --model <id>` | Especifica el modelo a usar (ej: `claude-opus-4-7`) |
| `claude --version` | Muestra la versión instalada de Claude Code |
| `claude --help` | Muestra la ayuda de la CLI |
| `claude --no-color` | Desactiva los colores en la salida |
| `claude --output-format json` | Salida en formato JSON (útil en scripts) |
| `claude --output-format stream-json` | Salida JSON en streaming línea por línea |
| `claude --max-turns <n>` | Límite de turnos agente en modo no interactivo |
| `claude --add-dir <ruta>` | Agrega una carpeta extra al contexto de trabajo |
| `claude --dangerously-skip-permissions` | Omite todos los prompts de permisos (usar con cuidado) |

### Pipes e integración con la terminal

```bash
# Pasar contenido por stdin
cat archivo.py | claude -p "¿Qué hace este código?"

# Leer un archivo y hacer una pregunta
claude -p "Resumí este log" < error.log

# Encadenar con otros comandos
git diff | claude -p "Escribí el mensaje de commit para este diff"
```

---

## 2. Comandos de barra (slash commands) — dentro del REPL

Estos comandos se usan **dentro de la sesión interactiva** de Claude Code, escribiéndolos directamente en el chat.

### Gestión de sesión

| Comando | Descripción |
| --- | --- |
| `/help` | Muestra todos los comandos disponibles |
| `/clear` | Limpia el historial de conversación actual |
| `/compact` | Compacta el historial (resume la conversación para liberar contexto) |
| `/compact <instrucción>` | Compacta con instrucciones específicas de resumen |
| `/exit` o `/quit` | Sale de la sesión de Claude Code |

### Modelos y modos

| Comando | Descripción |
| --- | --- |
| `/model` | Muestra el modelo actualmente en uso |
| `/model <id>` | Cambia el modelo (ej: `/model claude-opus-4-7`) |
| `/fast` | Activa/desactiva el modo rápido (Opus con output más veloz) |

### Contexto y archivos

| Comando | Descripción |
| --- | --- |
| `/add-dir <ruta>` | Agrega una carpeta extra al contexto de la sesión |
| `/init` | Genera un archivo `CLAUDE.md` con documentación del proyecto |

### Memoria

| Comando | Descripción |
| --- | --- |
| `/memory` | Muestra las memorias guardadas del usuario |

### Configuración

| Comando | Descripción |
| --- | --- |
| `/config` | Abre el menú de configuración interactivo |
| `/permissions` | Muestra y edita los permisos de herramientas activos |

### Revisión y agentes

| Comando | Descripción |
| --- | --- |
| `/review` | Revisa el pull request actual |
| `/ultrareview` | Revisión multi-agente en la nube del branch o PR actual |
| `/ultrareview <PR#>` | Revisión multi-agente de un PR de GitHub específico |
| `/security-review` | Revisión de seguridad de los cambios del branch actual |

### Tareas y planificación

| Comando | Descripción |
| --- | --- |
| `/plan` | Entra en modo planificación (diseña antes de ejecutar) |
| `/loop <intervalo> <comando>` | Ejecuta un comando en bucle con un intervalo dado |

### MCP (Model Context Protocol)

| Comando | Descripción |
| --- | --- |
| `/mcp` | Lista los servidores MCP conectados y su estado |

---

## 3. Atajos de teclado — dentro del REPL

Atajos para controlar la sesión interactiva sin necesidad de escribir comandos.

### Entrada de texto

| Atajo | Descripción |
| --- | --- |
| `Enter` | Envía el mensaje al modelo |
| `Shift + Enter` | Inserta un salto de línea sin enviar |
| `Ctrl + J` | Alternativa para salto de línea (igual que Shift+Enter) |
| `Tab` | Autocompletado de comandos `/` o rutas de archivo |

### Historial y navegación

| Atajo | Descripción |
| --- | --- |
| `↑` / `↓` | Navega por el historial de mensajes enviados |
| `Ctrl + R` | Búsqueda inversa en el historial de mensajes |
| `Ctrl + A` | Va al inicio de la línea |
| `Ctrl + E` | Va al final de la línea |
| `Ctrl + W` | Borra la palabra anterior al cursor |
| `Ctrl + U` | Borra toda la línea desde el cursor hacia el inicio |
| `Ctrl + K` | Borra desde el cursor hasta el final de la línea |
| `Alt + ←` / `Alt + →` | Mueve el cursor palabra por palabra |

### Control del proceso

| Atajo | Descripción |
| --- | --- |
| `Ctrl + C` | Interrumpe la respuesta en curso o cancela la entrada actual |
| `Ctrl + D` | Sale de Claude Code (si la línea está vacía) |
| `Escape` | Cancela la entrada o cierra un menú |

### Modos especiales

| Atajo | Descripción |
| --- | --- |
| `Ctrl + \` | Activa/desactiva el modo verbose (muestra herramientas internas) |

---

## 4. Modos de permisos

Claude Code puede ejecutarse en distintos niveles de autonomía. Se configuran al iniciar o en `settings.json`.

| Modo | Descripción |
| --- | --- |
| **Default** | Pide confirmación para acciones destructivas o que afectan archivos |
| **Auto-approve** | Aprueba automáticamente herramientas de lectura y búsqueda |
| `--dangerously-skip-permissions` | Omite todos los prompts de permisos (solo para entornos controlados) |

---

## 5. Variables de entorno útiles

| Variable | Descripción |
| --- | --- |
| `ANTHROPIC_API_KEY` | Clave de API de Anthropic (requerida) |
| `CLAUDE_MODEL` | Modelo por defecto para las sesiones |
| `CLAUDE_MAX_TOKENS` | Límite de tokens en la respuesta |
| `ANTHROPIC_BASE_URL` | URL base alternativa (para proxies o entornos enterprise) |
| `NO_COLOR` | Desactiva los colores en toda la salida de la CLI |

---

## 6. Archivos de configuración

| Archivo | Ubicación | Descripción |
| --- | --- | --- |
| `CLAUDE.md` | Raíz del proyecto | Instrucciones del proyecto que Claude lee automáticamente |
| `settings.json` | `.claude/settings.json` (proyecto) | Permisos, hooks y configuración local del proyecto |
| `settings.json` | `~/.claude/settings.json` (usuario) | Configuración global del usuario |
| `MEMORY.md` | `~/.claude/projects/<ruta>/memory/` | Índice de memorias persistentes del usuario |
| `keybindings.json` | `~/.claude/keybindings.json` | Atajos de teclado personalizados |

---

## 7. Flags de salida (exit codes) en modo no interactivo

| Código | Significado |
| --- | --- |
| `0` | Ejecución exitosa |
| `1` | Error general |
| `2` | Tarea interrumpida por el usuario (Ctrl+C) |
| `3` | Límite de turnos alcanzado (`--max-turns`) |

---

## 8. Ejemplos de uso combinado

```bash
# Hacer code review de los cambios no commiteados
git diff HEAD | claude -p "Hacé un code review de estos cambios"

# Analizar un archivo de log y guardar el resultado
claude -p "Resumí los errores críticos" < server.log > resumen_errores.md

# Generar tests para una función específica
cat src/utils.py | claude -p "Generá tests unitarios en pytest para estas funciones"

# Consulta con modelo específico y salida JSON
claude --model claude-opus-4-7 --output-format json -p "¿Cuáles son las mejores prácticas de Python?"

# Reanudar una sesión anterior
claude --resume
```

---

## 9. Tabla resumen rápida (cheat sheet)

| Categoría | Comando/Atajo | Efecto |
| --- | --- | --- |
| Inicio | `claude` | Abre sesión interactiva |
| Inicio rápido | `claude "pregunta"` | Una sola respuesta y sale |
| Ayuda | `/help` | Lista todos los slash commands |
| Limpiar | `/clear` | Borra el historial de la sesión |
| Compactar | `/compact` | Resume la conversación para liberar contexto |
| Modelo | `/model <id>` | Cambia el modelo en tiempo real |
| Salir | `/exit` o `Ctrl+D` | Cierra Claude Code |
| Interrumpir | `Ctrl+C` | Para la respuesta en curso |
| Nueva línea | `Shift+Enter` | Salto de línea sin enviar |
| Historial | `↑` / `↓` | Navega mensajes anteriores |
| Verbose | `Ctrl+\` | Muestra herramientas internas |
