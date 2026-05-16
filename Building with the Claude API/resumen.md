# Building with the Claude API — Resumen

---

## Unidad 1: Introducción

### Módulo 1: Introducción a Claude y sus Modelos

#### 1.1 La familia de modelos Claude

| Modelo | Inteligencia | Costo | Velocidad |
|---|---|---|---|
| **Claude Opus** | ⭐⭐⭐ Máxima | Alto | Moderada |
| **Claude Sonnet** | ⭐⭐ Alta | Medio | Rápida |
| **Claude Haiku** | ⭐ Moderada | Bajo | La más rápida |

#### Claude Opus
- Altamente inteligente, más caro, mayor latencia
- Ideal para: arquitecturas a gran escala, tareas complejas, planificación estratégica, reasoning avanzado

#### Claude Sonnet
- Fuerte balance entre inteligencia, costo y velocidad
- Ideal para: codificación, documentos, análisis de datos, imágenes, automatización

#### Claude Haiku
- Inteligencia moderada, bajo costo, máxima velocidad
- Ideal para: completado rápido de código, moderación, traducción, Q&A, alto volumen

---

#### 1.2 Cómo elegir el modelo correcto

El eje de decisión es **Inteligencia ←→ Costo/Velocidad**:

| Necesidad | Modelo recomendado |
|---|---|
| Máxima capacidad de razonamiento | Opus |
| Balance entre calidad y eficiencia | Sonnet |
| Velocidad y bajo costo a gran escala | Haiku |

---

### Módulo 2: Ciclo de vida de una solicitud a Claude

Comprender el ciclo de vida completo de las solicitudes ayuda a tomar mejores decisiones arquitectónicas y a depurar problemas con mayor eficacia.

#### 2.1 El flujo de solicitud de cinco pasos

Cada interacción con Claude sigue un patrón predecible con cinco fases:

1. **Solicitud al servidor** — el cliente envía la solicitud a tu backend.
2. **Solicitud a la API de Anthropic** — tu servidor llama a la API.
3. **Procesamiento del modelo** — Claude procesa la entrada y genera la respuesta.
4. **Respuesta al servidor** — la API devuelve el resultado a tu backend.
5. **Respuesta al cliente** — tu servidor envía la respuesta al usuario final.

![alt text](imagenes/image-8.png)

---

#### 2.2 Por qué necesitas un servidor

**Nunca** se debe llamar a la API de Anthropic directamente desde el cliente:

- Las solicitudes requieren una **clave API secreta**.
- Exponerla en el cliente crea una grave vulnerabilidad de seguridad.
- Cualquiera podría extraerla y hacer solicitudes no autorizadas.

La aplicación cliente envía solicitudes a tu propio servidor, que se comunica con Anthropic usando la clave almacenada de forma segura.

---

#### 2.3 Realizar solicitudes a la API

Puedes usar un SDK oficial (Python, TypeScript, JavaScript, Go, Ruby) o realizar peticiones HTTP directas.

Campos esenciales de cada solicitud:

| Campo | Descripción |
|---|---|
| **API Key** | Identifica tu solicitud ante Anthropic |
| **Model** | Nombre del modelo (ej. `claude-3-sonnet`) |
| **Messages** | Lista con el texto introducido por el usuario |
| **Max tokens** | Límite de tokens que Claude puede generar |

![alt text](imagenes/image-9.png)

---

#### 2.4 Dentro del proceso de Claude

Una vez recibida la solicitud, Claude procesa la entrada en cuatro etapas:

#### Tokenización
Divide el texto en fragmentos pequeños llamados **tokens** (palabras completas, partes de palabras, espacios o símbolos).

#### Embedding (incrustación)
Cada token se convierte en un **vector**: una lista de números que representa todos los posibles significados de esa palabra. Por ejemplo, "cuántico" puede referirse a:

- Una unidad discreta de magnitud física
- Conceptos de mecánica cuántica
- Algo extremadamente pequeño o subatómico
- Aplicaciones de computación cuántica

#### Contextualización
Claude refina cada vector basándose en las palabras circundantes para determinar el significado más probable en contexto.

#### Generación
Los embeddings contextualizados pasan por una capa de salida que calcula probabilidades para cada posible palabra siguiente. Claude combina probabilidad con aleatoriedad controlada para generar respuestas naturales y variadas. Tras seleccionar cada token, lo añade a la secuencia y repite el proceso.

![alt text](imagenes/image-10.png)
![alt text](imagenes/image-12.png)

---

#### 2.5 Cuándo Claude deja de generar

Después de cada token, Claude evalúa si continuar:

- **Máximo de tokens alcanzado** — llegó al límite especificado.
- **Final natural** — generó un token de fin de secuencia.
- **Stop sequence** — encontró una frase de parada predefinida.

---

#### 2.6 La respuesta de la API

Cuando termina la generación, la API devuelve una respuesta estructurada con:

- **Message** — el texto generado.
- **Usage** — recuento de tokens de entrada y salida.
- **Stop reason** — motivo por el que terminó la generación.

El servidor reenvía el texto generado a la aplicación cliente, donde aparece en la UI.
![alt text](imagenes/image-11.png)
![alt text](imagenes/image.png)

---

#### 2.7 Conclusiones clave

Comprender este flujo permite:

- Diseñar arquitecturas seguras que protejan las claves API.
- Establecer límites de tokens adecuados al caso de uso.
- Gestionar distintos *stop reasons* en la lógica de la aplicación.
- Depurar problemas identificando en qué etapa del flujo ocurren.

---

## Unidad 2: Acceso a Claude mediante API

### Módulo 3: Hacer una solicitud (primera llamada a la API)

Realizar la primera solicitud a la API de Anthropic es sencillo una vez configurado el entorno y comprendida la estructura básica.

#### 3.1 Configuración del entorno

Instalar dependencias en el notebook:

```
%pip install anthropic python-dotenv
```

Crear un archivo `.env` en el mismo directorio para guardar la clave API de forma segura:

```
ANTHROPIC_API_KEY="your-api-key-here"
```

> Esto mantiene la clave fuera del código y evita subirla por accidente al control de versiones. Siempre agrega `.env` a tu `.gitignore`.

Cargar variables de entorno y crear el cliente:

```python
from dotenv import load_dotenv
load_dotenv()          # Lee el archivo .env y carga las variables de entorno (incluida la API key)

from anthropic import Anthropic

client = Anthropic()           # Crea el cliente; toma la API key de la variable de entorno automáticamente
model = "claude-sonnet-4-0"    # Define el modelo a usar en todas las solicitudes
```

---

#### 3.2 La función `create`

La base para enviar solicitudes es `client.messages.create()`. Tres parámetros clave:

| Parámetro | Descripción |
|---|---|
| **model** | Nombre del modelo Claude a usar |
| **max_tokens** | Límite de seguridad para la longitud de la respuesta (no un objetivo) |
| **messages** | Historial de conversación enviado a Claude |

> `max_tokens` es un mecanismo de seguridad: si lo fijas en 1000, Claude se detendrá al llegar a ese número aunque tenga más que decir. Claude **no intenta alcanzar** el límite; simplemente escribe lo que considera apropiado y se detiene si toca el máximo.

---

#### 3.3 Comprender los mensajes

Los mensajes representan la conversación, similar a un chat. Dos tipos:

- **User messages** — contenido que envías a Claude (escrito por humanos).
- **Assistant messages** — respuestas generadas por Claude.

Cada mensaje es un diccionario con un `role` (`"user"` o `"assistant"`) y un `content` (el texto).

---

#### 3.4 Primera solicitud

```python
message = client.messages.create(
    model=model,        # Modelo a usar (definido arriba)
    max_tokens=1000,    # Límite de tokens en la respuesta; Claude se detiene aquí si no terminó antes
    messages=[
        {
            "role": "user",                                              # Quién envía el mensaje
            "content": "What is quantum computing? Answer in one sentence"  # El texto del mensaje
        }
    ]
)
```

Claude procesa la solicitud y devuelve un objeto de respuesta con el texto generado y metadatos.

---

#### 3.5 Extraer la respuesta

El objeto de respuesta trae mucha información, pero normalmente solo necesitas el texto generado:

```python
message.content[0].text
# message.content es una lista de bloques de contenido
# [0] accede al primer (y generalmente único) bloque
# .text extrae el texto generado como string limpio
```

Esto entrega una salida limpia y legible (por ejemplo, una definición de computación cuántica en una sola oración).

Con estas bases puedes empezar a experimentar con distintos prompts y construir interacciones más complejas con Claude.

---

### Módulo 4: Conversaciones de múltiples turnos

#### 4.1 Claude no tiene memoria

Claude **no almacena historial de conversaciones**. Cada solicitud es completamente independiente — no recuerda ningún intercambio anterior.

> Si le preguntas "¿Qué es la computación cuántica?" y luego envías "Escribe otra frase", Claude escribirá sobre algo aleatorio porque no recuerda el contexto previo.

---

#### 4.2 Cómo mantener el contexto

Para que Claude "recuerde" la conversación, tú debes gestionar el estado manualmente:

| Paso | Acción |
|---|---|
| 1 | Mantener una lista acumulativa de mensajes en tu código |
| 2 | Enviar el historial completo con cada nueva solicitud |

#### Flujo de una conversación de varios turnos

1. Enviar el mensaje inicial de usuario a Claude.
2. Tomar la respuesta y agregarla al historial como mensaje `assistant`.
3. Añadir la siguiente pregunta como mensaje `user`.
4. Enviar **todo el historial** en la próxima solicitud.

![alt text](imagenes/image-1.png)

---

#### 4.3 Funciones auxiliares

Para simplificar la gestión del historial:

```python
def add_user_message(messages, text):
    messages.append({"role": "user", "content": text})      # Agrega un turno del usuario al historial

def add_assistant_message(messages, text):
    messages.append({"role": "assistant", "content": text}) # Agrega la respuesta de Claude al historial

def chat(messages):
    message = client.messages.create(
        model=model,
        max_tokens=1000,
        messages=messages,   # Envía el historial completo en cada solicitud
    )
    return message.content[0].text   # Devuelve solo el texto de la respuesta
```

---

#### 4.4 Ejemplo completo

```python
messages = []   # Historial vacío al inicio de la conversación

add_user_message(messages, "Define quantum computing in one sentence")
answer = chat(messages)         # Primera llamada: solo hay un mensaje de usuario

add_assistant_message(messages, answer)          # Guarda la respuesta de Claude en el historial
add_user_message(messages, "Write another sentence")  # Agrega la segunda pregunta

final_answer = chat(messages)   # Segunda llamada: envía los 3 mensajes juntos para mantener contexto
```

Ahora Claude entiende que "Write another sentence" se refiere a la definición de computación cuántica, porque recibe el historial completo.

---

### Módulo 5: System Prompts

#### 5.1 ¿Qué son y para qué sirven?

Los **system prompts** permiten personalizar cómo Claude responde: su tono, rol, restricciones y enfoque. En lugar de respuestas genéricas, obtenés comportamiento adaptado a tu caso de uso.

