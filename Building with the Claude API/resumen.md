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

Claude es un modelo de lenguaje entrenado con datos hasta cierta fecha. Eso significa que tiene un conocimiento vasto pero estático: sabe mucho sobre el mundo, pero no puede consultar lo que ocurrió ayer, ni saber la temperatura actual en ninguna ciudad, ni acceder a bases de datos privadas. Las **herramientas** (tools) son el mecanismo que resuelve exactamente ese problema: le dan a Claude la capacidad de pedir información en tiempo real y actuar sobre sistemas externos.

La idea clave es que Claude no ejecuta el código — lo solicita. Claude dice "necesito saber la hora actual" y el código de la aplicación es quien ejecuta la función y le devuelve el resultado. Esta separación de responsabilidades es fundamental para entender cómo funciona el sistema.

---

#### 15.1 El problema sin herramientas

Por defecto, ante una pregunta como "¿Qué tiempo hace en San Francisco?", Claude solo puede responder que no tiene acceso a esa información. Esto ocurre porque su conocimiento viene del entrenamiento — que tiene una fecha de corte — y no tiene ninguna conexión con el mundo en tiempo real. Para el usuario, esto es una experiencia frustrante que limita enormemente la utilidad del asistente.

El mismo problema aparece en muchos otros escenarios: consultar el precio de una acción, buscar en una base de datos interna de la empresa, calcular cuántos días faltan para una fecha futura, o configurar un recordatorio. Todos son casos donde Claude tiene la inteligencia para razonar pero le falta acceso a la información o al sistema necesario.

---

#### 15.2 Cómo funciona el uso de herramientas

El protocolo de tools es un ciclo de ida y vuelta entre la aplicación y Claude. Lo importante es entender que Claude no actúa directamente — propone acciones y espera que el código las ejecute:

| Paso | Quién actúa | Qué ocurre |
| --- | --- | --- |
| 1. Solicitud inicial | Aplicación → Claude | Se envía la pregunta del usuario junto con el listado de herramientas disponibles y cómo usarlas |
| 2. Solicitud de herramienta | Claude → Aplicación | Claude decide que necesita datos externos y devuelve una solicitud estructurada indicando qué función llamar y con qué parámetros |
| 3. Recuperación de datos | Aplicación → sistema externo | El código de la aplicación ejecuta la función y obtiene los datos reales |
| 4. Respuesta final | Aplicación → Claude → Usuario | Se envían los datos a Claude con el historial completo; Claude genera la respuesta final en lenguaje natural |

Este ciclo puede repetirse múltiples veces en una sola conversación si Claude necesita encadenar varias herramientas para responder.

![alt text](image.png)

---

#### 15.3 Ejemplo: consulta del clima

Para concretar el flujo, consideremos la pregunta "¿Qué tiempo hace en San Francisco?":

1. La aplicación envía la pregunta + la descripción de una herramienta `get_weather(city)`
2. Claude entiende que necesita datos actuales y devuelve: *"necesito llamar a `get_weather` con `city="San Francisco"`"*
3. El servidor ejecuta la función real, que consulta una API meteorológica
4. La aplicación le devuelve el resultado a Claude (ej. `"18°C, parcialmente nublado"`)
5. Claude integra ese dato en una respuesta natural para el usuario

Claude nunca "sale" a internet — siempre es el código de la aplicación quien hace esa consulta. Claude solo razona sobre los datos que recibe.

---

#### 15.4 Beneficios clave

| Beneficio | Descripción |
| --- | --- |
| **Información en tiempo real** | Claude puede trabajar con datos actuales que no existen en su entrenamiento |
| **Integración de sistemas externos** | Permite conectar Claude con bases de datos, APIs, archivos y cualquier sistema con el que la aplicación pueda interactuar |
| **Respuestas dinámicas** | Cada respuesta puede basarse en el estado real del sistema, no en suposiciones estáticas |
| **Interacción estructurada** | El protocolo define exactamente qué información necesita Claude y cómo pedirla, evitando ambigüedades |

Las herramientas transforman a Claude de una base de conocimientos estática en un asistente dinámico. La clave del diseño es la claridad en la interfaz: Claude sabe qué herramientas tiene disponibles, el código sabe cuándo ejecutarlas, y ambos se comunican mediante un protocolo bien definido.

---

### Módulo 16: Proyecto práctico — sistema de recordatorios

El sistema de recordatorios es el proyecto que guía toda la unidad de tools. Es un caso de uso concreto y cotidiano que ilustra perfectamente cuándo y por qué las herramientas son necesarias.

#### 16.1 Objetivo

Permitir que Claude interprete solicitudes en lenguaje natural como:

> "Pon un recordatorio para mi cita con el médico, es el jueves de la semana que viene"

Y responda con algo como: "De acuerdo, te lo recordaré el jueves 22", habiendo calculado correctamente la fecha a partir de la fecha actual y configurado el recordatorio en el sistema. La complejidad está en que Claude necesita hacer tres cosas que por sí solo no puede hacer bien.

---

#### 16.2 Por qué es un desafío

Claude tiene tres limitaciones concretas que hacen este caso de uso imposible sin herramientas:

| Limitación | Por qué ocurre | Consecuencia |
| --- | --- | --- |
| **Conciencia temporal imprecisa** | Claude conoce aproximadamente la fecha de entrenamiento pero no la hora exacta del momento en que se le consulta | No puede anclar correctamente "el jueves que viene" |
| **Cálculo de fechas poco confiable** | Las operaciones aritméticas con fechas son propensas a errores cuando involucran semanas, meses y años | Puede decir que 177 días después del 1 de enero es una fecha incorrecta |
| **Sin mecanismo de recordatorio** | Claude es un modelo de lenguaje — no tiene acceso a ningún sistema de notificaciones ni persistencia | Aunque calculara bien la fecha, no puede "configurar" nada |

Lo interesante de este proyecto es que **no se puede resolver con prompt engineering**. No importa cómo se le pida a Claude que calcule mejor las fechas — seguirá siendo impreciso. La solución correcta es darle herramientas confiables.

---

#### 16.3 Las tres herramientas a construir

Cada herramienta está diseñada para resolver exactamente una de esas limitaciones:

| Herramienta | Limitación que resuelve | Qué hace |
| --- | --- | --- |
| `get_current_datetime` | Conciencia temporal imprecisa | Provee la fecha y hora exacta en el momento de la llamada |
| `add_duration_to_datetime` | Cálculo de fechas impreciso | Suma duraciones a fechas usando aritmética Python — siempre preciso |
| `set_reminder` | Sin mecanismo de recordatorio | Configura el recordatorio en el sistema con la fecha calculada |

Se implementan en orden de complejidad creciente. Eso permite ver primero el mecanismo básico de tool use con una sola herramienta simple, y luego ir agregando capas hasta llegar a una conversación de múltiples turnos con herramientas encadenadas.

![alt text](image-1.png)

---

#### 16.4 Principio clave

> Cuando el modelo tiene limitaciones, la solución correcta es ampliar sus capacidades con herramientas — no intentar compensar esas limitaciones con ingeniería de prompts.

Este principio vale más allá de los recordatorios: siempre que Claude necesite hacer algo que involucre precisión matemática, acceso a sistemas externos o estado persistente, la respuesta son herramientas.

---

### Módulo 17: Funciones de herramienta

Una **función de herramienta** es una función Python completamente normal — no tiene nada especial en su definición. Lo que la convierte en una herramienta para Claude es el **esquema JSON** que la acompaña (que se ve en el módulo siguiente) y el hecho de que la aplicación la registra como disponible en cada llamada a la API.

Es importante tener claro que Claude nunca ejecuta la función directamente. Claude produce una solicitud estructurada diciendo "quiero llamar a `get_current_datetime` con estos argumentos", y es el código de la aplicación quien realmente la ejecuta y le devuelve el resultado.

![alt text](image-2.png)

---

#### 17.1 Buenas prácticas al escribir funciones de herramienta

Las funciones de herramienta tienen un requisito especial: sus mensajes de error son leídos por Claude. Eso cambia cómo hay que escribirlos.

| Práctica | Por qué importa |
| --- | --- |
| **Nombres descriptivos** | Claude elige qué herramienta usar basándose en el nombre y la descripción — un nombre claro reduce llamadas incorrectas |
| **Validar entradas** | Si un parámetro obligatorio llega vacío o con un tipo incorrecto, mejor detectarlo temprano y dar un error descriptivo |
| **Mensajes de error legibles** | Claude puede leer el error y reintentar la llamada con parámetros corregidos — un error como `"date_format cannot be empty"` le permite autocorregirse |

La validación es especialmente poderosa: si Claude pasa un argumento inválido y recibe un error claro, puede entender qué estuvo mal y volver a intentarlo con los parámetros correctos. Esto convierte a Claude en un sistema que se autorrecupera de errores parciales.

![alt text](image-3.png)

---

#### 17.2 Primera herramienta: obtener fecha y hora actual

