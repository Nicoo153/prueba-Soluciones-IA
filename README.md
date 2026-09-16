# EP1 - Diseño de Solución con LLM y RAG: Clínica Vitalis

Asistente virtual basado en RAG (Retrieval-Augmented Generation) que responde
consultas administrativas de pacientes de Clínica Vitalis (horarios de
atención, preparación de exámenes y convenios de salud), evitando
alucinaciones y bloqueando cualquier intento de diagnóstico médico.

Proyecto desarrollado para la Evaluación Parcial N°1 de la asignatura
ISY0101 - Ingeniería de Soluciones con IA (Duoc UC).

## Arquitectura

El pipeline combina un flujo de ingesta (offline) y un flujo de consulta
(en tiempo real):

1. **Ingesta:** el documento base (`conocimiento_vitalis.txt`) se carga con
   `TextLoader`, se fragmenta con `RecursiveCharacterTextSplitter`
   (`chunk_size=400`, `chunk_overlap=50`) y se vectoriza con
   `HuggingFaceEmbeddings` (modelo local `all-MiniLM-L6-v2`), almacenando
   los vectores en una base `ChromaDB` (colección `vitalis_local_v1`).
2. **Consulta:** el `retriever` recupera los `k=2` fragmentos más
   relevantes para la pregunta del usuario, que se inyectan junto con la
   pregunta en un `System Prompt` con reglas estrictas. El flujo se
   orquesta con LangChain Expression Language (LCEL) y el modelo generador
   es Google Gemini (`gemini-3.6-flash`), con trazabilidad registrada en
   LangSmith.

## Requisitos

- Cuenta de Google con acceso a [Google Colab](https://colab.research.google.com/)
- Una API key de Google Gemini, generada en [Google AI Studio](https://aistudio.google.com/apikey)
  (crear la key en un proyecto de Google Cloud propio, no compartido)
- Una API key de [LangSmith](https://smith.langchain.com/) (opcional, solo
  para trazabilidad; el proyecto funciona sin ella si se elimina esa
  configuración)

## Cómo ejecutar el proyecto

1. Abre el notebook `Proyecto_RAG_Vitalis.ipynb` en Google Colab.
2. En el panel de secretos de Colab (ícono 🔑), agrega:
   - `GOOGLE_API_KEY`: tu clave de Google AI Studio
   - `LANGSMITH_API_KEY`: tu clave de LangSmith (opcional)
   Activa el acceso del notebook a ambos secrets.
3. Ejecuta las celdas en orden, de arriba hacia abajo (Entorno de
   ejecución → Ejecutar todas).
4. La celda de instalación de dependencias puede tardar unos minutos la
   primera vez.
5. Las últimas celdas ejecutan dos pruebas automáticas:
   - Una consulta dentro del contexto disponible (ayuno para examen).
   - Un intento de obtener un diagnóstico médico, que debe ser rechazado
     por el asistente.

## Notas técnicas

- Los embeddings se generan localmente (sin costo ni límite de cuota),
  por lo que solo se necesita API key para el modelo generador (Gemini).
- Si `rag_chain.invoke(...)` falla con un error de modelo no encontrado,
  verifica que el nombre del modelo en `ChatGoogleGenerativeAI` siga
  vigente (la lista de modelos disponibles se puede consultar con
  `client.models.list()` en el propio notebook).

## Declaración de uso de IA

El uso de herramientas de IA durante el desarrollo de este proyecto está
declarado en la sección 7 del informe técnico (`Informe_Técnico_-_Clínica_Vitalis.docx`).

## Autor

Nicolás Rain — Sección 009D — Duoc UC