> **Ejemplo:** Un tutor de matemáticas no debería dar la respuesta directa — debería guiar al estudiante paso a paso. Un system prompt hace exactamente eso.

| Sin system prompt | Con system prompt de tutor |
|---|---|
| Da la solución completa de inmediato | Hace preguntas orientadoras |
| Comportamiento genérico | Rol especializado y coherente |

---

#### 5.2 Estructura básica

```python
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""
# El system prompt define el rol y comportamiento de Claude para toda la conversación

client.messages.create(
    model=model,
    messages=messages,
    max_tokens=1000,
    system=system_prompt    # Se pasa como parámetro separado, no dentro de messages
)
```

---

#### 5.3 Función de chat flexible

La API de Claude **no acepta `system=None`**, por lo que hay que incluir el parámetro condicionalmente:

```python
def chat(messages, system=None):
    params = {              # Parámetros base que siempre se envían
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
    }
    if system:
        params["system"] = system   # Solo se agrega si se proporcionó (la API no acepta system=None)

    message = client.messages.create(**params)   # ** desempaqueta el diccionario como argumentos nombrados
    return message.content[0].text
```

Uso:

```python
# Sin system prompt
answer = chat(messages)

# Con system prompt
system = "You are a patient math tutor. Guide students step by step."
answer = chat(messages, system=system)
```

Los system prompts son esenciales para que una aplicación de IA se comporte de forma coherente y apropiada a su propósito.

---

### Módulo 6: Temperatura

#### 6.1 ¿Qué es la temperatura?

La **temperatura** es un parámetro decimal entre `0.0` y `1.0` que controla qué tan predecibles o creativas son las respuestas de Claude. Influye directamente en la etapa de **muestreo** del proceso de generación.

> La tokenización y la predicción de probabilidades ocurren igual siempre — la temperatura solo afecta cómo se elige el siguiente token a partir de esas probabilidades.

---

![alt text](imagenes/image-2.png)
![alt text](imagenes/image-3.png)

#### 6.2 Cómo afecta la temperatura a las probabilidades

| Temperatura | Efecto |
| --- | --- |
| **Cercana a 0** | Claude casi siempre elige el token más probable → respuestas deterministas |
| **Cercana a 1** | Las probabilidades se distribuyen más uniformemente → respuestas variadas y creativas |

Ejemplo: para el token siguiente a "¿Qué opinas?", Claude puede asignar 30 % a "about", 20 % a "would", 10 % a "of", etc. A temperatura 0.0 siempre elige "about"; a temperatura 1.0 cualquiera de ellos puede ganar.

---

#### 6.3 Temperatura según el tipo de tarea

| Rango | Tipo de tarea | Ejemplos |
| --- | --- | --- |
| **0.0 – 0.3** (baja) | Precisión y consistencia | Respuestas factuales, codificación, extracción de datos, moderación |
| **0.4 – 0.7** (media) | Balance entre calidad y variedad | Resúmenes, contenido educativo, resolución de problemas, escritura creativa acotada |
| **0.8 – 1.0** (alta) | Creatividad y diversidad | Lluvia de ideas, marketing, escritura libre, generación de humor |

---

![alt text](imagenes/image-4.png)

#### 6.4 Agregar temperatura a la función de chat

```python
def chat(messages, system=None, temperature=1.0):   # temperature=1.0 es el valor por defecto
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature   # Controla la creatividad: 0.0 = determinista, 1.0 = creativo
    }
    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

Los únicos cambios respecto a la versión anterior son añadir `temperature=1.0` como parámetro e incluirlo en el diccionario `params`.

---

#### 6.5 Conclusiones clave

- La temperatura **no garantiza** resultados diferentes; solo cambia la probabilidad de obtenerlos.
- Incluso a temperatura alta, Claude puede producir respuestas similares ocasionalmente.
- Ajustar la temperatura al caso de uso específico es más útil que dejarlo en el valor por defecto.

| Situación | Temperatura recomendada |
| --- | --- |
| Necesito respuestas consistentes y objetivas | Baja (0.0 – 0.3) |
| Quiero lluvia de ideas creativa | Alta (0.8 – 1.0) |
| Tarea general equilibrada | Media (0.4 – 0.7) |

![alt text](imagenes/image-5.png)

---

### Módulo 7: Transmisión de respuesta (Streaming)

#### 7.1 El problema con las respuestas estándar

En una configuración típica, el servidor espera la respuesta **completa** de Claude antes de enviar algo al cliente. Esto genera una demora de 10 a 30 segundos en la que el usuario no recibe ninguna señal de que algo está pasando.

---

#### 7.2 Cómo funciona el streaming

Con streaming habilitado, Claude envía fragmentos de texto a medida que los genera. Tu servidor puede reenviarlos al cliente en tiempo real, permitiendo que el usuario vea la respuesta aparecer palabra por palabra — igual que en claude.ai.

Todos estos fragmentos son parte de una única solicitud a la API.

---

#### 7.3 Tipos de eventos del stream

| Evento | Descripción |
| --- | --- |
| **MessageStart** | Inicio de un nuevo mensaje |
| **ContentBlockStart** | Inicio de un bloque de contenido (texto, tool use, etc.) |
| **ContentBlockDelta** | Fragmento del texto generado |
| **ContentBlockStop** | El bloque de contenido actual finalizó |
| **MessageDelta** | El mensaje actual está completo |
| **MessageStop** | Fin de la información del mensaje |

![alt text](imagenes/image-7.png)

Los eventos `ContentBlockDelta` contienen el texto que se muestra al usuario.

![alt text](imagenes/image-6.png)

---

#### 7.4 Implementación básica

```python
stream = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    stream=True     # Activa el modo streaming: Claude envía fragmentos en lugar de esperar la respuesta completa
)

for event in stream:
    print(event)    # Imprime cada evento raw (incluye metadata, no solo texto)