Esta función resuelve la primera limitación del sistema de recordatorios: Claude no sabe con certeza qué hora es en el momento de la consulta.

```python
from datetime import datetime

def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    # Si el formato llega vacío (ej. Claude pasó ""), lanzamos un error claro
    # Claude puede leer este mensaje y reintentar con un formato válido
    if not date_format:
        raise ValueError("date_format cannot be empty")

    # strftime convierte el objeto datetime en un string con el formato especificado
    # El formato usa los códigos de Python: %Y=año, %m=mes, %d=día, %H=hora, %M=minuto, %S=segundo
    return datetime.now().strftime(date_format)
```

El parámetro `date_format` usa los códigos de `strftime` de Python. El valor por defecto devuelve una cadena como `"2024-01-15 14:30:25"`. Claude puede pedirle a la función formatos alternativos según lo que necesite:

```python
get_current_datetime()            # "2024-01-15 14:30:25"  — año-mes-día hora:minuto:segundo
get_current_datetime("%H:%M")     # "14:30"                 — solo hora y minuto
get_current_datetime("%A, %d %B") # "Monday, 15 January"   — día de la semana y fecha larga
```

| Parámetro | Tipo | Descripción |
| --- | --- | --- |
| `date_format` | `str` | Patrón de formato compatible con `strftime`. Tiene valor por defecto, así que no es obligatorio |

---

#### 17.3 Próximos pasos

Tener la función Python es solo la mitad del trabajo. Para que Claude pueda usarla necesita dos cosas más:

1. Un **esquema JSON** que le explique qué hace la función, cuándo usarla y qué argumentos acepta — esto es lo que se ve en el Módulo 18
2. **Registrarla en la llamada a la API** para que Claude la tenga disponible

La separación entre la función y su esquema es intencional: permite cambiar la implementación sin modificar cómo Claude entiende la herramienta, y viceversa.

---

### Módulo 18: Esquemas de herramientas

Tener la función Python no es suficiente — Claude no puede leer código Python. Lo que Claude sí puede leer es un **esquema JSON** que describe qué hace la función, qué argumentos acepta y cuándo usarla. El esquema actúa como un contrato entre Claude y la aplicación: Claude lo lee para decidir si invocar la herramienta, y la aplicación lo usa para validar que Claude la está llamando correctamente.

Una mala descripción en el esquema = Claude usa la herramienta en momentos equivocados o la ignora cuando debería usarla. Por eso la calidad del esquema impacta directamente en la calidad del sistema.

![alt text](imagenes/image-27.png)

---

#### 18.1 Las tres partes de una especificación de herramienta

Todo esquema de herramienta tiene exactamente tres campos de nivel superior:

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `name` | `string` | Nombre de la función — debe ser claro y descriptivo, idealmente coincide con el nombre de la función Python |
| `description` | `string` | El campo más importante: Claude la lee para decidir si invocar la herramienta. Debe explicar qué hace, cuándo usarla y qué devuelve |
| `input_schema` | `object` | Esquema JSON (formato estándar JSON Schema) que describe los parámetros: tipos, descripciones y cuáles son obligatorios |

El campo `description` es el más crítico. Claude toma decisiones basándose en él — no en el código de la función. Una descripción vaga puede hacer que Claude use la herramienta en situaciones incorrectas.

---

#### 18.2 Cómo escribir descripciones efectivas

Una buena descripción de herramienta responde cuatro preguntas:

- **¿Qué hace?** — la acción principal en una frase
- **¿Cuándo debe usarla Claude?** — los casos de uso específicos que deben activarla
- **¿Qué devuelve?** — el formato y tipo del resultado
- **¿Qué hace cada argumento?** — descripción individual de cada parámetro

Cuanto más específica sea la descripción, menos ambigüedad tiene Claude para decidir cuándo y cómo usarla.

![alt text](imagenes/image-28.png)

---

#### 18.3 Generar el esquema con Claude

Escribir esquemas JSON a mano es tedioso y propenso a errores. Una alternativa práctica es pedirle al propio Claude que lo genere:

1. Copiar el código de la función Python
2. Pedirle a Claude: *"Write a valid JSON schema specification for tool calling for this function. Follow the best practices listed in the attached documentation."*
3. Adjuntar la documentación oficial de Anthropic sobre tool use como contexto adicional
4. Copiar el esquema generado al código

Esto funciona bien porque Claude entiende tanto la sintaxis de JSON Schema como las convenciones de la API de Anthropic.

![alt text](imagenes/image-29.png)

---

#### 18.4 Implementación del esquema

Por convención, el esquema se nombra igual que la función con el sufijo `_schema`. Esto hace explícita la relación entre la función y su descripción para Claude:

```python
def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")
    return datetime.now().strftime(date_format)

# Convención: function_name + "_schema" — deja clara la relación entre función y descripción
get_current_datetime_schema = {
    "name": "get_current_datetime",
    # Claude lee esta descripción para decidir cuándo invocar la herramienta
    "description": "Returns the current date and time formatted according to the specified format. Use this when you need to know the current time or date.",
    "input_schema": {
        "type": "object",      # Siempre "object" — los argumentos van como propiedades
        "properties": {
            "date_format": {
                "type": "string",   # Tipo del parámetro que Claude debe pasar
                "description": "A string specifying the format of the returned datetime. Uses Python's strftime format codes. Example: '%Y-%m-%d' for date only.",
                "default": "%Y-%m-%d %H:%M:%S"   # Valor que usa Claude si no especifica otro
            }
        },
        "required": []   # Lista vacía: todos los parámetros tienen valor por defecto, ninguno es obligatorio
    }
}
```

El campo `required` es una lista de nombres de parámetros que Claude **debe** incluir en la llamada. Si está vacío, todos son opcionales. Si un parámetro no tiene `default` pero tampoco está en `required`, Claude puede omitirlo — aunque eso probablemente cause un error en la función.

---

#### 18.5 Agregar seguridad de tipos con `ToolParam`

El diccionario del esquema funciona, pero puede tener errores de tipeo que solo se detectan en runtime. `ToolParam` agrega verificación de tipos en tiempo de escritura:

```python
from anthropic.types import ToolParam   # TypedDict de la SDK de Anthropic

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format.",
    "input_schema": { ... }
})
# ToolParam no cambia el comportamiento en ejecución
# Pero si el IDE o un type checker detecta que falta un campo requerido, avisa antes de correr el código
```

| Aspecto | Diccionario simple | `ToolParam` |
| --- | --- | --- |
| Comportamiento en ejecución | Idéntico | Idéntico |
| Detección de errores de estructura | Solo en runtime | En tiempo de escritura (IDE/mypy) |
| Uso recomendado | Prototipos rápidos | Código en producción |

---

### Módulo 19: Manejo de bloques de mensajes

Cuando Claude usa herramientas, la estructura de la respuesta cambia de manera fundamental. En una conversación sin herramientas, `response.content` es simplemente texto. Cuando hay herramientas involucradas, `response.content` se convierte en una **lista de bloques**, donde cada bloque tiene un tipo diferente. Entender esta estructura es crucial para poder procesar las respuestas correctamente.

---

#### 19.1 Llamada a la API con herramientas

La diferencia respecto a una llamada normal está en el parámetro `tools`. Al incluirlo, Claude sabe qué herramientas tiene disponibles y puede decidir usarlas:

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
    tools=[get_current_datetime_schema],   # Claude lee estos schemas para decidir qué herramientas invocar
)
# Si Claude decide usar una herramienta, response.stop_reason == "tool_use"
# Si responde directamente, response.stop_reason == "end_turn"
```

---

#### 19.2 Estructura de un mensaje multibloque

Cuando Claude decide usar una herramienta, `response.content` no es un string — es una lista con dos elementos:

| Índice | Bloque | Tipo | Contenido típico |
| --- | --- | --- | --- |
| `[0]` | **TextBlock** | `"text"` | Texto en lenguaje natural explicando lo que Claude va a hacer |
| `[1]` | **ToolUseBlock** | `"tool_use"` | Instrucción estructurada: qué función llamar y con qué argumentos |

El **ToolUseBlock** tiene tres campos internos que son los que el código necesita para ejecutar la función:

| Campo del ToolUseBlock | Descripción |
| --- | --- |
| `id` | Identificador único de esta llamada — se usa después para vincular el resultado con la solicitud |
| `name` | Nombre de la función a invocar, ej. `"get_current_datetime"` |
| `input` | Diccionario con los argumentos que Claude quiere pasar a la función |

![alt text](imagenes/image-30.png)

---

#### 19.3 Gestionar el historial con mensajes multibloque

Este es uno de los errores más comunes al implementar tool use. Al guardar la respuesta de Claude en el historial, hay que guardar **toda** la lista de bloques — no solo el texto:

```python
messages.append({
    "role": "assistant",
    # response.content es la lista completa [TextBlock, ToolUseBlock]
    # Si se guardara solo response.content[0].text, el ToolUseBlock se pierde
    "content": response.content
})
# ¿Por qué importa? En la siguiente llamada a la API, el historial debe contener
# el ToolUseBlock con su id. Si ese id no está, la API rechazará el tool_result
# que hace referencia a él — BadRequestError.
```

La regla es simple: cuando Claude responde con una lista de bloques, esa lista completa entra al historial como está.

---

#### 19.4 Flujo completo de uso de herramienta

```text
1. Enviar mensaje del usuario + esquemas de herramientas disponibles → Claude
2. Claude responde: [TextBlock: "Voy a consultar la hora"] + [ToolUseBlock: {name, id, input}]
3. El código lee el ToolUseBlock, extrae name e input, ejecuta la función Python
4. Se construye un bloque tool_result con el resultado y el id de la solicitud
5. Se envía el historial completo (incluyendo tool_result) → Claude
6. Claude genera la respuesta final para el usuario
```

Cada paso es necesario. Si se omite cualquiera — por ejemplo, si se salta el ToolUseBlock del historial o se manda un `tool_result` con el `id` incorrecto — la API devolverá un error.

![alt text](imagenes/image-31.png)

---

#### 19.5 Actualizar las funciones auxiliares

Las funciones `add_user_message()` y `add_assistant_message()` escritas antes de introducir herramientas solo manejaban strings. Para soportar mensajes multibloque hay que actualizarlas para aceptar tanto strings como listas de bloques:

```python
def add_assistant_message(messages, content):
    messages.append({
        "role": "assistant",
        # content puede ser:
        # - un string simple (respuesta sin herramientas)
        # - una lista de bloques [TextBlock, ToolUseBlock] (respuesta con herramientas)
        "content": content
    })
