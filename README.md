# Documentación
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

Se instaló n8n, al clonar el repositorio de Github https://github.com/n8n-io/self-hosted-ai-starter-kit en el equipo.
Se instaló Docker Desktop, la cual es una herramienta que ejecuta aplicaciones utilizando contenedores que almacenan todo lo que esta necesita para poder funcionar.
Y se instaló Ollama, una plataforma que permite correr modelos de lenguaje de manera local sin depender de servicios externos, sirve para descargar y ejecutar modelos (ej: llama3, mistral) y embeddings.

Una vez clonado y configurado el entorno de n8n, se creo una cuenta para poder ingresar, se creó el workflow donde se pondrán los nodos más adelante y se descargó el modelo a utilizar llama3.2:latest desde el contenedor Ollama en Docker desktop, este modelo sirve para procesar entradas y generar salidas inteligentes basado en lo que ha aprendido.

Se añadió el nodo de Qdrant, una base de datos vectorial, que está especialmente diseñada para trabajar con inteligencia artificial y busquedas semánticas, esta sirve para guardar la información sacada de los documentos en forma de vectores según su contexto y así poder dar una respuesta acertada. Se eligió debido a su potencia y facilidad. Para esto también se hace uso de otro nodo conectado a Qdrant llamado embedding, en este caso se utilizó el nomic-embed-text:latest de Ollama debido a su dimensionalidad para capturar detalles y compatibilidad con el modelo de lenguaje.

**Paso 2: Automatización de carga de PDF**

Se creó  un nodo de Google Drive Trigger para detectar archivos nuevos en una carpeta específica, para esto también se conectó a una cuenta de google drive que tenga la carpeta con los archivos a utilizar, para así poder acceder a ellos.
.
Luego se agregó el nodo Edit Fields, que en n8n se utiliza para modificar, renombrar o reestructurar campos de datos manejados en el flujo, en este caso se usó para extraer el ID del archivo cargado que viene desde el Google Drive Trigger y así poder pasarselo al siguiente nodo y que se mantenga la claridad y compatibilidad.

Después se creo el nodo Google Drive para descargar el archivo PDF y se conectó al nodo de Edit Fields.

Se creó el nodo Extract from File que sirve para convertir el PDF a texto plano, es decir sacar todo el texto dentro del archivo y se conectó al nodo de Google Drive anterior.

**Paso 3: Fragmentación y embeddings**


El resultado  de Extract from file luego fue insertado en un nodo de Qdrant Vector Store1 (modo insert documents) para guardar texto embebido en la colección "test" y luego poder buscarlo por su significado.

Se creó el nodo Default data loader y se conectó a Qdrant Vector Store1 para preparar el contenido de manera estructurada para la fragmentación.

Se añadió el nodo Recursive Character Text Splitter con chunk_size=1000 y chunk_overlap=150 para dividir los bloques de información y se conectó al Default data loader.

Se conectó también Qdrant Vector Store1 a Embeddings Ollama para generar vectores de información con el modelo nomic-embed-text.


**Paso 4: Construcción del flujo RAG y generación de respuesta con IA**

Se creó un nodo When chat message received (chat trigger) para activar el flujo al recibir preguntas del usuario.

Se conectó a un nodo de agente con prompt tipo define below, para que siga instrucciones claras y haga uso de las herramientas requeridas.

el agente se conectó a la herramienta BUSCAR, a la memoria simple memory (que guarda mensajes de conversaciones anteriores para mantener el contexto) y al Ollama chat model llama3.2:latest.

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