```

---

#### 7.5 Interfaz simplificada con `messages.stream`

En lugar de parsear manualmente los eventos, el SDK ofrece una interfaz simplificada que extrae solo el texto:

```python
with client.messages.stream(   # Interfaz de alto nivel del SDK; maneja el stream automáticamente
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:     # Itera solo sobre los fragmentos de texto (filtra el resto de eventos)
        print(text, end="")             # end="" evita saltos de línea entre fragmentos; el texto fluye continuo
```

Esto filtra automáticamente todo excepto el contenido de texto, que es lo que normalmente se necesita para mostrar al usuario.

---

#### 7.6 Obtener el mensaje completo al final

Después del streaming se puede recuperar el mensaje ensamblado completo, útil para guardarlo en base de datos o procesarlo:

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        pass    # Aquí iría el código para enviar cada fragmento al cliente en tiempo real

    # Una vez terminado el stream, get_final_message() devuelve el objeto Message completo
    # (con el texto ensamblado, uso de tokens, stop_reason, etc.)
    final_message = stream.get_final_message()
```

Esto da lo mejor de ambos mundos: streaming en tiempo real para el usuario y un objeto de mensaje completo para la lógica de la aplicación.

---

### Módulo 8: Datos estructurados

#### 8.1 El problema

Cuando le pides a Claude que genere JSON u otro contenido estructurado, por defecto añade texto explicativo y lo envuelve en bloques de código Markdown:

````markdown
```json
{
  "source": ["aws.ec2"],
  ...
}
```
This rule captures EC2 instance state changes when instances start running.
````

Para aplicaciones donde el usuario necesita copiar el dato directamente (ej. una regla de AWS EventBridge), esto genera fricción innecesaria.

---

#### 8.2 La solución: prefill + stop sequences

Combinar el **prellenado del mensaje del asistente** con **secuencias de parada** permite extraer exactamente el contenido deseado:

```python
messages = []

add_user_message(messages, "Generate a very short event bridge rule as json")
add_assistant_message(messages, "```json")   # Prefill: Claude "cree" que ya empezó un bloque de código

# stop_sequences hace que Claude deje de generar cuando intente escribir el cierre ```
text = chat(messages, stop_sequences=["```"])
```

**Cómo funciona:**

| Paso | Qué ocurre |
| --- | --- |
| 1 | El mensaje de usuario indica qué generar |
| 2 | El prefill hace que Claude crea que ya abrió un bloque de código |
| 3 | Claude continúa escribiendo solo el contenido JSON |
| 4 | Cuando Claude intenta cerrar con ` ``` `, la stop sequence detiene la generación |

El resultado es JSON limpio sin ningún formato adicional.

---

#### 8.3 Procesar la respuesta

La respuesta puede tener saltos de línea sobrantes; `json.loads` con `strip()` los elimina:

```python
import json

clean_json = json.loads(text.strip())
# text.strip() elimina espacios y saltos de línea al inicio y al final
# json.loads() convierte el string JSON en un diccionario Python
```

---

#### 8.4 Más allá de JSON

La misma técnica aplica a cualquier contenido estructurado donde solo se quiere el dato, sin explicaciones:

- Fragmentos de código Python
- Listas con viñetas
- Datos CSV
- Cualquier formato donde Claude envuelva naturalmente el contenido

La clave es identificar el patrón de apertura que Claude usaría y usarlo como prefill y como stop sequence de cierre.

---

## Unidad 3: Evaluación inmediata

### Módulo 9: Evaluación de indicaciones

#### 9.1 Ingeniería vs. evaluación de indicaciones

Trabajar con Claude implica dos disciplinas distintas:

| Concepto | Qué es |
| --- | --- |
| **Ingeniería de indicaciones** | Técnicas para redactar mejores prompts (few-shot, XML, etc.) |
| **Evaluación de indicaciones** | Medir la eficacia de los prompts mediante pruebas automatizadas |

La ingeniería te dice *cómo* escribir; la evaluación te dice *qué tan bien* funciona lo que escribiste.

![alt text](imagenes/image-13.png)

---

#### 9.2 Tres opciones tras escribir una indicación

| Opción | Descripción | Riesgo |
| --- | --- | --- |
| **1** | Probar una sola vez y decidir si es suficiente | Alto — falla ante entradas inesperadas en producción |
| **2** | Probar varias veces y ajustar uno o dos casos borde | Medio — los usuarios siempre encontrarán casos no considerados |
| **3** | Someter la indicación a un proceso de evaluación con métricas objetivas | Bajo — mayor inversión inicial, mucha más confianza en la fiabilidad |

---

#### 9.3 Por qué las opciones 1 y 2 son trampas

Es natural escribir un prompt para una aplicación seria y no probarlo lo suficiente. Los ingenieros tienden a subestimar la variedad de entradas que generarán los usuarios reales. Lo que parece sólido en pruebas limitadas puede fallar rápidamente en producción.

---

#### 9.4 El enfoque de evaluación primero (Opción 3)

Someter la indicación a un proceso de evaluación sistemático permite:

- Identificar debilidades antes de que lleguen a producción
- Comparar objetivamente distintas versiones del mismo prompt
- Iterar con confianza basándose en mejoras medibles
- Construir aplicaciones de IA más fiables

Requiere mayor inversión inicial en infraestructura de pruebas, pero los beneficios en robustez y confiabilidad justifican el esfuerzo.

---

#### 9.5 Flujo de trabajo de evaluación típico

El proceso consta de cinco pasos que se repiten iterativamente:

#### Paso 1 — Redactar una indicación inicial

```python
prompt = f"""
Please answer the user's question:

{question}
"""
```

Esta es la línea de base que se va a medir y mejorar.

#### Paso 2 — Crear un conjunto de datos de evaluación

Reúne ejemplos representativos de las entradas que el prompt recibirá en producción. Pueden ser decenas, cientos o miles de registros. También puedes usar Claude para generarlos.

Ejemplo mínimo:

- "What's 2+2?"
- "How do you make oatmeal?"
- "How far away is the Moon?"

#### Paso 3 — Pasar cada entrada por Claude

Combina cada ejemplo con la plantilla de la indicación y envíalo a Claude para obtener una respuesta por cada registro del dataset.

#### Paso 4 — Calificar las respuestas

Un evaluador (modelo o código) analiza cada par pregunta-respuesta y asigna una puntuación, típicamente del 1 al 10. El promedio da una métrica objetiva de referencia.

Ejemplo:

| Pregunta | Puntuación |
| --- | --- |
| Matemáticas (2+2) | 10 |
| Avena | 4 |
| Distancia a la Luna | 9 |
| **Promedio** | **7.66** |

#### Paso 5 — Modificar la indicación y repetir

Con la métrica de referencia en mano, ajusta el prompt y vuelve a correr todo el proceso. Si la nueva puntuación es más alta, el cambio fue una mejora real.

```python
prompt = f"""
Please answer the user's question:

{question}

Answer the question with ample detail
"""
```

Nueva puntuación promedio: **8.7** → la instrucción adicional mejoró el rendimiento.

---

#### 9.6 La ventaja del enfoque

Este ciclo sistemático elimina las conjeturas de la ingeniería de indicaciones:

- Las comparaciones son numéricas y objetivas
- Siempre se usa la versión con mejor puntuación
- Se puede seguir iterando con confianza en que cada cambio es una mejora medible, no solo una variación diferente

---

#### 9.7 Generación de conjuntos de datos de prueba

#### Objetivo de ejemplo

Construir un evaluador para una indicación que ayude a usuarios a escribir código específico de AWS. Los formatos de salida esperados son:

- Código Python
- Archivos de configuración JSON
- Expresiones regulares

La indicación inicial (v1):

```python
prompt = f"""
Please provide a solution to the following task:
{task}
"""
```

---

#### Generar el dataset con Claude

En lugar de crear los casos de prueba manualmente, se puede usar un modelo rápido (Haiku) para generarlos automáticamente:

```python
def generate_dataset():
    prompt = """
Generate an evaluation dataset for a prompt evaluation. The dataset will be used to evaluate
prompts that generate Python, JSON, or Regex specifically for AWS-related tasks.
Generate an array of JSON objects, each representing a task that requires Python, JSON, or a Regex.

Example output:
[
  {"task": "Description of task"},
  ...
]

* Focus on tasks solvable by a single Python function, JSON object, or regex
* Focus on tasks that do not require much code

Please generate 3 objects.
"""
    messages = []
    add_user_message(messages, prompt)
    add_assistant_message(messages, "```json")  # Prefill para forzar respuesta JSON limpia
    text = chat(messages, stop_sequences=["```"])  # Detiene la generación antes del cierre ```
    return json.loads(text)   # Convierte el string en lista de diccionarios Python
```

> Se usa prefill + stop sequence para extraer el JSON limpio directamente. Si el modelo no soporta prefill, reemplazar por un system prompt que indique devolver solo JSON.

---

#### Guardar el dataset

Una vez generado, se persiste en disco para reutilizarlo en cada ciclo de evaluación:

```python
dataset = generate_dataset()   # Llama a Claude para generar los casos de prueba

with open('dataset.json', 'w') as f:
    json.dump(dataset, f, indent=2)   # indent=2 da formato legible con sangría al JSON guardado
```

Esto crea un archivo `dataset.json` en el mismo directorio del notebook, listo para ser cargado en las siguientes etapas del flujo de evaluación.

---

#### 9.8 Ejecutar la evaluación

El flujo es: dataset → combinar con prompt → enviar a Claude → calificar resultado.

#### Tres funciones principales

**`run_prompt`** — combina el caso de prueba con la plantilla y llama a Claude:

```python
def run_prompt(test_case):
    prompt = f"""
Please solve the following task:

{test_case["task"]}   
"""
    # test_case["task"] inserta la descripción del caso de prueba en la plantilla del prompt
    messages = []
    add_user_message(messages, prompt)
    return chat(messages)   # Devuelve el texto de la respuesta de Claude
```

**`run_test_case`** — ejecuta un caso y califica el resultado:

```python
def run_test_case(test_case):
    output = run_prompt(test_case)
    score = 10  # placeholder — se reemplaza con lógica real de calificación
    return {
        "output": output,
        "test_case": test_case,
        "score": score
    }
```

**`run_eval`** — coordina todo el proceso sobre el dataset completo:

```python
def run_eval(dataset):
    results = []
    for test_case in dataset:
        result = run_test_case(test_case)
        results.append(result)
    return results
```

---

#### Ejecutar y examinar resultados

```python
with open("dataset.json", "r") as f:
    dataset = json.load(f)   # Carga el dataset guardado como lista de diccionarios Python

results = run_eval(dataset)
print(json.dumps(results, indent=2))   # Imprime los resultados formateados con sangría para leerlos fácil
```

Cada objeto de resultados contiene:

| Campo | Descripción |
| --- | --- |
| `output` | Respuesta completa de Claude |
| `test_case` | El caso de prueba original |
| `score` | Puntuación de evaluación (por ahora fija en 10) |

> Procesar un dataset completo con Haiku puede tomar ~30 segundos. La puntuación fija de `10` es un placeholder — la lógica real de calificación se agrega en el siguiente paso.

---

#### 9.9 Calificación basada en modelos

Un evaluador toma la salida del modelo y devuelve una señal objetiva de calidad, generalmente un número del 1 al 10.

#### Tipos de evaluadores

| Tipo | Descripción | Mejor para |
| --- | --- | --- |
| **Código** | Lógica programática personalizada | Longitud, sintaxis, palabras clave |
| **Modelo** | Otro modelo de IA evalúa la salida | Calidad, utilidad, seguimiento de instrucciones |
| **Humano** | Revisión manual | Calidad general, profundidad, relevancia |

![alt text](imagenes/image-14.png)

---

#### Criterios de evaluación para generación de código AWS

Antes de implementar el evaluador, definir criterios claros:

| Criterio | Evaluador recomendado |
| --- | --- |
| **Formato** — solo Python, JSON o regex, sin explicación | Código |
| **Sintaxis válida** — el código generado compila/parsea | Código |
| **Seguimiento de tarea** — responde directamente lo que se pidió | Modelo |

---

#### Implementar el evaluador por modelo

```python
def grade_by_model(test_case, output):
    eval_prompt = f"""
Eres un revisor experto de código. Evalúa esta solución generada por IA.

Tarea: {test_case["task"]}
Solución: {output}

Devuelve un objeto JSON con:
- "strengths": lista de 1-3 fortalezas clave
- "weaknesses": lista de 1-3 áreas de mejora
- "reasoning": explicación concisa de tu evaluación
- "score": número entre 1 y 10
"""
    messages = []
    add_user_message(messages, eval_prompt)
    add_assistant_message(messages, "```json")

    eval_text = chat(messages, stop_sequences=["```"])
    return json.loads(eval_text)
```

> Pedir `strengths`, `weaknesses` y `reasoning` junto con el puntaje es clave. Sin ese contexto, los modelos tienden a dar puntajes mediocres alrededor de 6 sin mucha variación.

---

#### Integrar la calificación en el flujo

```python
def run_test_case(test_case):
    output = run_prompt(test_case)         # 1. Obtiene la respuesta de Claude
    model_grade = grade_by_model(test_case, output)  # 2. La evalúa con el calificador

    return {
        "output": output,                  # Respuesta original de Claude
        "test_case": test_case,            # Caso de prueba usado
        "score": model_grade["score"],     # Puntaje numérico (1-10)
        "reasoning": model_grade["reasoning"]  # Justificación del puntaje
    }
```

```python
from statistics import mean

def run_eval(dataset):
    results = []
    for test_case in dataset:
        result = run_test_case(test_case)  # Corre y califica cada caso
        results.append(result)

    # Calcula el promedio de todos los puntajes para tener una métrica global
    average_score = mean([result["score"] for result in results])
    print(f"Puntuación promedio: {average_score}")
    return results
```

`mean([result["score"] for result in results])` recorre la lista de resultados, extrae el campo `score` de cada uno y calcula el promedio. Ese número es la métrica de referencia: si al cambiar el prompt el promedio sube, la nueva versión es objetivamente mejor.

La puntuación promedio es la métrica de referencia que se usa para comparar versiones del prompt. Los evaluadores de modelos pueden ser algo variables, pero ofrecen una línea base consistente para medir mejoras.

---

#### 9.10 Calificación basada en código

Mientras el evaluador por modelo mide calidad semántica, el evaluador por código verifica dos aspectos técnicos objetivos:

| Criterio | Evaluador |
| --- | --- |
| **Formato** — solo Python, JSON o regex, sin explicación | Código |
| **Sintaxis válida** — el código parsea correctamente | Código |
| **Seguimiento de tarea** — responde lo que se pidió con precisión | Modelo |

---

#### Funciones de validación de sintaxis

Cada función intenta parsear el texto en su formato correspondiente. Si tiene éxito devuelve 10; si falla devuelve 0:

```python
import json, ast, re

def validate_json(text):
    try:
        json.loads(text.strip())   # Intenta convertir el string a un objeto JSON
        return 10                  # Sintaxis válida
    except json.JSONDecodeError:
        return 0                   # JSON malformado

def validate_python(text):
    try:
        ast.parse(text.strip())    # Intenta parsear el string como código Python
        return 10
    except SyntaxError:
        return 0                   # Error de sintaxis Python

def validate_regex(text):
    try:
        re.compile(text.strip())   # Intenta compilar el string como expresión regular
        return 10
    except re.error:
        return 0                   # Regex inválida
```

![alt text](imagenes/image-15.png)

---

#### Agregar el campo `format` al dataset

Para que el evaluador sepa qué validador usar, cada caso de prueba debe incluir el formato esperado:

```python
# Ejemplo de caso de prueba con formato especificado
{
    "task": "Create a Python function to validate an AWS IAM username",
    "format": "python"   # Indica qué validador aplicar: "python", "json" o "regex"
}
```

---

#### Mejorar el prompt para forzar formato limpio

Agregar instrucciones explícitas de formato reduce respuestas con texto explicativo innecesario:

```python
# Instrucciones de formato en el prompt
"""
* Respond only with Python, JSON, or a plain Regex
* Do not add any comments or commentary or explanation
"""

# Prefill genérico para code blocks (sin especificar el lenguaje)
add_assistant_message(messages, "```code")
# Claude interpreta esto como la apertura de un bloque de código y continúa solo con el código
```

---

#### Combinar puntaje de modelo y puntaje de código

```python
model_grade = grade_by_model(test_case, output)
model_score = model_grade["score"]       # Calidad semántica: ¿resuelve bien la tarea?
syntax_score = grade_syntax(output, test_case)  # Corrección técnica: ¿es sintaxis válida?

score = (model_score + syntax_score) / 2   # Promedio simple: ambos criterios tienen igual peso
```

El peso de cada criterio se puede ajustar según el caso de uso. Por ejemplo, si la sintaxis válida es crítica, podría dársele más peso que la calidad semántica.

> La puntuación de referencia no es buena ni mala en sí misma — lo que importa es si mejora al iterar sobre el prompt. El evaluador por código convierte esa medición en algo objetivo y reproducible.

---

## Unidad 4: Ingeniería de indicaciones

### Módulo 10: Ingeniería rápida — proceso iterativo

La ingeniería de indicaciones consiste en tomar un prompt existente y mejorarlo sistemáticamente para obtener resultados más fiables y de mayor calidad.

![alt text](imagenes/image-16.png)

---

#### 10.1 El ciclo de mejora iterativa

| Paso | Acción |
| --- | --- |
| 1 | Definir el objetivo |
| 2 | Escribir una indicación inicial básica |
| 3 | Evaluar con métricas objetivas |
| 4 | Aplicar técnicas de ingeniería |
| 5 | Reevaluar y comparar puntajes |
| 6 | Repetir hasta alcanzar el rendimiento deseado |

Cada iteración debería mostrar una mejora apreciable en el puntaje de evaluación.

![alt text](imagenes/image-17.png)

---

#### 10.2 Configurar el evaluador

```python
# PromptEvaluator gestiona la generación del dataset y la calificación
evaluator = PromptEvaluator(max_concurrent_tasks=5)
# max_concurrent_tasks controla cuántas tareas corren en paralelo
# Empezar con 3 para evitar errores de límite de velocidad de la API
```

---

#### 10.3 Generar el dataset de prueba automáticamente

```python
dataset = evaluator.generate_dataset(
    task_description="Write a compact, concise 1 day meal plan for a single athlete",
    prompt_inputs_spec={          # Define qué variables necesita el prompt
        "height": "Athlete's height in cm",
        "weight": "Athlete's weight in kg",
        "goal": "Goal of the athlete",
        "restrictions": "Dietary restrictions of the athlete"
    },
    output_file="dataset.json",   # Guarda el dataset en disco para reutilizarlo
    num_cases=3                   # Mantener bajo (2-3) durante desarrollo para iterar rápido
)
```

![alt text](imagenes/image-18.png)

---

#### 10.4 Escribir la indicación inicial (versión naive)

Empezar con algo deliberadamente básico para establecer una línea de base:

```python
def run_prompt(prompt_inputs):
    prompt = f"""
What should this person eat?

- Height: {prompt_inputs["height"]}      # Inserta la estatura del atleta
- Weight: {prompt_inputs["weight"]}      # Inserta el peso
- Goal: {prompt_inputs["goal"]}          # Inserta el objetivo
- Dietary restrictions: {prompt_inputs["restrictions"]}  # Inserta restricciones
"""
    messages = []
    add_user_message(messages, prompt)
    return chat(messages)   # Devuelve la respuesta de Claude sin procesamiento adicional
```

> Una puntuación inicial de ~2.3/10 es normal. El objetivo no es empezar bien, sino **medir la mejora** en cada iteración.

---

#### 10.5 Ejecutar la evaluación con criterios específicos

```python
results = evaluator.run_evaluation(
    run_prompt_function=run_prompt,    # Función que genera la respuesta para cada caso
    dataset_file="dataset.json",       # Dataset con los casos de prueba
    extra_criteria="""
The output should include:
- Daily caloric total
- Macronutrient breakdown
- Meals with exact foods, portions, and timing
"""
    # extra_criteria guía al calificador sobre qué aspectos evaluar específicamente
)
```

El resultado incluye un puntaje numérico y un informe HTML detallado con la justificación del calificador para cada caso de prueba.

---

#### 10.6 Principios clave

- Hacer **un cambio a la vez** para aislar el impacto de cada técnica
- Usar el informe detallado para entender exactamente **dónde falla** el prompt
- La ingeniería de indicaciones es efectiva solo cuando se combina con evaluación objetiva — sin métricas, no se puede saber si un cambio es una mejora real

---

### Módulo 11: Ser claro y directo

La primera línea del prompt es la parte más importante: sienta las bases para todo lo que sigue.

---

#### 11.1 Dos principios clave

| Principio | Qué significa |
| --- | --- |
| **Claridad** | Lenguaje sencillo, sin ambigüedad, que exprese exactamente lo que se quiere |
| **Concisión** | Instrucciones directas, sin rodeos ni contexto innecesario |

![alt text](imagenes/image-19.png)

---

#### 11.2 Claridad: decir exactamente lo que se quiere

Empezar con una declaración directa de la tarea, sin ambigüedades.

| En lugar de... | Usar... |
| --- | --- |
| "Necesito saber sobre esas cosas que ponen en los techos que usan el sol..." | "Escribe tres párrafos sobre cómo funcionan los paneles solares." |

---

#### 11.3 Directividad: instrucciones, no preguntas

Comenzar con **verbos de acción** como *Escribir*, *Crear*, *Generar*, *Identificar*.

| En lugar de... | Usar... |
| --- | --- |
| "He estado leyendo sobre energías renovables... ¿qué países usan geotérmica?" | "Identifica tres países que utilicen energía geotérmica. Incluye estadísticas de generación para cada uno." |

---

#### 11.4 Aplicado al ejemplo del plan de comidas

**Versión naive (base):**

```text
What should this person eat?
```

**Versión mejorada:**

```text
Generate a one-day meal plan for an athlete that meets their dietary restrictions.
```

Esta revisión le indica inmediatamente a Claude:

- **Qué acción** tomar → `Generate`
- **Qué crear** → un plan de comidas de un día
- **Para quién y con qué restricción** → un atleta, respetando su dieta

**Resultado:** el puntaje de evaluación pasó de **2.32 → 3.92** con un solo cambio en la primera línea.

> Claude responde mejor cuando se lo trata como un asistente competente que necesita instrucciones claras, no como alguien que tiene que adivinar lo que se quiere.

---

### Módulo 12: Ser específico

Especificar claramente lo que se desea reduce la ambigüedad y guía a Claude hacia resultados más consistentes y de mayor calidad.

![alt text](imagenes/image-20.png)

---

#### 12.1 Dos tipos de directrices

| Tipo | Para qué sirve |
| --- | --- |
| **Directrices de calidad de salida** | Controlar longitud, formato, atributos y tono del resultado |
| **Pasos del proceso** | Guiar a Claude para que analice sistemáticamente antes de responder |

Ambos tipos se usan frecuentemente en conjunto en prompts profesionales.

![alt text](imagenes/image-21.png)

---

#### 12.2 Directrices de calidad de salida

Enumeran las cualidades que debe tener el resultado final. Ejemplos para un plan de comidas:

```text
Guidelines:
1. Include accurate daily calorie amount
2. Show protein, fat, and carb amounts
3. Specify when to eat each meal
4. Use only foods that fit restrictions
5. List all portion sizes in grams
6. Keep budget-friendly if mentioned
```

**Resultado en el ejemplo:** el puntaje de evaluación pasó de **3.92 → 7.86** — más del doble, solo por agregar directrices específicas.

---

#### 12.3 Pasos del proceso

Instrucciones que Claude debe seguir antes de generar la respuesta final. Útiles cuando se quiere análisis sistemático o consideración de múltiples perspectivas.

Ejemplo para una historia:

```text
1. Think of three talents that would create dramatic tension
2. Choose the most interesting talent
3. Describe a key scene that reveals the talent
4. Think of supporting characters that could increase the impact
```

---

#### 12.4 Cuándo usar cada tipo

| Situación | Enfoque recomendado |
| --- | --- |
| Cualquier prompt | Siempre incluir directrices de calidad de salida |
| Resolución de problemas complejos | Agregar pasos del proceso |
| Toma de decisiones o pensamiento crítico | Agregar pasos del proceso |
| Tareas donde Claude debe considerar múltiples perspectivas | Agregar pasos del proceso |

![alt text](imagenes/image-22.png)

> La especificidad no limita a Claude — le da un objetivo claro al que aspirar, lo que mejora tanto la coherencia como la calidad del resultado.

---

### Módulo 13: Estructura con etiquetas XML

Las etiquetas XML permiten agregar estructura y claridad a los prompts, especialmente cuando se interpolan grandes cantidades de datos o se mezclan diferentes tipos de contenido.

![alt text](imagenes/image-23.png)

---

#### 13.1 Por qué la estructura importa

Sin delimitadores claros, Claude puede tener dificultades para distinguir entre las instrucciones y los datos que debe analizar. Las etiquetas XML crean límites explícitos entre secciones.

| Sin estructura | Con etiquetas XML |
| --- | --- |
| Instrucciones y datos mezclados | Cada sección claramente delimitada |
| Claude puede malinterpretar el contexto | Claude entiende el propósito de cada bloque |

![alt text](imagenes/image-24.png)

---

#### 13.2 Nombres de etiquetas personalizados

No se necesitan etiquetas XML oficiales — se crean nombres descriptivos según el contenido:

| Etiqueta genérica | Etiqueta descriptiva (preferida) |
| --- | --- |
| `<data>` | `<sales_records>` |
| `<info>` | `<athlete_information>` |
| `<text>` | `<docs>` |

Cuanto más descriptivo el nombre, mejor entiende Claude el propósito de cada sección.

---

#### 13.3 Ejemplo aplicado al plan de comidas

```xml
<athlete_information>
- Height: 6'2"
- Weight: 180 lbs
- Goal: Build muscle
- Dietary restrictions: Vegetarian
</athlete_information>

Generate a meal plan based on the athlete information above.
```

Las etiquetas agrupan todos los datos del atleta como una unidad coherente, dejando en claro qué es contexto y qué es la instrucción.

---

#### 13.4 Cuándo usar etiquetas XML

- Al incluir grandes cantidades de contexto o datos
- Al mezclar diferentes tipos de contenido (código, documentación, datos)
- Cuando los límites del contenido deben ser inequívocos
- En prompts complejos que interpolan múltiples variables

> Las etiquetas XML se vuelven cada vez más valiosas a medida que los prompts se vuelven más complejos. Con prompts simples el impacto es menor, pero en prompts avanzados pueden ser la diferencia entre una respuesta correcta y una confusa.

---

### Módulo 14: Proporcionar ejemplos (few-shot prompting)

Proveer ejemplos de entrada/salida es una de las técnicas más efectivas de ingeniería de indicaciones. Se conoce como **one-shot** (un ejemplo) o **few-shot** (varios ejemplos).

---

#### 14.1 Cómo funcionan los ejemplos

En lugar de describir lo que se quiere con palabras, se **demuestra** directamente con pares de entrada y salida ideal. Esto es especialmente útil para casos ambiguos o difíciles de describir verbalmente.

**Ejemplo:** análisis de sentimiento con sarcasmo

Sin ejemplos, Claude puede interpretar "¡Realmente necesitaba un retraso en el vuelo esta noche! ¡Excelente!" como positivo. Con ejemplos, aprende que el sarcasmo debe clasificarse como negativo.

![alt text](imagenes/image-25.png)

---

#### 14.2 Estructura con etiquetas XML

Los ejemplos deben envolverse en etiquetas XML para que Claude entienda claramente qué es entrada y qué es salida ideal:

```xml
<sample_input>
Oh sure, I really needed a flight delay tonight! Just excellent!
</sample_input>
<ideal_output>
Negative
</ideal_output>
```

---

#### 14.3 Agregar contexto al ejemplo

No basta con dar el par entrada/salida — explicar **por qué** la salida es buena ayuda a Claude a entender el razonamiento, no solo el formato:

```xml
<ideal_output>
[Tu ejemplo de salida aquí]
</ideal_output>

This example is well-structured, provides detailed information
on food choices and quantities, and aligns with the athlete's
goals and restrictions.
```

![alt text](imagenes/image-26.png)

---

#### 14.4 One-shot vs. few-shot

| Tipo | Cuándo usarlo |
| --- | --- |
| **One-shot** (1 ejemplo) | Para establecer el patrón general |
| **Few-shot** (varios ejemplos) | Para cubrir distintos casos borde o mostrar variedad de respuestas válidas |

---

#### 14.5 Cómo encontrar buenos ejemplos

Al correr evaluaciones, buscar los resultados con el puntaje más alto (idealmente 10/10) y usar esos pares entrada/salida como ejemplos en el prompt. Así Claude aprende a partir de sus propias mejores respuestas.

---

#### 14.6 Buenas prácticas

- Usar siempre etiquetas XML para estructurar los ejemplos
- Ser explícito: "Aquí hay un ejemplo de entrada con una respuesta ideal"
- Incluir ejemplos que cubran los casos de fallo más comunes
- Explicar por qué cada ejemplo de salida es considerado ideal
- Usar ejemplos relevantes y específicos para la tarea

> Los ejemplos muestran en lugar de describir. Esto hace los prompts mucho más fiables y ayuda a Claude a entender requisitos sutiles que serían difíciles de expresar solo con instrucciones.

---

## Unidad 5: Tools

### Módulo 15: Introducción al uso de herramientas

Las herramientas (tools) permiten a Claude acceder a información del mundo exterior, superando la limitación de solo conocer lo aprendido durante el entrenamiento.

---

#### 15.1 El problema sin herramientas

Por defecto, Claude no puede acceder a eventos actuales, datos en tiempo real ni sistemas externos. Ante una pregunta como "¿Qué tiempo hace en San Francisco?", solo puede responder que no tiene acceso a esa información — una experiencia frustrante para el usuario.

---

#### 15.2 Cómo funciona el uso de herramientas

El flujo es un ciclo de ida y vuelta entre la aplicación y Claude:

| Paso | Quién actúa | Qué ocurre |
| --- | --- | --- |
| 1. Solicitud inicial | Aplicación → Claude | Se envía la pregunta del usuario junto con instrucciones de qué herramientas están disponibles |
| 2. Solicitud de herramienta | Claude → Aplicación | Claude decide que necesita datos externos y solicita información específica |
| 3. Recuperación de datos | Aplicación → API externa | El servidor ejecuta código para obtener los datos solicitados |
| 4. Respuesta final | Aplicación → Claude → Usuario | Se envían los datos a Claude, quien genera la respuesta completa |


![alt text](image.png)
---

#### 15.3 Ejemplo: consulta del clima

1. El usuario pregunta por el clima actual
2. Se incluyen en el mensaje instrucciones sobre cómo obtener datos meteorológicos
3. Claude reconoce que necesita información actualizada y la solicita para la ubicación específica
4. El servidor consulta una API meteorológica y devuelve los datos a Claude
5. Claude combina los datos en tiempo real con la pregunta para dar una respuesta precisa

---

#### 15.4 Beneficios clave

| Beneficio | Descripción |
| --- | --- |
| **Información en tiempo real** | Datos actuales no disponibles en el entrenamiento |
| **Integración de sistemas externos** | Bases de datos, APIs y otros servicios |
| **Respuestas dinámicas** | Basadas en la información más reciente disponible |
| **Interacción estructurada** | Claude sabe exactamente qué información necesita y cómo pedirla |

> Las herramientas transforman a Claude de una base de conocimientos estática en un asistente dinámico capaz de trabajar con datos en tiempo real: clima, cotizaciones bursátiles, consultas a bases de datos, y cualquier otra información que los usuarios necesiten al momento.

---

### Módulo 16: Proyecto práctico — sistema de recordatorios

#### 16.1 Objetivo

Permitir que Claude interprete solicitudes en lenguaje natural como:

> "Pon un recordatorio para mi cita con el médico, es el jueves de la semana que viene"

Y responda con: "De acuerdo, te lo recordaré", habiendo calculado y configurado el recordatorio correctamente.

---

#### 16.2 Por qué es un desafío

Claude tiene tres limitaciones específicas para este caso de uso:

| Limitación | Descripción |
| --- | --- |
| **Conciencia temporal limitada** | Puede conocer la fecha actual pero no la hora exacta |
| **Cálculo de fechas impreciso** | No siempre maneja bien la suma de fechas con muchos días de anticipación |
| **Sin mecanismo de recordatorio** | No tiene forma incorporada de configurar recordatorios |

---

#### 16.3 Las tres herramientas a construir

Cada herramienta resuelve una limitación específica:

| Herramienta | Problema que resuelve |
| --- | --- |
| `get_current_datetime` | Provee la fecha y hora exacta actual |
| `add_duration_to_datetime` | Calcula fechas futuras de forma confiable |
| `set_reminder` | Configura el recordatorio en el sistema |

Se implementan una a una, de la más simple a la más compleja, para entender el mecanismo de llamada a herramientas progresivamente.

![alt text](image-1.png)

---

#### 16.4 Principio clave

> Cuando el modelo tiene limitaciones, se amplían sus capacidades mediante herramientas en lugar de intentar sortear esas limitaciones en el prompt.

---

### Módulo 17: Funciones de herramienta

Una **función de herramienta** es una función Python normal que se ejecuta automáticamente cuando Claude necesita información adicional para responder al usuario.

![alt text](image-2.png)
---

#### 17.1 Buenas prácticas al escribir funciones de herramienta

| Práctica | Por qué importa |
| --- | --- |
| **Nombres descriptivos** | Tanto el nombre de la función como el de los parámetros deben indicar claramente su propósito |
| **Validar entradas** | Verificar que los parámetros obligatorios no estén vacíos o sean inválidos |
| **Mensajes de error claros** | Claude puede leer los errores y reintentar la llamada con parámetros corregidos |

La validación es especialmente importante porque Claude aprende de los errores: un mensaje como `"La ubicación no puede estar vacía"` le permite corregir la llamada automáticamente.

![alt text](image-3.png)
---

#### 17.2 Primera herramienta: obtener fecha y hora actual

```python
from datetime import datetime

def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")   # Error claro que Claude puede interpretar
    return datetime.now().strftime(date_format)   # Formatea la fecha/hora según el patrón recibido
```

El formato por defecto devuelve `"2024-01-15 14:30:25"` (año-mes-día hora:minuto:segundo).

Ejemplos de uso:

```python
get_current_datetime()          # "2024-01-15 14:30:25"  — formato completo por defecto
get_current_datetime("%H:%M")   # "14:30"                — solo hora y minuto
```

---

#### 17.3 Próximos pasos

Crear la función es solo el primer paso. Luego se necesita:

1. Escribir un **esquema JSON** que describa la función a Claude
2. **Integrarla** en el sistema de chat para que Claude pueda invocarla

Este enfoque mantiene el código organizado y le da a Claude capacidades poderosas sin complejidad innecesaria.

---

### Módulo 18: Esquemas de herramientas

Tras escribir la función, el siguiente paso es crear un **esquema JSON** que le indique a Claude qué argumentos espera la función y cómo usarla. Actúa como documentación que Claude consulta para saber cuándo y cómo llamar a cada herramienta.

![alt text](imagenes/image-27.png)
---

#### 18.1 Las tres partes de una especificación de herramienta

| Campo | Descripción |
| --- | --- |
| `name` | Nombre claro y descriptivo de la herramienta |
| `description` | Qué hace, cuándo usarla y qué devuelve |
| `input_schema` | Esquema JSON que describe los argumentos de la función |

---

#### 18.2 Cómo escribir descripciones efectivas

La descripción es clave para que Claude sepa cuándo invocar la herramienta:

- Explicar qué hace la herramienta en 3-4 frases
- Indicar cuándo debería usarla Claude
- Describir qué tipo de datos devuelve
- Dar descripciones detalladas para cada argumento

![alt text](imagenes/image-28.png)
---

#### 18.3 Generar el esquema con Claude

En lugar de escribirlo desde cero, se puede pedirle a Claude que lo genere:

1. Copiar el código de la función
2. Pedirle a Claude: *"Write a valid JSON schema specification for tool calling for this function. Follow the best practices listed in the attached documentation."*
3. Incluir la documentación de Anthropic sobre tool use como contexto
4. Copiar el esquema generado al archivo de código

![alt text](imagenes/image-29.png)
---

#### 18.4 Implementación del esquema

Convención de nombres: usar `function_name` seguido de `function_name_schema` para mantener la relación clara entre función y esquema.

```python
def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")
    return datetime.now().strftime(date_format)

# El esquema describe la función para que Claude sepa cómo invocarla
get_current_datetime_schema = {
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format",
    "input_schema": {
        "type": "object",
        "properties": {
            "date_format": {
                "type": "string",                              # Tipo del parámetro
                "description": "A string specifying the format of the returned datetime. Uses Python's strftime format codes.",
                "default": "%Y-%m-%d %H:%M:%S"               # Valor por defecto si no se especifica
            }
        },
        "required": []   # Lista vacía: ningún parámetro es obligatorio (todos tienen valor por defecto)
    }
}
```

---

#### 18.5 Agregar seguridad de tipos con `ToolParam`

```python
from anthropic.types import ToolParam   # Tipo de la librería de Anthropic

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format",
    # ... resto del esquema
})
# ToolParam no cambia el comportamiento, pero evita errores de tipo al pasar
# el esquema a la API y hace el código más robusto
```

---

### Módulo 19: Manejo de bloques de mensajes

Cuando Claude usa herramientas, la estructura de respuesta cambia: en lugar de un bloque de texto simple, devuelve mensajes **multibloque** que contienen tanto texto como información de uso de herramientas.

---

#### 19.1 Llamada a la API con herramientas

```python
messages = []
messages.append({
    "role": "user",
    "content": "What is the exact time, formatted as HH:MM:SS?"
})