```

La razón por la que el mismo campo `content` puede recibir ambos tipos es que la API de Anthropic acepta ambos formatos: un string es equivalente a una lista con un solo TextBlock.

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

Una vez que Claude devuelve un `ToolUseBlock`, el flujo pasa al código de la aplicación: ejecutar la función real y enviar el resultado de vuelta a Claude. Este paso completa el ciclo y le da a Claude la información que necesita para generar su respuesta final.

---

#### 20.1 Ejecutar la función de la herramienta

El `ToolUseBlock` tiene el campo `input` con los argumentos que Claude quiere usar. El operador `**` de Python los desempaqueta como argumentos nombrados de la función:

```python
# response.content[1] es el ToolUseBlock
# .input es un diccionario, ej: {"date_format": "%H:%M:%S"}
tool_block = response.content[1]

# ** desempaqueta el dict como kwargs: equivale a get_current_datetime(date_format="%H:%M:%S")
result = get_current_datetime(**tool_block.input)
```

Si la función no tiene parámetros, `input` será un dict vacío `{}` y `**{}` no pasa ningún argumento — lo cual es correcto.

![alt text](imagenes/image-32.png)

---

#### 20.2 Estructura del bloque de resultado (`tool_result`)

El resultado se envía como un mensaje de usuario con un bloque de tipo `"tool_result"`. El campo más crítico es `tool_use_id`: debe coincidir exactamente con el `id` del `ToolUseBlock` que generó la solicitud. Es así como Claude sabe que este resultado responde a esa solicitud específica.

```python
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": tool_block.id,   # El mismo id que tenía el ToolUseBlock — los vincula
        "content": result,              # El resultado de ejecutar la función (string o JSON serializado)
        "is_error": False               # False = ejecución exitosa; True = la función lanzó una excepción
    }]
})
```

| Propiedad | Tipo | Descripción |
| --- | --- | --- |
| `type` | `str` | Siempre `"tool_result"` — identifica el tipo de bloque |
| `tool_use_id` | `str` | ID del `ToolUseBlock` al que responde. Si no coincide, la API devuelve error |
| `content` | `str` | Resultado de la función. Si es un objeto, serializarlo con `json.dumps()` antes |
| `is_error` | `bool` | `True` si ocurrió una excepción durante la ejecución. Claude puede leer el error y reintentar |

![alt text](imagenes/image-33.png)

---

#### 20.3 Manejo de múltiples llamadas a herramientas

Claude puede incluir varios `ToolUseBlock` en una misma respuesta cuando las herramientas son independientes entre sí (ej. calcular dos fechas al mismo tiempo). En ese caso, el mensaje de usuario debe contener **un `tool_result` por cada `ToolUseBlock`**, todos juntos en la misma lista:

```python
# Si Claude pidió dos herramientas, el mensaje de resultado tiene dos bloques
messages.append({
    "role": "user",
    "content": [
        {"type": "tool_result", "tool_use_id": id_herramienta_1, "content": resultado_1, "is_error": False},
        {"type": "tool_result", "tool_use_id": id_herramienta_2, "content": resultado_2, "is_error": False},
    ]
})
```

Los `id` únicos de cada bloque permiten vincular cada resultado con su solicitud, sin importar el orden en que lleguen.

![alt text](imagenes/image-34.png)

![alt text](imagenes/image-35.png)

---

#### 20.4 El historial completo en la solicitud de seguimiento

Después de agregar el `tool_result`, el historial tiene exactamente esta estructura antes de hacer la siguiente llamada a la API:

```text
Posición 1  │  role: "user"       │  Pregunta original del usuario
Posición 2  │  role: "assistant"  │  [TextBlock] + [ToolUseBlock] — la respuesta de Claude con la solicitud
Posición 3  │  role: "user"       │  [tool_result] — el resultado de ejecutar la función
```

Este historial completo le da a Claude todo el contexto para generar la respuesta final.

---

#### 20.5 Solicitud final a Claude

La llamada de seguimiento incluye el historial completo y, crucialmente, los esquemas de herramientas de nuevo. Aunque no se espere otra llamada a herramientas, incluir los esquemas es obligatorio: Claude necesita entender las referencias a herramientas que ya aparecen en el historial.

```python
final_response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,                          # Historial con las 3 posiciones descritas arriba
    tools=[get_current_datetime_schema]         # Obligatorio aunque no se espere otra tool call
)
# final_response.stop_reason == "end_turn" — Claude ya tiene lo que necesita y responde directamente
print(final_response.content[0].text)          # La respuesta final en lenguaje natural
```

![alt text](imagenes/image-36.png)

Si se omiten los esquemas en la llamada de seguimiento, la API puede fallar porque el historial contiene referencias a herramientas que la llamada actual no reconoce.

---

### Módulo 21: Conversaciones de varios turnos con herramientas

Hasta ahora el flujo era simple: Claude pide una herramienta, el código la ejecuta, Claude responde. Pero muchas preguntas requieren **varias herramientas en secuencia**, donde el resultado de una es la entrada de la siguiente. Un ejemplo concreto: "¿Qué día será dentro de 103 días?". Para responder, Claude necesita primero saber cuál es la fecha de hoy (`get_current_datetime`), y con esa fecha calcular cuánto es hoy + 103 días (`add_duration_to_datetime`). No puede hacer ambas cosas en un solo turno.

La solución es un bucle que automatiza el ciclo completo: Claude pide herramienta → el código la ejecuta → Claude pide otra herramienta → el código la ejecuta → Claude da la respuesta final.

![alt text](imagenes/image-37.png)

---

#### 21.1 Patrón de múltiples turnos

```text
1. Usuario pregunta: "¿Qué día será dentro de 103 días?"
2. Claude solicita → get_current_datetime (necesita saber la fecha de hoy primero)
3. Servidor ejecuta get_current_datetime() → devuelve "2024-01-15"
4. Claude recibe el resultado y solicita → add_duration_to_datetime(date="2024-01-15", days=103)
5. Servidor ejecuta la función → devuelve "2024-04-28"
6. Claude tiene todo lo que necesita → genera la respuesta final: "El día será el 28 de abril de 2024"
```

Claude no puede hacer ambas operaciones en un solo paso porque cada herramienta depende del resultado de la anterior. El bucle permite manejar esta cadena de dependencias automáticamente.

![alt text](imagenes/image-38.png)

---

#### 21.2 Bucle de conversación

El patrón `while True` con `break` es la forma idiomática de implementar este ciclo. La condición de salida es simple: cuando `stop_reason` ya no es `"tool_use"`, Claude terminó.

```python
def run_conversation(messages):
    while True:
        response = chat(messages)   # Llama a Claude con el historial acumulado hasta ahora

        # Agrega la respuesta completa (TextBlock + posibles ToolUseBlocks) al historial
        add_assistant_message(messages, response)

        # Si Claude no pidió más herramientas, tiene su respuesta final → salir del bucle
        if response.stop_reason != "tool_use":
            break

        # Claude quiere una o más herramientas — ejecutarlas y devolver los resultados
        tool_result_blocks = run_tools(response)

        # Los resultados entran al historial como mensaje de usuario para el próximo turno
        add_user_message(messages, tool_result_blocks)

    # Al salir del while, el último mensaje del asistente es la respuesta final
    return messages
