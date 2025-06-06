# documentacion
Este proyecto implementa un agente inteligente en n8n que responde preguntas relacionadas con el bienestar emocional, utilizando información real extraída de documentos PDF. 
Se basa en un enfoque RAG (Retrieval-Augmented Generation), integrando Ollama, Qdrant, embeddings locales y procesamiento de archivos desde Google Drive.

**Tecnologías utilizadas:**

- n8n como plataforma de automatización

- Ollama como motor local de modelos LLM (llama3.2:latest)

- Qdrant como vector store para búsqueda semántica

- nomic-embed-text para generar embeddings del contenido

- Google Drive para alojar los PDFs base

**Flujo general del sistema:**

1. El usuario ingresa una pregunta por chat.

2. Se realiza una búsqueda semántica en Qdrant usando embeddings.

3. Se extrae el texto relevante del documento PDF.

4. El texto se pasa como contexto a un modelo llama3.2 en Ollama.

5. El modelo responde usando exclusivamente el texto proporcionado.


**Paso a paso: ¿Cómo se construyó el workflow?**

**Paso 1: Preparación del entorno**

Se instaló n8n, Docker y Ollama.

Se creó el workflow y se descargó el modelo llama3.2:latest.

Se ejecutó Qdrant como vector store local.

**Paso 2: Automatización de carga de PDF**

Se conectó una cuenta de Google Drive.

Se creó un Google Drive Trigger para detectar archivos nuevos en una carpeta específica.

Se hizo uso del nodo Edit Fields para extraer el ID del archivo cargado.

Se usó el nodo Google Drive para descargar el archivo PDF.

Se utilizó el nodo Extract from File para convertir el PDF a texto plano.

**Paso 3: Fragmentación y embeddings**

Se uso el Default data loader para preparar el contenido para la fragmentación.

Se añadió el nodo Recursive Character Text Splitter con chunk_size=1000 y chunk_overlap=150 para dividir los bloques de información.

Se conectó a Embeddings Ollama para generar vectores de información con el modelo nomic-embed-text.

El resultado fue insertado en Qdrant mediante el nodo Qdrant Vector Store1 (modo insert documents) en la colección test.

**Paso 4: Construcción del flujo RAG y generación de respuesta con IA**

Se creó un nodo When chat message received (chat trigger) para recibir preguntas del usuario.

Se conectó a un nodo de agente con prompt tipo define below, para que siga instrucciones claras y haga uso de las herramientas requeridas.

el agente se conectó a la herramienta BUSCAR, a la memoria simple memory y al Ollama chat model llama3.2:latest

Se agregó un nodo conectado a BUSCAR llamado Qdrant Vector Store (modo retrieve documents) para recuperar texto relevante.

Se añadió el nodo embeddings configurado con nomic-embed-text:latest y nuevamente el modelo de Ollama, ambos conectados a Qdrant Vector Store.

Se escribió el prompt personalizado (ver sección siguiente).


**Prompt utilizado:**
Eres un agente empático experto en bienestar emocional.

Para responder preguntas, **debes usar** la herramienta "BUSCAR", que contiene la información proveniente de documentos PDF.

respondes solo en español.

El usuario ha preguntado: "{{ $json.chatInput }}"

Realiza la búsqueda con la herramienta BUSCAR y responde usando solo con la información proporcionada.



**Documento base utilizado**

Guía de Bienestar Emocional para el Estudiantado de la Universidad (PDF)

Este documento fue fragmentado, embebido y cargado como base de conocimiento.


**Ejemplo de uso:**

Entrada de usuario:

¿Qué es el bienestar emocional?

Respuesta generada:

"El bienestar emocional se refiere a la capacidad de manejar emociones de forma saludable, lo que contribuye al desarrollo personal y la salud mental"