response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],   # Lista de esquemas de herramientas disponibles para Claude
)
```

---

#### 19.2 Estructura de un mensaje multibloque

Cuando Claude decide usar una herramienta, `response.content` contiene una lista con dos bloques:

| Bloque | Tipo | Contenido |
| --- | --- | --- |
| **TextBlock** | `text` | Texto legible explicando lo que Claude está haciendo |
| **ToolUseBlock** | `tool_use` | Instrucciones para el código: qué función llamar y con qué parámetros |

El **ToolUseBlock** incluye:

- Un `id` para rastrear la llamada a la herramienta
- El `name` de la función a invocar (ej. `"get_current_datetime"`)
- Los `input` formateados como diccionario con los parámetros

![alt text](imagenes/image-30.png)
---

#### 19.3 Gestionar el historial con mensajes multibloque

Al agregar la respuesta de Claude al historial, hay que conservar **toda** la estructura del contenido, no solo el texto:

```python
messages.append({
    "role": "assistant",
    "content": response.content   # Preserva tanto el TextBlock como el ToolUseBlock
})
# Si se guardara solo response.content[0].text se perdería el ToolUseBlock
# y Claude perdería el contexto de qué herramienta llamó
```

---

#### 19.4 Flujo completo de uso de herramienta

```text
1. Enviar mensaje de usuario + esquemas de herramientas disponibles → Claude
2. Claude responde con TextBlock + ToolUseBlock
3. El código extrae la info del ToolUseBlock y ejecuta la función
4. Se envía el resultado de la herramienta + historial completo → Claude
5. Claude genera la respuesta final para el usuario
```

Cada paso requiere conservar la estructura completa del mensaje para que Claude mantenga el contexto necesario.

![alt text](imagenes/image-31.png)
---

#### 19.5 Actualizar las funciones auxiliares

Las funciones `add_user_message()` y `add_assistant_message()` originales solo soportan texto. Deben actualizarse para admitir contenido multibloque:

```python
def add_assistant_message(messages, content):
    messages.append({
        "role": "assistant",
        "content": content   # Acepta tanto string (texto) como lista de bloques (multibloque)
    })