```

El bucle puede iterar cualquier número de veces. Cada iteración agrega dos mensajes al historial: la respuesta del asistente y el `tool_result` del usuario. Cuando Claude finalmente responde sin pedir herramientas, `stop_reason == "end_turn"` y el bucle termina.

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

El módulo 21 mostró el concepto del bucle de conversación. Este módulo profundiza en la implementación concreta de cada componente: cómo detectar si Claude quiere una herramienta, cómo procesar varias herramientas en paralelo, cómo manejar errores sin romper el flujo, y cómo agregar nuevas herramientas sin tocar la lógica central.

---

#### 22.1 Detección de solicitudes de herramientas

`stop_reason` es el campo que le dice a la aplicación por qué Claude dejó de generar texto. No es solo un indicador técnico — es la señal de control del bucle:

| Valor de `stop_reason` | Significado | Qué hace la aplicación |
| --- | --- | --- |
| `"tool_use"` | Claude necesita ejecutar una o más herramientas antes de poder continuar | Ejecutar las herramientas y volver a llamar a Claude |
| `"end_turn"` | Claude completó su respuesta — no necesita más información | Salir del bucle y mostrar la respuesta al usuario |
| `"max_tokens"` | Se alcanzó el límite de tokens antes de terminar | Manejar como error o aumentar `max_tokens` |

```python
if response.stop_reason != "tool_use":
    break   # Cualquier stop_reason que no sea "tool_use" significa que Claude terminó
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
![alt text](imagenes/image-54.png)
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

La belleza del diseño modular de tools es que agregar una nueva herramienta no requiere tocar la lógica central del bucle (`run_conversation`, `run_tools`). Todo lo nuevo se localiza en dos lugares: la lista de schemas que se le pasa a Claude, y el enrutador que despacha las llamadas a las funciones correctas.

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

Cuando se combina streaming con herramientas, aparece una tensión: el streaming existe para dar respuestas en tiempo real, pero los argumentos de las herramientas son JSON que deben estar completos y válidos antes de que el código pueda ejecutar la función. La API resuelve esto con un mecanismo de **buffering y validación** que introduce un compromiso entre velocidad y corrección.

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

Hasta ahora todas las herramientas vistas eran **custom**: el desarrollador escribe tanto el esquema como la función Python. La herramienta de edición de texto rompe ese patrón. El esquema ya está integrado en Claude — no hay que escribirlo. Pero la implementación de las operaciones de archivo sí es responsabilidad del desarrollador. Es el caso inverso: Claude sabe qué quiere hacer con los archivos, pero necesita que alguien le provea el código que realmente ejecuta esas operaciones en el sistema de archivos local.

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

La búsqueda web es el segundo ejemplo de herramienta integrada en Claude (además del editor de texto). La diferencia con las herramientas custom es total: aquí Claude no solo tiene el esquema integrado, sino que también **ejecuta la búsqueda internamente**. La aplicación no necesita conectarse a ninguna API de búsqueda ni procesar los resultados — Claude lo gestiona todo. El rol del desarrollador se reduce a activar la herramienta con un esquema mínimo y luego procesar los bloques que Claude devuelve en la respuesta.

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

---

## Unidad 6: RAG y búsqueda agencial

| Módulo | Tema |
| --- | --- |
| 27 | Presentamos la generación aumentada de recuperación |
| 28 | Estrategias de segmentación de texto |
| 29 | Incrustaciones de texto |
| 30 | El flujo RAG completo |
| 31 | Implementación del flujo RAG |
| 32 | Búsqueda léxica BM25 |
| 33 | Una canalización RAG de índice múltiple |

---

### Módulo 27: Presentamos la generación aumentada de recuperación (RAG)

La Generación Aumentada de Recuperación (RAG) es una técnica que permite trabajar con documentos extensos que no caben en un solo prompt. En lugar de incluir toda la información, RAG divide los documentos en fragmentos y solo incluye las partes más relevantes al responder cada pregunta.

---

#### 27.1 El problema de los documentos extensos

Ante un documento de 800 páginas y una pregunta específica como "¿Qué factores de riesgo tiene esta empresa?", hay dos enfoques posibles:

| Opción | Descripción | Limitaciones |
| --- | --- | --- |
| **Incluir todo** | Extraer todo el texto e insertarlo en el mensaje | Límite de longitud del prompt, menor eficacia de Claude con prompts muy largos, mayor costo y latencia |
| **RAG** | Dividir el documento en fragmentos y seleccionar solo los relevantes | Requiere preprocesamiento y un mecanismo de búsqueda |

![alt text](imagenes/image-55.png)
---

#### 27.2 Opción 1: incluir todo en la solicitud

```xml
Answer the user's question about the financial document.

<user_question>
{user_question}
</user_question>

<financial_document>
{financial_document}   <!-- El documento completo: puede ser demasiado largo -->
</financial_document>
```
![alt text](imagenes/image-56.png)
Problemas de este enfoque:

- Existe un límite estricto en la longitud del prompt
- Claude se vuelve menos eficaz con prompts muy largos
- Los mensajes más extensos cuestan más y tardan más en procesarse

---

#### 27.3 Opción 2: RAG — dividir y recuperar

RAG adopta un enfoque de dos etapas:

| Etapa | Cuándo ocurre | Qué hace |
| --- | --- | --- |
| **Preprocesamiento** | Una sola vez, al ingerir el documento | Divide el documento en fragmentos más pequeños |
| **Recuperación** | Cada vez que llega una pregunta | Busca los fragmentos más relevantes e incluye solo esos en el prompt |

**Ejemplo:** ante la pregunta "¿Qué riesgos enfrenta esta empresa?", el sistema busca en los fragmentos, encuentra la sección "Factores de riesgo" e incluye solo ese fragmento en el mensaje a Claude.

![alt text](imagenes/image-57.png)
![alt text](imagenes/image-58.png)
---


#### 27.4 Beneficios y desafíos de RAG

| Beneficios | Desafíos |
| --- | --- |
| Claude se enfoca solo en el contenido relevante | Requiere un paso de preprocesamiento |
| Se adapta a documentos de gran tamaño | Necesita un mecanismo de búsqueda |
| Funciona con múltiples documentos | Los fragmentos pueden perder contexto |
| Prompts más pequeños → menor costo y latencia | Hay muchas estrategias de segmentación posibles |

---

#### 27.5 Cuándo usar RAG

RAG implica más trabajo inicial que simplemente incluir toda la información en el prompt. Conviene evaluar si los beneficios justifican la complejidad según el caso de uso:

| Usar RAG cuando... | Preferir prompt completo cuando... |
| --- | --- |
| El documento es demasiado extenso para el prompt | El documento es corto y cabe en el contexto |
| Se trabaja con múltiples documentos | Se necesita una solución rápida y simple |
| Se necesita optimizar costo y rendimiento | La complejidad adicional no aporta valor |

> RAG prioriza escalabilidad y eficiencia sobre simplicidad. Requiere más trabajo inicial, pero permite trabajar con colecciones de documentos que serían imposibles de manejar con relleno directo del prompt.

---

### Módulo 28: Estrategias de segmentación de texto

La segmentación de texto es uno de los pasos más críticos en un sistema RAG. La forma en que se dividen los documentos impacta directamente en la calidad de las respuestas: una estrategia deficiente puede hacer que Claude reciba contexto completamente irrelevante y genere respuestas erróneas o incoherentes.

Para entender por qué esto importa tanto, pensá en este escenario: tenés un documento con secciones sobre investigación médica e ingeniería de software. Si la segmentación es pobre, la pregunta "¿Cuántos errores corrigieron los ingenieros este año?" puede terminar devolviendo fragmentos de la sección médica — simplemente porque ahí también aparece la palabra "error", pero en un contexto completamente diferente. Claude no tiene manera de saber que ese fragmento es irrelevante si es lo que le pasás.

Existen cuatro estrategias principales para segmentar texto, cada una con un nivel creciente de sofisticación y costo computacional.

![alt text](imagenes/image-59.png)

![alt text](imagenes/image-60.png)

---

#### 28.1 Segmentación basada en el tamaño

Es el método más simple: el texto se divide en cadenas de longitud fija, sin importar el contenido. Funciona como un cuchillo que corta el documento cada N caracteres.

El problema obvio es que un corte puede caer en medio de una palabra o separar un encabezado de su párrafo. Para mitigar esto se introduce el concepto de **solapamiento** (`chunk_overlap`): cada fragmento comparte algunos caracteres con el fragmento anterior. Así, si una idea importante queda cortada al final de un fragmento, también aparece al principio del siguiente, lo que da más contexto al sistema de recuperación.

![alt text](imagenes/image-61.png)

```python
def chunk_by_char(text, chunk_size=150, chunk_overlap=20):
    chunks = []
    start_idx = 0

    while start_idx < len(text):
        end_idx = min(start_idx + chunk_size, len(text))   # No superar el largo del texto
        chunk_text = text[start_idx:end_idx]
        chunks.append(chunk_text)

        # Si todavía hay texto por procesar, el próximo fragmento empieza
        # chunk_overlap caracteres antes del final del actual — así se solapan
        start_idx = (
            end_idx - chunk_overlap if end_idx < len(text) else len(text)
        )

    return chunks
```