```

---

#### 19.6 Apéndice: flujo completo con ejemplos

##### El problema de fondo

Claude es un modelo de lenguaje — no puede ejecutar código ni acceder al mundo real. Si le preguntás "¿qué hora es?", no lo sabe. Los tools son la solución: Claude puede *pedir* que alguien ejecute una función por él, y luego usar el resultado para responder.

---

##### Paso 1 — Definir la función y su schema

Toda herramienta tiene dos partes: la función Python que hace el trabajo, y el schema JSON que le explica a Claude cómo y cuándo llamarla.

```python
from datetime import datetime

def get_current_datetime():
    return datetime.now().strftime("%H:%M:%S")

get_current_datetime_schema = {
    "name": "get_current_datetime",          # Nombre exacto de la función
    "description": "Returns the current time in HH:MM:SS format.",  # Claude decide si usarla basándose en esto
    "input_schema": {
        "type": "object",
        "properties": {}                     # Vacío pero obligatorio — la función no necesita argumentos
    }
}
```

> Si `input_schema` se omite o está malformado, la API devuelve un `BadRequestError 400`. Siempre incluir `"type": "object"` y `"properties"`, aunque estén vacíos.

---

##### Paso 2 — Enviar la pregunta con el menú de tools

Se manda el mensaje del usuario y se adjuntan los schemas. Claude los usa como un menú: decide si invocar alguno o responder directamente.

```python
messages = [{"role": "user", "content": "What is the exact time, formatted as HH:MM:SS?"}]

response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],   # Claude sabe que tiene esta herramienta disponible
)
```

---

##### Paso 3 — Claude responde con dos bloques

`response.content` no es texto simple — es una lista de bloques:

| Bloque | Tipo | Contenido |
| --- | --- | --- |
| `response.content[0]` | `TextBlock` | `"I'll check the current time for you."` |
| `response.content[1]` | `ToolUseBlock` | `{id: "toolu_01...", name: "get_current_datetime", input: {}}` |

Claude se detiene aquí (`stop_reason == "tool_use"`). No puede continuar solo — necesita que el código ejecute la función.

---

##### Paso 4 — Ejecutar la función y construir el historial

Este es el paso más crítico. Hay que:

1. Agregar la respuesta del asistente **con ambos bloques** (no solo el texto)
2. Ejecutar la función real
3. Enviar el resultado como mensaje de usuario con tipo `tool_result`

```python
# 1. Guardar la respuesta completa del asistente — INCLUYE el ToolUseBlock con su id
messages.append({
    "role": "assistant",
    "content": response.content        # ← Nunca usar solo response.content[0].text aquí
})

# 2. Ejecutar la función real
tool_use = next(b for b in response.content if b.type == "tool_use")
result = get_current_datetime()        # "14:32:07"

# 3. Devolver el resultado — el tool_use_id conecta este resultado con la solicitud anterior
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": tool_use.id,    # ← Debe coincidir con el id del ToolUseBlock
        "content": result              # "14:32:07"
    }]
})
```

> Si en el paso anterior se guardara solo el texto y se omitiera el `ToolUseBlock`, la API rechazaría la solicitud porque el `tool_result` hace referencia a un `id` que no existe en el historial.

---

##### Paso 5 — Claude genera la respuesta final

Ahora Claude tiene todo el contexto: la pregunta original, su propio razonamiento previo, y el resultado de la función. Puede responder al usuario de forma natural.

```python
final = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],   # Siempre incluir los schemas en llamadas de seguimiento
)
print(final.content[0].text)
# → "The exact time is 14:32:07."
```

---

##### Resumen visual del flujo

```text
Vos      →  Claude     Pregunta + schemas disponibles
Claude   →  Vos        [TextBlock] + [ToolUseBlock: llamar a get_current_datetime]
Vos ejecuta  →         "14:32:07"
Vos      →  Claude     historial completo + tool_result: "14:32:07"
Claude   →  Vos        "The exact time is 14:32:07."
```

---

##### Por qué el historial completo es obligatorio

Claude no tiene memoria entre llamadas. Cada `client.messages.create()` empieza desde cero. El historial que se envía en el paso 5 debe contener:

| Posición | Rol | Contenido |
| --- | --- | --- |
| 1 | `user` | Pregunta original |
| 2 | `assistant` | TextBlock + ToolUseBlock |
| 3 | `user` | Bloque `tool_result` con el resultado |

Sin cualquiera de esos tres elementos, Claude no tiene el contexto suficiente para generar la respuesta final.

---

### Módulo 20: Enviando resultados de la herramienta

Después de que Claude solicita una llamada a herramienta, el código debe ejecutar la función y devolver el resultado para completar el flujo.

---

#### 20.1 Ejecutar la función de la herramienta

```python
# Acceder a los parámetros que Claude quiere pasar a la función
response.content[1].input   # Devuelve un diccionario con los argumentos

# ** desempaqueta el diccionario como argumentos nombrados de la función
get_current_datetime(**response.content[1].input)
```

![alt text](imagenes/image-32.png)

---

#### 20.2 Estructura del bloque de resultado (`tool_result`)

El bloque de resultado tiene tres propiedades clave:

| Propiedad | Descripción |
| --- | --- |
| `tool_use_id` | Debe coincidir con el ID del ToolUseBlock al que corresponde este resultado |
| `content` | Salida de la ejecución de la herramienta, serializada como string |
| `is_error` | `True` si ocurrió un error durante la ejecución; `False` si fue exitosa |

El resultado se envía dentro de un mensaje de usuario con este formato:

```python
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": response.content[1].id,   # Debe coincidir con el ID del ToolUseBlock
        "content": "15:04:22",                    # Resultado serializado como string
        "is_error": False                          # True si ocurrió un error durante la ejecución
    }]
})
```
![alt text](imagenes/image-33.png)
---

#### 20.3 Manejo de múltiples llamadas a herramientas

Claude puede solicitar varias herramientas en una sola respuesta (ej. dos sumas independientes). Cada llamada tiene un `id` único — los resultados deben enviarse con el `tool_use_id` correcto para que Claude sepa qué resultado corresponde a cada solicitud, incluso si llegan en distinto orden.

![alt text](imagenes/image-34.png)

![alt text](imagenes/image-35.png)

---

#### 20.4 El historial completo en la solicitud de seguimiento

En el momento de enviar el resultado, el historial contiene:

```text
1. Mensaje original del usuario
2. Mensaje del asistente (TextBlock + ToolUseBlock)
3. Mensaje del usuario con el bloque tool_result
```

---

#### 20.5 Solicitud final a Claude

Al enviar la solicitud de seguimiento hay que incluir el esquema de la herramienta aunque no se espere otra llamada — Claude lo necesita para entender las referencias a herramientas en el historial:

```python
final_response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,              # Historial completo con el tool_result incluido
    tools=[get_current_datetime_schema]   # Siempre incluir el esquema en solicitudes de seguimiento
)
```
![alt text](imagenes/image-36.png)

Claude responde con un mensaje final en lenguaje natural que incorpora el resultado de la herramienta.

---

### Módulo 21: Conversaciones de varios turnos con herramientas

Cuando una pregunta requiere múltiples herramientas en secuencia (ej. "¿Qué día será dentro de 103 días?"), Claude primero obtiene la fecha actual y luego suma los días. La aplicación debe gestionar este patrón automáticamente.

![alt text](imagenes/image-37.png)
---

#### 21.1 Patrón de múltiples turnos

```text
1. Usuario pregunta: "¿Qué día será dentro de 103 días?"
2. Claude solicita → get_current_datetime
3. Servidor ejecuta la función y devuelve el resultado
4. Claude solicita → add_duration_to_datetime
5. Servidor ejecuta la función y devuelve el resultado
6. Claude tiene suficiente información → genera respuesta final
```
![alt text](imagenes/image-38.png)
---

#### 21.2 Bucle de conversación

```python
def run_conversation(messages):
    while True:
        response = chat(messages)              # Llama a Claude con el historial actual

        add_assistant_message(messages, response)   # Agrega la respuesta al historial

        if response isn't asking for a tool:   # Si Claude no pide más herramientas, terminar
            break

        tool_result_blocks = run_tools(response)         # Ejecuta las herramientas solicitadas
        add_user_message(messages, tool_result_blocks)   # Agrega los resultados al historial

    return messages   # Devuelve el historial completo con toda la conversación
```

El bucle continúa hasta que `stop_reason` sea `"end_turn"` en lugar de `"tool_use"`.

![alt text](imagenes/image-39.png)
---

#### 21.3 Actualizar `add_user_message` para contenido flexible

```python
from anthropic.types import Message

def add_user_message(messages, message):
    user_message = {
        "role": "user",
        # Si se recibe un objeto Message completo, extrae su contenido; si no, usa el valor tal cual
        "content": message.content if isinstance(message, Message) else message
    }
    messages.append(user_message)
```

| Tipo de entrada | Resultado |
| --- | --- |
| `str` | Se usa directamente como contenido |
| `list` de bloques | Se usa directamente como contenido |
| Objeto `Message` | Se extrae `message.content` |

---

#### 21.4 Actualizar `chat` para soportar herramientas

```python
def chat(messages, system=None, temperature=1.0, stop_sequences=[], tools=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
        "stop_sequences": stop_sequences,
    }

    if tools:
        params["tools"] = tools       # Solo se agrega si se proporcionaron herramientas

    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message   # Devuelve el objeto Message completo, no solo el texto