| Parámetro | Descripción |
| --- | --- |
| `chunk_size` | Longitud máxima de cada fragmento en caracteres. Valores típicos en producción: 500–1500 caracteres |
| `chunk_overlap` | Caracteres compartidos entre fragmentos consecutivos. Suele ser el 10–20% del `chunk_size` |

| Ventaja | Desventaja |
| --- | --- |
| Fácil de implementar y depurar | Puede cortar palabras o frases a mitad |
| Funciona con cualquier tipo de documento, incluido código | Los encabezados pueden quedar separados de su contenido |
| Predecible: siempre genera fragmentos del mismo tamaño | Los fragmentos pueden carecer de coherencia semántica |

---

#### 28.2 Segmentación basada en la estructura

En lugar de cortar cada N caracteres, este enfoque respeta la estructura natural del documento: divide por encabezados, párrafos o secciones. El resultado es que cada fragmento contiene una unidad de significado completa, no un corte arbitrario del texto.
![alt text](imagenes/image-62.png)
Funciona muy bien con documentos Markdown, donde los encabezados están marcados con `##`, `###`, etc. Al dividir por esos marcadores, cada fragmento corresponde exactamente a una sección del documento.

```python
def chunk_by_section(document_text):
    # El patrón busca saltos de línea seguidos de "## " (encabezado nivel 2 de Markdown)
    # re.split divide el texto en cada ocurrencia de ese patrón
    pattern = r"\n## "
    return re.split(pattern, document_text)
```

La limitación es clara: si el documento no tiene estructura explícita — como muchos PDFs escaneados o texto plano exportado desde un procesador de texto — este método no puede aplicarse.

| Ventaja | Desventaja |
| --- | --- |
| Fragmentos semánticamente completos y con sentido propio | Solo funciona con documentos que tienen marcadores estructurales claros |
| La recuperación es más precisa porque cada fragmento cubre un tema | No aplica a PDFs, texto plano o documentos sin formato |

---

#### 28.3 Segmentación basada en la semántica

Este es el enfoque más sofisticado. La idea es dividir el texto en oraciones individuales y luego usar modelos de procesamiento del lenguaje natural (NLP) para calcular qué tan relacionadas están dos oraciones consecutivas. Si su relación semántica baja de un umbral, ahí se hace el corte; si siguen siendo temáticamente cercanas, se mantienen en el mismo fragmento.

El resultado son fragmentos que agrupan ideas relacionadas independientemente de su extensión o de si están en la misma sección del documento. En teoría, es la estrategia que produce los fragmentos más útiles para el sistema de recuperación. En la práctica, requiere modelos de embeddings adicionales para calcular similitud semántica en tiempo real, lo que lo hace costoso en términos de cómputo y más difícil de mantener.

| Ventaja | Desventaja |
| --- | --- |
| Produce los fragmentos más relevantes semánticamente | Alto costo computacional — requiere modelos de embeddings durante el preprocesamiento |
| Respeta el significado del texto, no solo su formato | Implementación compleja y más difícil de depurar |

---

#### 28.4 Segmentación basada en oraciones

Es la solución intermedia más práctica. En lugar de cortar por caracteres o por encabezados, divide el texto en oraciones completas usando expresiones regulares y las agrupa en fragmentos de N oraciones. El solapamiento ahora opera a nivel de oraciones en lugar de caracteres, lo que preserva mejor el contexto.

La regex `(?<=[.!?])\s+` usa un **lookbehind**: busca un espacio en blanco que esté precedido por `.`, `!` o `?`. Así detecta el final de una oración sin consumir el punto en sí.

```python
def chunk_by_sentence(text, max_sentences_per_chunk=5, overlap_sentences=1):
    # (?<=[.!?]) es un lookbehind: detecta un espacio DESPUÉS de punto, ! o ?
    # Esto divide el texto en oraciones completas sin perder la puntuación
    sentences = re.split(r"(?<=[.!?])\s+", text)

    chunks = []
    start_idx = 0

    while start_idx < len(sentences):
        end_idx = min(start_idx + max_sentences_per_chunk, len(sentences))
        current_chunk = sentences[start_idx:end_idx]
        chunks.append(" ".join(current_chunk))   # Vuelve a unir las oraciones en un string

        # El siguiente fragmento empieza overlap_sentences antes del fin del actual
        # Ej: con max=5 y overlap=1, el siguiente empieza en la oración 5 (no en la 6)
        start_idx += max_sentences_per_chunk - overlap_sentences

        if start_idx < 0:
            start_idx = 0

    return chunks
```

| Parámetro | Descripción |
| --- | --- |
| `max_sentences_per_chunk` | Máximo de oraciones por fragmento. Valores típicos: 3–7 oraciones |
| `overlap_sentences` | Oraciones compartidas entre fragmentos consecutivos. Con 1, el último fragmento siempre aparece al inicio del siguiente |

---

#### 28.5 Elegir la estrategia correcta

No existe una estrategia universalmente óptima — la elección depende del tipo de documentos con los que trabajás y de cuánta complejidad estás dispuesto a manejar.

| Estrategia | Cuándo usarla |
| --- | --- |
| **Basada en estructura** | Controlás el formato de los documentos (ej. informes internos en Markdown) |
| **Basada en oraciones** | Buen punto intermedio para textos en lenguaje natural sin formato definido |
| **Basada en tamaño** | La más confiable para producción: funciona con cualquier contenido, incluido código |
| **Semántica** | Cuando la calidad de recuperación es prioritaria y se dispone de recursos computacionales |

En producción, la segmentación por **tamaño con solapamiento** suele ganar porque combina simplicidad con consistencia. No genera fragmentos perfectos, pero es predecible, fácil de ajustar y no falla ante documentos con formatos inesperados. Ajustar `chunk_size` y `chunk_overlap` ya permite mejorar significativamente los resultados sin cambiar de estrategia.

---

### Módulo 29: Incrustaciones de texto

Una vez que el documento está dividido en fragmentos, el siguiente problema es: ¿cómo sabemos cuáles son relevantes para la pregunta del usuario? Si el documento tiene miles de fragmentos, no se puede enviar todo a Claude — hay que seleccionar solo los más útiles. Eso es exactamente lo que resuelven las incrustaciones de texto.
![alt text](imagenes/image-63.png)
---

#### 29.1 El problema de búsqueda

Encontrar fragmentos relevantes es, en esencia, un problema de búsqueda. La solución más intuitiva sería buscar palabras exactas de la pregunta dentro de los fragmentos — pero ese enfoque falla rápido. Si alguien pregunta "¿cuáles son los riesgos financieros?" y el documento habla de "exposición al mercado" o "volatilidad cambiaria", una búsqueda por palabras clave no va a encontrar esa sección porque las palabras exactas no coinciden.

La solución es la **búsqueda semántica**: en lugar de comparar palabras, se compara *significado*. Y para hacer eso, se necesita una forma de representar el significado de un texto como algo que las computadoras puedan comparar matemáticamente — ahí entran los embeddings.

---

#### 29.2 Qué es una incrustación de texto

Una incrustación (embedding) es una lista de números que representa el significado de un texto. El proceso es el siguiente:

1. Se pasa un texto a un modelo de incrustación
2. El modelo devuelve una lista larga de números (típicamente entre 256 y 3072 valores)
3. Cada número está en el rango **-1 a +1**
4. En conjunto, esos números capturan el "significado" del texto en un espacio matemático
![alt text](imagenes/image-64.png)

La analogía útil es pensar en un mapa: dos ciudades cercanas en el mapa son geográficamente similares. Con embeddings pasa lo mismo — dos textos con significados similares producen listas de números que están "cerca" en ese espacio matemático. Esa cercanía se puede medir con operaciones como la **similitud coseno**.

---

#### 29.3 Entendiendo los números

Cada número en el embedding es una "puntuación" de alguna cualidad del texto. Podría ser tentador imaginar que el número 47 representa "cuán positivo es el texto" o el número 312 "cuánto habla sobre finanzas" — pero eso no es cómo funciona.

![alt text](imagenes/image-65.png)

El significado real de cada dimensión lo aprende el modelo durante el entrenamiento con enormes cantidades de texto, y **no es directamente interpretable por humanos**. Lo que sí sabemos es que el modelo aprendió a organizar los textos de modo que textos similares queden cerca entre sí en ese espacio de miles de dimensiones. No necesitamos entender cada número individualmente — solo importa la distancia entre vectores.

---

#### 29.4 VoyageAI para incrustaciones

Anthropic no provee embeddings directamente, por lo que el proveedor recomendado es **VoyageAI**. Requiere una cuenta separada y su propia clave API (hay un nivel gratuito para empezar).

Agregar la clave al archivo `.env`:

```env
VOYAGE_API_KEY="your_key_here"
```

Instalar la librería:

```bash
%pip install voyageai
```

---