```

> Cambio clave respecto a versiones anteriores: ahora `chat` devuelve el objeto `Message` completo en lugar de solo `message.content[0].text`, para preservar todos los bloques (TextBlock + ToolUseBlock).

---

#### 21.5 Función auxiliar para extraer texto

```python
def text_from_message(message):
    # Filtra solo los bloques de tipo "text" e ignora los ToolUseBlocks
    return "\n".join(
        [block.text for block in message.content if block.type == "text"]
    )
```

| Propiedad del bloque | Descripción |
| --- | --- |
| `block.type` | Tipo del bloque: `"text"` o `"tool_use"` |
| `block.text` | Texto legible (solo disponible en bloques de tipo `"text"`) |

---

#### 21.6 Mejoras clave de esta refactorización

| Mejora | Descripción |
| --- | --- |
| **Gestión flexible de mensajes** | Las funciones auxiliares aceptan texto, listas de bloques u objetos `Message` |
| **Soporte de herramientas en `chat`** | La función puede recibir y transmitir esquemas de herramientas a la API |
| **Objeto `Message` completo devuelto** | Se preservan todos los bloques (TextBlock + ToolUseBlock), no solo el texto |
| **Utilidad de extracción de texto** | `text_from_message()` permite obtener texto legible desde mensajes complejos |

---

### Módulo 22: Implementar múltiples giros

Para crear un sistema de conversación con herramientas funcional, es necesario implementar un bucle que llame a Claude repetidamente hasta que deje de solicitar herramientas. El módulo cubre los componentes clave: detección de solicitudes, ejecución múltiple, manejo de errores y enrutamiento escalable.

---

#### 22.1 Detección de solicitudes de herramientas

La clave para saber si Claude quiere usar una herramienta está en el campo `stop_reason` del mensaje de respuesta:

| Valor de `stop_reason` | Significado |
| --- | --- |
| `"tool_use"` | Claude quiere llamar a una o más herramientas |
| `"end_turn"` | Claude tiene una respuesta final, no necesita más herramientas |

```python
if response.stop_reason != "tool_use":
    break   # Claude terminó, no hay más herramientas que ejecutar
```

---

#### 22.2 El ciclo de conversación

`run_conversation` encapsula el bucle completo: llama a Claude, procesa herramientas, repite hasta tener respuesta final:

```python
def run_conversation(messages):
    while True:
        # Llama a Claude con el historial actual y las herramientas disponibles
        response = chat(messages, tools=[get_current_datetime_schema])
        add_assistant_message(messages, response)   # Agrega la respuesta al historial

        print(text_from_message(response))          # Muestra el texto legible de la respuesta

        if response.stop_reason != "tool_use":      # Si Claude no pide más herramientas, salir del bucle
            break

        tool_results = run_tools(response)          # Ejecuta todas las herramientas solicitadas
        add_user_message(messages, tool_results)    # Devuelve los resultados a Claude

    return messages   # Historial completo con toda la conversación
```

---

#### 22.3 La función `run_tools`

![alt text](imagenes/image-40.png)

Claude puede pedir varias herramientas en una sola respuesta. `run_tools` filtra todos los bloques de tipo `tool_use` y los procesa uno a uno:

```python
def run_tools(message):
    # Filtra solo los bloques que son solicitudes de herramienta
    tool_requests = [
        block for block in message.content if block.type == "tool_use"
    ]
    tool_result_blocks = []

    for tool_request in tool_requests:
        # Procesa cada solicitud individualmente (ver sección 22.4 y 22.5)
        ...

    return tool_result_blocks   # Lista de bloques tool_result para enviar a Claude
```

---

#### 22.4 Estructura del bloque de resultado (`tool_result`)

Cada `ToolUseBlock` debe responderse con un bloque de resultado que tenga el mismo `id`. Así Claude sabe qué resultado corresponde a qué solicitud:

![alt text](imagenes/image-41.png)

```python
tool_result_block = {
    "type": "tool_result",
    "tool_use_id": tool_request.id,          # Debe coincidir con el id del ToolUseBlock
    "content": json.dumps(tool_output),       # Resultado serializado como string JSON
    "is_error": False                          # False si la ejecución fue exitosa
}
```

| Propiedad | Tipo | Descripción |
| --- | --- | --- |
| `type` | `str` | Siempre `"tool_result"` |
| `tool_use_id` | `str` | ID del `ToolUseBlock` al que responde este resultado |
| `content` | `str` | Resultado de la herramienta serializado como string |
| `is_error` | `bool` | `True` si ocurrió un error; `False` si fue exitosa |

---

#### 22.5 Manejo de errores en la ejecución

Cuando una herramienta falla, igualmente hay que enviar un bloque de resultado a Claude (con `is_error: True`) para que pueda decidir cómo continuar:

```python
try:
    tool_output = run_tool(tool_request.name, tool_request.input)
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": json.dumps(tool_output),   # Resultado exitoso serializado
        "is_error": False
    }
except Exception as e:
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": f"Error: {e}",   # Mensaje de error legible para que Claude lo interprete
        "is_error": True            # Señal explícita de fallo
    }
```

> Siempre hay que devolver un bloque de resultado aunque la herramienta falle — omitirlo rompe el flujo de conversación.

---

#### 22.6 Enrutamiento de herramientas escalable

Para soportar varias herramientas, `run_tool` mapea nombres a implementaciones. Agregar una nueva herramienta solo requiere un `elif` adicional, sin tocar el resto de la lógica:

```python
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)    # ** desempaqueta el dict como argumentos
    elif tool_name == "add_duration_to_datetime":
        return add_duration_to_datetime(**tool_input)
    elif tool_name == "another_tool":
        return another_tool(**tool_input)
    # Agregar más herramientas aquí sin cambiar run_conversation ni run_tools
```

---

#### 22.7 Flujo de trabajo completo

| Paso | Quién actúa | Qué ocurre |
| --- | --- | --- |
| 1 | Aplicación → Claude | Se envía el mensaje del usuario con la lista de herramientas disponibles |
| 2 | Claude → Aplicación | Claude responde con texto y/o solicitudes de herramientas (`ToolUseBlock`) |
| 3 | Aplicación | `run_tools` ejecuta cada herramienta y construye los bloques de resultado |
| 4 | Aplicación → Claude | Se envían los resultados como mensaje de usuario |
| 5 | — | Se repite desde el paso 1 hasta que `stop_reason != "tool_use"` |

El historial completo se conserva en cada iteración, lo que permite a Claude basarse en los resultados de herramientas anteriores para dar respuestas exhaustivas a solicitudes complejas.

---

### Módulo 23: Utilizar múltiples herramientas

Una vez que la infraestructura básica de manejo de herramientas está en su lugar, agregar herramientas adicionales es directo: se crea la función, se define su esquema, se registra en la lista y se agrega un caso al enrutador.

---

#### 23.1 Las herramientas que estamos añadiendo

Para el sistema de recordatorios se necesitan tres herramientas complementarias:

| Herramienta | Por qué es necesaria |
| --- | --- |
| `get_current_datetime` | Claude necesita conocer la fecha y hora actuales para calcular fechas futuras |
| `add_duration_to_datetime` | Claude no es confiable para sumar fechas manualmente; la herramienta lo hace con precisión |
| `set_reminder` | Mecanismo para configurar el recordatorio en el sistema |

![alt text](imagenes/image-42.png)
---

#### 23.2 Añadiendo herramientas a la conversación

En `run_conversation`, se pasan todos los esquemas en la lista `tools` para que Claude sepa cuáles tiene disponibles:

```python
response = chat(messages, tools=[
    get_current_datetime_schema,       # Herramienta 1: fecha y hora actual
    add_duration_to_datetime_schema,   # Herramienta 2: suma de duración a una fecha
    set_reminder_schema                # Herramienta 3: configurar el recordatorio
])
```

---

#### 23.3 Actualización del enrutador de herramientas

Se agregan casos `elif` en `run_tool` por cada nueva herramienta. La función actúa como un despachador: recibe el nombre, llama a la función correcta con los argumentos de Claude y devuelve el resultado:

```python
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)        # Desempaqueta los argumentos del dict

    elif tool_name == "add_duration_to_datetime":
        return add_duration_to_datetime(**tool_input)    # Claude pasa fecha base + duración

    elif tool_name == "set_reminder":
        return set_reminder(**tool_input)                # Claude pasa fecha/hora y texto del recordatorio
```

---

#### 23.4 Pruebas de uso de múltiples herramientas

Para verificar que el flujo funciona, se puede usar una solicitud que obligue a Claude a encadenar herramientas:

> *"Configurar un recordatorio para mi cita con el médico. Faltan 177 días para el 1 de enero de 2050."*

Claude resuelve esta solicitud en secuencia:

| Paso | Herramienta usada | Qué hace |
| --- | --- | --- |
| 1 | `add_duration_to_datetime` | Calcula que 177 días antes del 1/1/2050 es el 27/6/2050 |
| 2 | `set_reminder` | Configura el recordatorio para esa fecha calculada |

![alt text](imagenes/image-43.png)

---

#### 23.5 Comprender el flujo de mensajes

Al revisar el historial completo de la conversación se puede ver la estructura de mensajes encadenados:

| Turno | Rol | Contenido |
| --- | --- | --- |
| 1 | `user` | Solicitud original del usuario |
| 2 | `assistant` | TextBlock explicativo + uno o más ToolUseBlocks |
| 3 | `user` | Bloques `tool_result` con los resultados de las herramientas |
| 4 | `assistant` | Mensaje de seguimiento (puede pedir más herramientas o dar respuesta final) |

> Un solo mensaje del asistente puede contener varios bloques simultáneamente: texto explicativo y múltiples solicitudes de herramientas combinadas.

---

#### 23.6 El patrón simple para agregar herramientas

Cada nueva herramienta sigue exactamente el mismo proceso de cuatro pasos:

| Paso | Acción |
| --- | --- |
| 1 | Crear la implementación de la función Python |
| 2 | Definir el esquema JSON de la herramienta |
| 3 | Agregar el esquema a la lista `tools` en `run_conversation` |
| 4 | Agregar un caso `elif` en `run_tool` |

Este enfoque modular permite ampliar las capacidades del asistente sin reestructurar el código existente. Cada herramienta nueva se integra al flujo de conversación y a la lógica de gestión sin modificar lo ya construido.

![alt text](imagenes/image-44.png)

---

### Módulo 24: Llamada de herramienta de grano fino

Al combinar herramientas con streaming, Claude devuelve actualizaciones en tiempo real a medida que genera los argumentos de cada herramienta. Comprender el mecanismo interno de validación es clave para elegir el comportamiento correcto.

---

#### 24.1 Transmisión de herramientas básicas

Con streaming habilitado, además de los eventos `ContentBlockDelta` del texto normal, aparece un nuevo tipo de evento para herramientas: `InputJsonEvent`.

| Propiedad | Descripción |
| --- | --- |
| `partial_json` | Fragmento de JSON que representa parte de los argumentos de la herramienta |
| `snapshot` | JSON acumulativo construido a partir de todos los fragmentos recibidos hasta el momento |

```python
for chunk in stream:
    if chunk.type == "input_json":
        print(chunk.partial_json)      # Fragmento actual recibido
        current_args = chunk.snapshot  # Argumentos completos acumulados hasta ahora