#### 29.5 Implementación

```python
from dotenv import load_dotenv
import voyageai

load_dotenv()                  # Carga la VOYAGE_API_KEY desde el archivo .env
voyage_client = voyageai.Client()   # Inicializa el cliente con la clave de entorno

def generate_embedding(text, model="voyage-3-large", input_type="query"):
    # client.embed() recibe una LISTA de textos, por eso pasamos [text]
    # Devuelve un objeto con .embeddings — una lista de listas de floats
    result = voyage_client.embed([text], model=model, input_type=input_type)
    return result.embeddings[0]   # Devuelve el embedding del primer (y único) texto
```
![alt text](imagenes/image-66.png)

| Parámetro | Descripción |
| --- | --- |
| `text` | El texto a convertir en embedding (puede ser una pregunta o un fragmento del documento) |
| `model` | Modelo de VoyageAI a usar. `"voyage-3-large"` ofrece alta calidad; hay modelos más ligeros para producción de bajo costo |
| `input_type` | `"query"` para preguntas del usuario; `"document"` para fragmentos del corpus. Esta distinción mejora la calidad de la búsqueda |

El parámetro `input_type` merece atención especial: VoyageAI optimiza los embeddings de forma diferente según si el texto es una consulta (lo que busca el usuario) o un documento (lo que se va a recuperar). Usar el tipo correcto mejora la precisión del sistema de recuperación.

La función devuelve una lista de números de coma flotante — el embedding del texto. El próximo paso es comparar ese vector con los embeddings de todos los fragmentos del documento para encontrar los más similares, lo que constituye la base de la búsqueda semántica en RAG.

---

### Módulo 30: El flujo RAG completo

Este módulo conecta todo lo visto hasta ahora — segmentación, embeddings y búsqueda — en un proceso unificado de seis pasos. Lo importante no es solo memorizar los pasos, sino entender **por qué** el sistema puede encontrar información relevante aunque las palabras exactas de la pregunta no aparezcan en el documento.

---

#### 30.1 Paso 1 — Dividir el texto fuente en fragmentos

El primer paso es el preprocesamiento: el documento se divide en fragmentos con alguna de las estrategias del Módulo 28. Para este ejemplo, dos secciones:

| Fragmento | Sección | Contenido |
| --- | --- | --- |
| #1 | Investigación médica | *"Este año se han producido avances significativos en nuestra comprensión de XDR-47, un virus que no habíamos visto antes."* |
| #2 | Ingeniería de software | *"Esta división dedicó un esfuerzo significativo al estudio de diversos vectores de infección en nuestros sistemas distribuidos."* |

Notá que ambos fragmentos comparten vocabulario similar ("infección", "vectores") pero hablan de temas completamente distintos. Una búsqueda por palabras clave exactas no sabría diferenciarlos — los embeddings sí.

---

#### 30.2 Paso 2 — Generar embeddings para cada fragmento

Cada fragmento se convierte en un vector numérico usando el modelo de embeddings. Para ilustrar el concepto con un ejemplo simplificado, imaginemos un modelo que devuelve solo dos números con significado interpretable:

- **Número 1:** cuánto habla el texto sobre medicina
- **Número 2:** cuánto habla el texto sobre ingeniería de software

![alt text](imagenes/image-67.png)

> En la realidad, los modelos de embeddings devuelven miles de dimensiones y ninguna de ellas tiene un significado humano interpretable directamente. Este ejemplo de dos dimensiones es solo una analogía pedagógica.

| Fragmento | Vector crudo | Después de normalización |
| --- | --- | --- |
| Investigación médica | [0.97, 0.34] | [0.944, 0.331] |
| Ingeniería de software | [0.30, 0.97] | [0.295, 0.955] |

La **normalización** escala cada vector para que tenga magnitud 1.0 (es decir, que su longitud en el espacio vectorial sea exactamente 1). La API de VoyageAI hace esto automáticamente. Permite que la comparación entre vectores mida solo la *dirección* — o sea, el significado — sin verse afectada por la longitud de los textos.
![alt text](imagenes/image-68.png)
![alt text](imagenes/image-69.png)

---

#### 30.3 Paso 3 — Almacenar en una base de datos vectorial

Los vectores normalizados se guardan en una **base de datos vectorial**: una base de datos especializada optimizada para almacenar listas de números y para hacer búsquedas de similitud de forma eficiente.

Este paso es el límite del **preprocesamiento**: todo lo realizado hasta aquí ocurre una sola vez, antes de que llegue cualquier consulta del usuario. A partir de aquí, el sistema espera.

![alt text](imagenes/image-70.png)
---

#### 30.4 Paso 4 — Procesar la consulta del usuario

Cuando el usuario hace una pregunta, esa pregunta pasa por el **mismo modelo de embeddings** que procesó los fragmentos:

> *"¿Qué hizo el departamento de ingeniería de software este año?"*

El modelo genera un vector para esta consulta, por ejemplo `[0.1, 0.89]` → normalizado: `[0.112, 0.993]`. Este vector tiene puntuación médica baja y puntuación de software alta, lo que refleja el significado semántico de la pregunta.
![alt text](imagenes/image-71.png)
El punto crítico: **la consulta y los fragmentos están en el mismo espacio vectorial**. Eso permite compararlos matemáticamente para encontrar cuál fragmento tiene el significado más cercano a la pregunta.

---

#### 30.5 Paso 5 — Encontrar los fragmentos más similares (similitud coseno)

La base de datos vectorial compara el vector de la consulta con todos los vectores almacenados y devuelve los más similares. La métrica que usa se llama **similitud coseno**: mide el coseno del ángulo entre dos vectores.

| Métrica | Rango | Interpretación |
| --- | --- | --- |
| Similitud coseno | -1 a 1 | Cercano a **1** = muy similares; cercano a **-1** = muy diferentes; **0** = sin relación |
| Distancia coseno | 0 a 2 | `1 - similitud coseno`. Cercano a **0** = muy similares; valores grandes = muy diferentes |

![alt text](imagenes/image-72.png)

En el ejemplo:

| Fragmento | Similitud coseno con la consulta | Interpretación |
| --- | --- | --- |
| Ingeniería de software | **0.983** | Muy alta similitud — este es el fragmento relevante |
| Investigación médica | 0.398 | Similitud baja — no es lo que el usuario busca |

La base de datos devuelve el fragmento de ingeniería de software, que es el semánticamente más cercano a la pregunta — aunque la pregunta no usa las palabras exactas del fragmento.

> La **distancia coseno** (1 - similitud) aparece frecuentemente en documentación de bases de datos vectoriales porque es más intuitiva para búsquedas: cuanto más cerca de 0, más similar. Los dos sistemas son equivalentes — solo es una convención de presentación.

![alt text](imagenes/image-73.png)

---

#### 30.6 Paso 6 — Construir el prompt final y enviar a Claude

Con el fragmento relevante recuperado, se construye el prompt combinando la pregunta original del usuario y el contexto encontrado:

```xml
Answer the user's question about the company report.

<user_question>
What did the software engineering department do this year?
</user_question>

<report>
## Section 2: Software Engineering
This division dedicated significant effort to studying various infection vectors
in our distributed systems.
</report>
```
![alt text](imagenes/image-74.png)

Claude recibe el fragmento relevante como contexto y puede responder con precisión — sin necesidad de haber visto el documento completo.

---

#### 30.7 Resumen del flujo completo

| Etapa | Cuándo ocurre | Qué hace |
| --- | --- | --- |
| **Preprocesamiento** | Una vez, al ingerir el documento | Segmentar → generar embeddings → almacenar en base de datos vectorial |
| **En tiempo real** | Cada vez que el usuario pregunta | Embeddear la consulta → buscar fragmentos similares → construir prompt → enviar a Claude |

La ventaja clave de este diseño: el preprocesamiento costoso ocurre una sola vez. Cada consulta del usuario solo requiere convertir una frase corta en embedding y hacer una búsqueda rápida — operaciones muy eficientes independientemente del tamaño del documento original.

---

#### Resumen rápido del flujo RAG

```text
① Segmentar    →  Dividir el documento en fragmentos manejables
② Embeddear    →  Convertir cada fragmento en un vector numérico (modelo de embeddings)
③ Almacenar    →  Guardar los vectores en una base de datos vectorial
                  ── hasta aquí: preprocesamiento (ocurre una sola vez) ──
④ Consultar    →  El usuario hace una pregunta
⑤ Embeddear    →  Convertir la pregunta en un vector (mismo modelo)
⑥ Buscar       →  Encontrar los fragmentos más similares (similitud coseno)
⑦ Prompt       →  Combinar pregunta + fragmentos relevantes y enviar a Claude
⑧ Respuesta    →  Claude genera la respuesta usando solo el contexto relevante
```

---

### Módulo 31: Implementación del flujo RAG

El módulo anterior mostró el flujo RAG conceptualmente. Este lo lleva a código real. Los cinco pasos de implementación siguen exactamente la misma lógica, pero ahora con clases y funciones concretas. El punto más importante a entender es que el sistema tiene **dos fases de tiempo distintas**: el preprocesamiento (que corre una sola vez) y la consulta (que corre cada vez que el usuario pregunta).

---

#### 31.1 Los cinco pasos de implementación

```text
Preprocesamiento (una vez):
  1. Dividir el documento en fragmentos
  2. Generar embeddings para cada fragmento
  3. Almacenar embeddings + texto original en un VectorIndex

En tiempo real (por cada consulta):
  4. Generar embedding de la pregunta del usuario
  5. Buscar los fragmentos más similares y construir el prompt
```

![alt text](imagenes/image-75.png)

---

#### 31.2 Paso 1 — Dividir el texto en fragmentos

```python
with open("./report.md", "r") as f:
    text = f.read()           # Lee el documento completo como string

# chunk_by_section divide por encabezados Markdown (## )
# Cada elemento de la lista es una sección completa del documento
chunks = chunk_by_section(text)

chunks[2]   # Inspeccionar un fragmento para verificar que la segmentación es correcta
```

---

#### 31.3 Paso 2 — Generar embeddings para todos los fragmentos

En lugar de llamar a `generate_embedding()` una vez por fragmento, la función se actualizó para aceptar una **lista** de textos y procesarlos en un solo llamado a la API — mucho más eficiente:

```python
# generate_embedding ahora acepta tanto un string individual como una lista de strings
# Pasar toda la lista de fragmentos de una vez reduce las llamadas a la API de VoyageAI
embeddings = generate_embedding(chunks, input_type="document")
# input_type="document" porque son fragmentos del corpus, no consultas del usuario
# Devuelve una lista de vectores, uno por cada fragmento en chunks
```

| Comportamiento | Llamadas individuales | Lista completa |
| --- | --- | --- |
| Llamadas a la API | Una por fragmento | Una sola |
| Velocidad | Lenta (N llamadas) | Rápida (1 llamada) |
| Uso recomendado | Prototipado | Producción |

---

#### 31.4 Paso 3 — Almacenar en el índice vectorial

`VectorIndex` es la base de datos vectorial. Cada entrada almacena **dos cosas juntas**: el vector numérico (para hacer la búsqueda) y el texto original (para poder devolverlo como contexto a Claude):

```python
store = VectorIndex()   # Crea el almacén vectorial en memoria

for embedding, chunk in zip(embeddings, chunks):
    # zip() empareja cada embedding con su fragmento de texto correspondiente
    # Se guarda el texto original en {"content": chunk} porque la búsqueda devuelve
    # ese dict — si no se guardara el texto, solo se recibirían números sin utilidad
    store.add_vector(embedding, {"content": chunk})
```

> **Por qué guardar el texto original es obligatorio:** cuando se hace una búsqueda, la base de datos devuelve los vectores más cercanos. Pero Claude necesita el texto, no los números. Si no se almacena el texto junto al vector, el resultado de la búsqueda es inútil para construir el prompt.

---

#### 31.5 Paso 4 — Embeddear la consulta del usuario

La pregunta del usuario pasa por el **mismo modelo de embeddings** que procesó los fragmentos. Esto es lo que hace posible la comparación matemática:

```python
# input_type="query" porque es una consulta del usuario, no un documento
# VoyageAI optimiza los vectores de forma diferente según el tipo
user_embedding = generate_embedding(
    "What did the software engineering dept do last year?",
    input_type="query"
)
```

Si se usaran modelos distintos para documentos y consultas, los vectores estarían en espacios matemáticos diferentes y la similitud coseno no tendría sentido. El mismo modelo garantiza que los vectores son comparables.

---

#### 31.6 Paso 5 — Buscar los fragmentos más relevantes

```python
# search() compara el embedding de la consulta contra todos los almacenados
# El segundo argumento (2) indica cuántos resultados devolver — los 2 más similares
results = store.search(user_embedding, 2)

for doc, distance in results:
    # distance es la distancia coseno: cuanto más cerca de 0, más similar
    # doc["content"] es el texto original del fragmento
    print(distance, "\n", doc["content"][0:200], "\n")
```

| Campo devuelto | Tipo | Descripción |
| --- | --- | --- |
| `doc` | `dict` | El diccionario que se pasó a `add_vector` — contiene `"content"` con el texto original |
| `distance` | `float` | Distancia coseno: `0` = idénticos, `2` = opuestos. Valores típicos relevantes: `< 0.75` |

---

#### 31.7 Comprender los resultados

Con la consulta *"What did the software engineering dept do last year?"*, los resultados son:

| Fragmento recuperado | Distancia coseno | Interpretación |
| --- | --- | --- |
| Sección 2: Ingeniería de Software | **0.71** | Más cercano — el más relevante |
| Sección de Metodología | 0.72 | Segundo más cercano |

![alt text](imagenes/image-76.png)

La distancia coseno más baja indica mayor similitud. La diferencia entre 0.71 y 0.72 parece pequeña, pero en búsqueda vectorial esos decimales importan — el sistema está correctamente priorizando el fragmento de ingeniería sobre el de metodología.

> **Conclusión central de RAG:** el sistema convierte texto en números (embeddings), almacena esos números eficientemente y usa similitud matemática para encontrar contenido relevante — sin depender de coincidencias exactas de palabras. Esta es la diferencia fundamental respecto a la búsqueda por palabras clave tradicional.

---

### Módulo 32: Búsqueda léxica BM25

La búsqueda semántica es poderosa, pero tiene un punto ciego específico: cuando el usuario busca un término exacto — un ID de incidente, un código de producto, un nombre técnico preciso — el sistema puede devolver secciones *conceptualmente relacionadas* que no contienen ese término. La solución no es reemplazar la búsqueda semántica, sino complementarla con **búsqueda léxica**, que opera sobre coincidencias exactas de palabras. La combinación de ambas se llama **búsqueda híbrida**.

---

#### 32.1 El problema de la búsqueda semántica sola

Supongamos que el usuario busca el incidente "INC-2023-Q4-011". La búsqueda semántica puede devolver:

| Fragmento devuelto | ¿Contiene el ID? | Por qué lo devolvió |
| --- | --- | --- |
| Sección de Ciberseguridad | ✅ Sí | Contiene el ID y es temáticamente relevante |
| Sección de Análisis Financiero | ❌ No | Es semánticamente cercana al contexto de incidentes, pero no tiene el término exacto |

El segundo resultado es un falso positivo: el sistema encontró algo *conceptualmente relacionado* pero que no responde realmente a la consulta. Para términos únicos y específicos, la coincidencia exacta de palabras es más útil que la similitud de significado.

---

#### 32.2 Estrategia de búsqueda híbrida

La solución es ejecutar **dos búsquedas en paralelo** y combinar los resultados:

| Tipo de búsqueda | Cómo encuentra los fragmentos | Ideal para |
| --- | --- | --- |
| **Semántica** (embeddings) | Similitud de significado en el espacio vectorial | Preguntas conceptuales, paráfrasis, consultas en lenguaje natural |
| **Léxica** (BM25) | Coincidencia exacta de términos con ponderación por rareza | IDs, códigos, nombres técnicos, términos poco frecuentes |
| **Híbrida** (combinada) | Fusión de ambas puntuaciones | La mayoría de los casos de uso reales |

Ningún método es universalmente superior — se complementan. Una pregunta como "¿cuáles fueron los problemas del Q4?" se responde mejor con búsqueda semántica; una búsqueda de "INC-2023-Q4-011" se responde mejor con búsqueda léxica.

---

#### 32.3 Cómo funciona BM25 — los cuatro pasos internos

**BM25 (Best Match 25)** es el algoritmo estándar para búsqueda léxica en sistemas RAG. Su nombre viene de que es la 25ª iteración de la familia de funciones de ranking BM. El proceso para cada consulta:

| Paso | Qué hace | Ejemplo |
| --- | --- | --- |
| **1. Tokenizar** | Divide la consulta en términos individuales | `"find INC-2023-Q4-011"` → `["find", "INC-2023-Q4-011"]` |
| **2. Contar frecuencia** | Cuenta cuántas veces aparece cada término en todos los fragmentos | `"find"` aparece 50 veces; `"INC-2023-Q4-011"` aparece 1 vez |
| **3. Ponderar por rareza** | Los términos raros reciben mayor peso; los comunes, menor | `"find"` → peso bajo; `"INC-2023-Q4-011"` → peso alto |
| **4. Rankear documentos** | Devuelve los fragmentos con más ocurrencias de los términos de mayor peso | Prioriza el fragmento que contiene el ID específico |

La ponderación por rareza es la clave de BM25: una palabra que aparece en casi todos los documentos (como "el", "de", "año") aporta muy poco para distinguir un fragmento relevante de uno irrelevante. Una palabra que aparece solo una vez en todo el corpus casi siempre señala el fragmento correcto.

---

#### 32.4 Implementación de BM25