```

![alt text](imagenes/image-45.png)
![alt text](imagenes/image-46.png)
---

#### 24.2 Cómo funciona la validación JSON

La API no envía cada fragmento de inmediato. Almacena en búfer y valida antes de transmitir:

| Paso | Qué hace la API |
| --- | --- |
| 1 | Espera a que se complete el valor de un par clave-valor de nivel superior |
| 2 | Valida ese par contra el esquema de la herramienta |
| 3 | Envía todos los fragmentos almacenados en búfer de ese par de una sola vez |
| 4 | Repite el proceso para cada clave de nivel superior |

```json
{
  "abstract": "This paper presents a novel...",
  "meta": {
    "word_count": 847,
    "review": "This paper introduces QuanNet..."
  }
}
```

> Esto explica por qué con streaming se producen retrasos seguidos de ráfagas de texto: los fragmentos se retienen hasta que hay un par clave-valor completo y válido.

---

#### 24.3 Llamada de herramientas de grano fino

La llamada de grano fino desactiva la validación JSON en la API, entregando cada fragmento tan pronto como Claude lo genera:

| Comportamiento | Validación por defecto | Grano fino (`fine_grained=True`) |
| --- | --- | --- |
| Velocidad de entrega | Con retrasos de búfer | Inmediata, sin espera |
| Validación JSON | Activa en la API | Desactivada — el código debe validar |
| JSON parcial inválido | La API lo corrige o encapsula | Llega tal cual, puede ser inválido |

```python
run_conversation(
    messages,
    tools=[save_article_schema],
    fine_grained=True   # Desactiva la validación en la API; los fragmentos llegan sin retraso
)
```

![alt text](imagenes/image-47.png)

---

#### 24.4 Manejo de JSON no válido

Con grano fino activo, Claude puede generar JSON inválido temporalmente (ej. `"word_count": undefined`). El código debe tolerarlo:

```python
try:
    parsed_args = json.loads(chunk.snapshot)   # Intenta parsear el snapshot acumulado
except json.JSONDecodeError:
    # El snapshot aún no es JSON válido — es normal con grano fino
    print("Received invalid JSON, continuing...")
```

> Sin grano fino, la API detecta este tipo de error y puede encapsular el valor en un string, lo que podría no coincidir con el esquema esperado.

![alt text](imagenes/image-48.png) 

![alt text](imagenes/image-49.png)
---

#### 24.5 Cuándo utilizar la llamada de grano fino

| Usar grano fino cuando... | Mantener validación por defecto cuando... |
| --- | --- |
| Se necesita mostrar progreso en tiempo real al usuario | La experiencia de usuario no depende de velocidad de fragmentos |
| Se quiere procesar resultados parciales lo antes posible | Se prefiere recibir JSON siempre válido sin lógica adicional |
| Los retrasos de búfer afectan negativamente la UX | El manejo de errores JSON robusto no está implementado |

Para la mayoría de las aplicaciones el comportamiento por defecto con validación es suficiente. La llamada de grano fino es una optimización para casos donde la capacidad de respuesta inmediata es crítica.

![alt text](imagenes/image-50.png)

---

### Módulo 25: La herramienta de edición de texto

Claude incluye una herramienta integrada que no necesita crearse desde cero: el editor de texto. Permite a Claude leer, modificar y crear archivos como lo haría un editor estándar, ampliando drásticamente sus capacidades hacia las de un ingeniero de software.

---

#### 25.1 Qué puede hacer la herramienta de edición de texto

| Operación | Descripción |
| --- | --- |
| **Ver contenido** | Muestra el contenido de un archivo o directorio |
| **Ver rango de líneas** | Visualiza un rango específico de líneas de un archivo |
| **Reemplazar texto** | Sustituye texto en un archivo existente |
| **Crear archivos** | Crea nuevos archivos con el contenido indicado |
| **Insertar texto** | Inserta texto en una línea específica de un archivo |
| **Deshacer ediciones** | Revierte las ediciones recientes en un archivo |

---

#### 25.2 Comprender los requisitos de implementación

A diferencia de otras herramientas donde hay que escribir tanto el esquema como la implementación, con el editor de texto el esquema ya está integrado en Claude. Solo se debe implementar el código que ejecuta las operaciones:

| Componente | Herramientas personalizadas | Editor de texto |
| --- | --- | --- |
| Esquema JSON | Lo escribe el desarrollador | Integrado en Claude |
| Implementación de funciones | Lo escribe el desarrollador | Lo escribe el desarrollador |

> Claude sabe cómo solicitar operaciones de archivo, pero el código que realmente las ejecuta (crear, leer, reemplazar, etc.) debe ser provisto por la aplicación.

![alt text](imagenes/image-51.png)

---

#### 25.3 Versiones de esquema

Aunque el esquema principal está integrado, hay que incluir un pequeño fragmento al realizar solicitudes. El esquema exacto depende del modelo utilizado:

```python
def get_text_edit_schema(model):
    if model.startswith("claude-3-7-sonnet"):
        return {
            "type": "text_editor_20250124",   # Versión del esquema para Sonnet 3.7
            "name": "str_replace_editor",
        }
    elif model.startswith("claude-3-5-sonnet"):
        return {
            "type": "text_editor_20241022",   # Versión del esquema para Sonnet 3.5
            "name": "str_replace_editor",
        }
```

| Campo | Descripción |
| --- | --- |
| `type` | Versión del esquema de la herramienta (varía según el modelo) |
| `name` | Nombre fijo de la herramienta: siempre `"str_replace_editor"` |

Claude recibe este pequeño fragmento y lo expande automáticamente en segundo plano a la especificación completa de la herramienta.

---

#### 25.4 Ejemplo práctico

**Lectura y resumen de un archivo:**

> *"Abre el archivo `./main.py` y resume su contenido."*

Claude usará la herramienta para:

1. Ver el archivo con la operación de lectura
2. Leer el contenido completo
3. Generar y devolver el resumen

**Modificación y creación de archivos:**

> *"Abre `./main.py` y escribe una función para calcular pi con 5 dígitos de precisión. Luego crea `./test.py` para probar la implementación."*

Claude usará la herramienta para:

1. Ver el archivo `main.py` existente
2. Reemplazar su contenido con la nueva implementación
3. Crear el archivo `test.py` con las pruebas unitarias correspondientes

---

#### 25.5 ¿Por qué utilizar la herramienta de edición de texto?

| Caso de uso | Por qué la herramienta es valiosa |
| --- | --- |
| Aplicaciones que editan archivos por código | Integración directa sin depender de un editor externo |
| Entornos sin IDE avanzado | Proporciona capacidades de edición donde no hay editor disponible |
| Control preciso sobre el sistema de archivos | Define exactamente cómo Claude interactúa con los archivos de la app |

La herramienta de edición de texto permite replicar gran parte de la funcionalidad de un editor de código con IA directamente dentro de aplicaciones propias, sin necesitar herramientas de terceros.

---

### Módulo 26: La herramienta de búsqueda web

Claude incluye una herramienta de búsqueda web integrada que le permite consultar internet para obtener información actual o especializada. A diferencia de otras herramientas, Claude gestiona todo el proceso de búsqueda automáticamente — solo se necesita un esquema sencillo para activarla.

> **Requisito previo:** la organización debe habilitar la herramienta de búsqueda web desde la consola de configuración de Anthropic antes de usarla.

![alt text](imagenes/image-52.png)
---

#### 26.1 Configuración de la herramienta de búsqueda web

El esquema tiene tres campos:

```python
web_search_schema = {
    "type": "web_search_20250305",   # Versión de la herramienta integrada
    "name": "web_search",            # Nombre con el que Claude identifica la herramienta
    "max_uses": 5                    # Límite de búsquedas por conversación
}
```

| Campo | Descripción |
| --- | --- |
| `type` | Versión de la herramienta de búsqueda web |
| `name` | Nombre fijo: siempre `"web_search"` |
| `max_uses` | Máximo de búsquedas que Claude puede realizar; evita exceso de llamadas a la API |

> Una sola búsqueda puede devolver varios resultados, pero Claude puede considerar necesarias búsquedas adicionales. `max_uses` controla ese límite.

---

#### 26.2 Cómo funciona la respuesta

Cuando Claude usa la herramienta, la respuesta contiene múltiples tipos de bloques:

| Tipo de bloque | Contenido |
| --- | --- |
| **TextBlock** | Explicación de Claude sobre lo que está haciendo |
| **ServerToolUseBlock** | Consulta de búsqueda exacta que usó Claude |
| **WebSearchToolResultBlock** | Contenedor con todos los resultados de la búsqueda |
| **WebSearchResultBlock** | Resultado individual con título y URL de la fuente |
| **Bloque de citas** | Texto específico que Claude usó para respaldar sus afirmaciones, con URL de origen |

Esta estructura permite ver exactamente qué buscó Claude, qué fuentes encontró y qué fragmentos específicos cita para sustentar cada afirmación.

![alt text](imagenes/image-53.png)
---

#### 26.3 Restricción de dominios de búsqueda

El campo `allowed_domains` limita las búsquedas a dominios de confianza específicos:

```python
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5,
    "allowed_domains": ["nih.gov"]   # Solo busca en dominios autorizados
}
```

| Campo | Descripción |
| --- | --- |
| `allowed_domains` | Lista de dominios permitidos para la búsqueda (ej. `["nih.gov", "who.int"]`) |

> Útil para garantizar que Claude obtenga información basada en evidencia (ej. PubMed) en lugar de contenido de blogs o fuentes no verificadas.

---

#### 26.4 Mostrando resultados de búsqueda

Cada tipo de bloque está pensado para una representación específica en la UI:

| Bloque | Cómo mostrarlo |
| --- | --- |
| **TextBlock** | Contenido de texto normal |
| **WebSearchResultBlock** | Lista de fuentes en la parte superior de la respuesta |
| **Bloque de citas** | En línea con el texto: dominio, título de página, URL y texto citado |

Esta estructura da transparencia al usuario sobre qué información proviene de cada fuente, generando confianza en las respuestas de la IA.

---

#### 26.5 Uso práctico

La herramienta se incluye en la lista `tools` de la llamada a la API. Claude decide automáticamente cuándo una búsqueda ayudaría a responder al usuario:

```python
response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[web_search_schema]   # Claude usa la búsqueda cuando lo considera necesario
)
```

Casos donde la herramienta aporta más valor:

| Caso de uso | Por qué es útil |
| --- | --- |
| Eventos actuales y noticias recientes | No están en los datos de entrenamiento de Claude |
| Información especializada o técnica | Fuentes actualizadas superan el conocimiento estático |
| Verificación de datos | Contrasta información con fuentes autorizadas |
| Investigación con datos actualizados | Garantiza información vigente, no desactualizada |