La interfaz de `BM25Index` es deliberadamente similar a `VectorIndex` para que los dos sistemas sean intercambiables. La diferencia interna es que BM25 no necesita embeddings — trabaja directamente sobre el texto:

```python
# Paso 1: segmentar el documento (igual que en la búsqueda semántica)
chunks = chunk_by_section(text)

# Paso 2: crear el índice BM25 y agregar los fragmentos
store = BM25Index()
for chunk in chunks:
    # A diferencia de VectorIndex, no se necesita generar embeddings primero
    # BM25Index indexa el texto directamente y construye su estructura interna
    store.add_document({"content": chunk})

# Paso 3: buscar — la consulta va como texto, no como embedding
results = store.search("What happened with INC-2023-Q4-011?", 3)

for doc, distance in results:
    # En BM25, "distance" es una puntuación de relevancia (mayor = más relevante)
    # Convención opuesta a la distancia coseno donde menor = más relevante
    print(distance, "\n", doc["content"][:200], "\n----\n")
```

| Diferencia | `VectorIndex` (semántico) | `BM25Index` (léxico) |
| --- | --- | --- |
| Entrada para indexar | Embedding (lista de floats) | Texto directamente |
| Entrada para buscar | Embedding de la consulta | Texto de la consulta |
| Llamadas a API externa | Sí (VoyageAI) | No — el algoritmo corre localmente |
| Métrica de resultado | Distancia coseno (menor = mejor) | Puntuación BM25 (mayor = mejor) |

---

#### 32.5 Por qué BM25 supera a la búsqueda semántica para términos específicos

Con la consulta "What happened with INC-2023-Q4-011?", los resultados de BM25 priorizan correctamente las secciones que **contienen ese ID exacto** — no solo las que tratan sobre incidentes en general.

| Situación | Búsqueda recomendada |
| --- | --- |
| IDs, códigos, identificadores únicos | **BM25** — necesita coincidencia exacta |
| Nombres técnicos precisos (nombres de funciones, errores) | **BM25** |
| Preguntas en lenguaje natural ("¿qué hizo el equipo este año?") | **Semántica** |
| Preguntas conceptuales o con sinónimos | **Semántica** |
| La mayoría de los casos reales | **Híbrida** — combina ambas |

> La búsqueda semántica entiende el *significado*; BM25 garantiza que no se pierda ninguna *coincidencia exacta*. Usarlas juntas cubre las debilidades de cada una.

---

### Módulo 33: Una canalización RAG de índice múltiple

Los módulos anteriores desarrollaron `VectorIndex` y `BM25Index` como sistemas separados. Este módulo los unifica en un único componente llamado `Retriever`. El desafío de combinarlos no es técnico — es matemático: cada sistema usa una métrica de puntuación diferente, así que no se pueden sumar o comparar directamente. La solución es **Reciprocal Rank Fusion (RRF)**: un algoritmo que normaliza los rankings de ambos sistemas usando solo la *posición* de cada resultado, ignorando la puntuación absoluta.

---

#### 33.1 La arquitectura del Retriever

`VectorIndex` y `BM25Index` comparten la misma interfaz (`add_document()` y `search()`). Esta decisión de diseño es lo que hace posible unificarlos sin acoplamiento fuerte: el `Retriever` no sabe ni le importa cómo funciona cada índice internamente — solo sabe que ambos responden a los mismos mensajes.

```python
class Retriever:
    def __init__(self, *indexes: SearchIndex):
        if len(indexes) == 0:
            raise ValueError("At least one index must be provided")
        # Acepta cualquier cantidad de índices — VectorIndex, BM25Index, o cualquier
        # implementación futura que respete la interfaz SearchIndex
        self._indexes = list(indexes)

    def add_document(self, document: Dict[str, Any]):
        # Reenvía cada documento a TODOS los índices registrados
        # Así cada índice construye su propia estructura interna (vectores o tabla BM25)
        for index in self._indexes:
            index.add_document(document)

    def search(self, query_text: str, k: int = 1, k_rrf: int = 60):
        # Consulta cada índice por separado y recopila sus listas de resultados rankeados
        # Luego aplica RRF para fusionarlos en una sola lista ordenada
        all_results = [index.search(query_text, k) for index in self._indexes]
        return self._reciprocal_rank_fusion(all_results, k_rrf)
```

| Parámetro de `search` | Descripción |
| --- | --- |
| `query_text` | La consulta del usuario en texto plano |
| `k` | Cuántos resultados pedir a cada índice individual |
| `k_rrf` | Constante de suavizado de RRF. Valor estándar: `60`. Valores más bajos amplifican la diferencia entre posiciones |

---

#### 33.2 Reciprocal Rank Fusion — cómo combinar resultados incomparables

El problema central: `VectorIndex` devuelve distancias coseno (0.71, 0.72…) y `BM25Index` devuelve puntuaciones de relevancia (12.3, 8.7…). Estos números viven en escalas completamente distintas — no se pueden sumar ni comparar directamente.

RRF resuelve esto ignorando los valores absolutos y usando solo la **posición** (rank) de cada resultado en cada lista. La fórmula es:

```text
RRF_score(d) = Σ  1 / (k + rank_i(d))
```

Donde `rank_i(d)` es la posición del fragmento `d` en la lista del índice `i` (empezando en 1), y `k` es una constante de suavizado.

**Ejemplo concreto** con `k = 1` para ver el efecto claramente:

| Fragmento | Rank en VectorIndex | Rank en BM25Index | Cálculo RRF | Score final |
| --- | --- | --- | --- | --- |
| Sección 2 | 1° | 2° | 1/(1+1) + 1/(1+2) | **0.833** |
| Sección 6 | 3° | 1° | 1/(1+3) + 1/(1+1) | **0.750** |
| Sección 7 | 2° | 3° | 1/(1+2) + 1/(1+3) | **0.583** |

Resultado final: Sección 2 > Sección 6 > Sección 7.

La Sección 2 gana porque estuvo en el top de *ambos* sistemas — aunque no fue primera en ninguno. La Sección 6 quedó segunda porque fue primera en BM25 pero última en vectores. La intuición es correcta: **un fragmento que ambos sistemas consideran relevante merece más confianza que uno que solo destaca en un sistema**.

> Con `k = 60` (valor estándar), las diferencias entre posiciones se suavizan más — un fragmento en posición 1 y otro en posición 5 quedan más cercanos en el score final. Con `k = 1`, la diferencia entre posiciones es más pronunciada.

---

#### 33.3 Resultado: búsqueda híbrida en acción

Con la consulta `"¿qué pasó con INC-2023-Q4-011?"`, los resultados del `Retriever` híbrido mejoran claramente sobre cada sistema solo:

| # | Fragmento | ¿Por qué aparece aquí? |
| --- | --- | --- |
| 1 | Sección 10: Análisis de Ciberseguridad | Contiene el ID exacto (BM25) y es temáticamente relevante (semántica) |
| 2 | Sección 2: Ingeniería de Software | Relevante semánticamente y mencionada en contexto del incidente |
| 3 | Sección 5: Novedades Legales | Relevancia semántica secundaria |

Comparado con el resultado de solo vectores que devolvía el análisis financiero como segundo resultado (un falso positivo), el sistema híbrido produce resultados más precisos porque BM25 penaliza los fragmentos que no contienen el ID exacto.

---

#### 33.4 Extensibilidad — la ventaja del diseño por interfaz

Porque todos los índices implementan la misma interfaz `SearchIndex` (`add_document` + `search`), agregar un nuevo tipo de búsqueda no requiere modificar el `Retriever`:

| Tipo de índice | Cómo funciona | Caso de uso ideal |
| --- | --- | --- |
| `VectorIndex` | Similitud coseno sobre embeddings | Consultas semánticas y conceptuales |
| `BM25Index` | Frecuencia inversa de términos | IDs, códigos, términos técnicos exactos |
| Índice de grafos *(futuro)* | Relaciones entre entidades | Consultas sobre conexiones entre personas/eventos |
| Índice especializado *(futuro)* | Dominio específico | Búsqueda médica, legal, financiera con vocabulario propio |

La clave del diseño modular es que cada implementación puede desarrollarse, probarse y optimizarse de forma independiente. El `Retriever` solo necesita que respeten la interfaz — el resto es transparente para él.

---

#### 33.5 Resumen de la Unidad 6 — RAG completo

```text
Unidad 6: RAG y búsqueda agencial

Módulo 27  →  Qué es RAG y cuándo usarlo
Módulo 28  →  Cómo segmentar documentos (tamaño, estructura, oraciones, semántica)
Módulo 29  →  Qué son los embeddings y cómo generarlos con VoyageAI
Módulo 30  →  El flujo completo paso a paso (con similitud coseno y normalización)
Módulo 31  →  Implementación real con VectorIndex
Módulo 32  →  Búsqueda léxica BM25 para términos exactos
Módulo 33  →  Retriever híbrido con Reciprocal Rank Fusion

Sistema final = Segmentación + Embeddings + VectorIndex + BM25Index + RRF
```
