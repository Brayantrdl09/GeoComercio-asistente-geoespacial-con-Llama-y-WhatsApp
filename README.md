# GeoComercio

**Asistente geoespacial por WhatsApp para consultar y comparar establecimientos de la Ciudad de México.**

Built with Llama.

GeoComercio combina datos del Directorio Estadístico Nacional de Unidades Económicas (DENUE), consultas en SQLite y un modelo de lenguaje para responder preguntas sobre establecimientos cercanos. También consulta el documento metodológico del INEGI y conserva el contexto de cada conversación.

El proyecto se ejecuta en un notebook de Google Colab. WhatsApp funciona como interfaz; FastAPI recibe los mensajes, ngrok proporciona la conexión pública y Ollama ejecuta los modelos en la máquina de Colab.

## Archivos

| Archivo | Contenido |
| --- | --- |
| [GeoComercio.ipynb](GeoComercio.ipynb) | Notebook completo del proyecto. |
| [GUIA_CONFIGURACION.md](GUIA_CONFIGURACION.md) | Cuentas, permisos, secretos de Colab, conexión con Meta y primeros mensajes. |
| README.md | Descripción, funcionamiento, alcance y uso del proyecto. |

## Qué permite hacer

- Buscar establecimientos por giro, coordenadas y radio.
- Comparar ubicaciones por cantidad de registros, densidad y cercanía.
- Guardar comparaciones con folio y recuperarlas después.
- Continuar una consulta conservando el contexto de la conversación.
- Consultar conceptos y cobertura del DENUE mediante recuperación de fragmentos del PDF metodológico (RAG).
- Recibir texto y ubicaciones compartidas desde WhatsApp.
- Revisar las herramientas ejecutadas, las decisiones de los filtros y los errores.

## Giros disponibles

| Nombre de consulta | Clase SCIAN | Alcance |
| --- | --- | --- |
| Lavanderías | 812210 | Lavanderías y tintorerías. |
| Ferreterías | 467111 | Comercio al por menor en ferreterías y tlapalerías. |
| Cafeterías | 722515 | Cafeterías, fuentes de sodas, neverías, refresquerías y similares. |

La búsqueda usa la clasificación económica del registro. La clase de cafeterías incluye otros negocios similares y no equivale a una lista exclusiva de cafeterías.

## Cómo funciona

1. WhatsApp entrega el mensaje al webhook público de ngrok y FastAPI.
2. El backend registra el mensaje y lo procesa en una cola de un trabajador.
3. Para las consultas al modelo, Prompt Guard revisa posibles instrucciones maliciosas y Llama Guard revisa el contenido de entrada.
4. Llama, mediante Llama Stack, solicita las herramientas que necesita. Python valida sus argumentos antes de ejecutarlas.
5. Las herramientas consultan SQLite o recuperan fragmentos del PDF metodológico.
6. Llama redacta la respuesta y Llama Guard la revisa antes de enviarla mediante la API de Meta.

Los saludos y la bienvenida fija se gestionan con reglas del backend. Los cálculos geográficos se realizan en Python; el modelo no calcula los conteos por su cuenta.

## Componentes

| Componente | Implementación |
| --- | --- |
| Entorno | Google Colab, Python 3.12 o superior. |
| Modelo conversacional | `llama3.1:8b`, ejecutado con Ollama. |
| Conversaciones y herramientas | Llama Stack 0.7.3; cliente 0.7.5. |
| Datos y análisis | SQLite, pandas y NumPy. |
| Distancias | Haversine y filtro por radio. |
| Embeddings | `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`. |
| Detección de instrucciones maliciosas | `meta-llama/Llama-Prompt-Guard-2-86M`. |
| Clasificación de contenido | `llama-guard3:8b`, ejecutado con Ollama. |
| Backend HTTP | FastAPI y Uvicorn. |
| Conexión pública | ngrok, administrado desde Python con pyngrok. |
| Mensajería | WhatsApp Cloud API de Meta. |

El apartado 0 prepara las dependencias y un entorno separado para el servidor Llama Stack. Las versiones fijadas se encuentran en `REQUISITOS_NOTEBOOK_GEO` y `REQUISITOS_STACK_GEO` dentro del notebook.

## Inicio rápido

1. Sigue la [guía de configuración](GUIA_CONFIGURACION.md): incluye la solicitud de acceso a Prompt Guard en Hugging Face.
2. Abre el notebook en Colab y selecciona una GPU disponible antes de ejecutar.
3. Guarda los cuatro secretos: `huggin`, `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID` y `NGROK_AUTHTOKEN`.
4. Ejecuta los apartados en orden hasta preparar los filtros.
5. Configura tu número de prueba con el bloque del apartado 7 de la guía y continúa con WhatsApp.
6. Registra en Meta la URL `/webhook` que imprime el notebook y el mismo token de verificación.
7. Suscribe el campo `messages` y envía un mensaje nuevo desde tu teléfono autorizado.

Usa los números de los **encabezados del notebook**, no el número de ejecución que aparece junto a una celda. La celda titulada **“Borrar el historial de conversacion en llama stack”** es una acción manual: omítela durante el arranque normal.

## Ejemplos de uso

**Presentación del asistente**

> Hola, ¿qué puedes hacer?

**Búsqueda por radio**

> Busca cafeterías a 700 metros de latitud 19.4983 y longitud -99.1449.

**Continuación con memoria**

> ¿Y en el mismo punto a 500 metros?

**Comparación**

> Compara lavanderías a 1 km en A (19.3501, -99.1621) y B (19.3860, -99.1645).

**Recuperación**

> Muéstrame la comparación del folio 3.

Sustituye el folio por uno que el asistente haya generado para tu usuario.

**Consulta documental**

> ¿Cómo se determina el tamaño de un establecimiento en el DENUE? Consulta la metodología y cita la página.

También puedes compartir una ubicación de WhatsApp e indicar el giro y el radio en un mensaje. Identifica las coordenadas como **latitud y longitud**; si provienen de un sistema X/Y, X suele corresponder a longitud y Y a latitud en coordenadas geográficas.

## Organización del notebook

| Apartados | Trabajo |
| --- | --- |
| 0 | Dependencias y entorno separado de Llama Stack. |
| 1–10 | Descarga DENUE, validación, SQLite y análisis geográfico. |
| 11–20 | Ollama, Llama Stack, herramientas y memoria. |
| 21–28 | Descarga del PDF, embeddings y consulta documental. |
| 29–34 | Llama Guard 8B, Prompt Guard y flujo con filtros. |
| 35–40 | Credenciales, procesamiento, webhook, ngrok y diagnóstico de WhatsApp. |
| 41 | Definición de la función para detener WhatsApp. |

## Datos y persistencia

La fuente es el [DENUE del INEGI](https://www.inegi.org.mx/app/mapa/denue/). El notebook descarga el [CSV de CDMX](https://www.inegi.org.mx/contenidos/masiva/denue/denue_09_csv.zip) y el [documento metodológico](https://www.inegi.org.mx/contenidos/temas/directorio/doc/metodologia.pdf).

La edición utilizada en las pruebas del proyecto fue **05/2026**. Las URL pueden apuntar a ediciones posteriores: consulta los metadatos registrados por tu ejecución. El proyecto guarda fecha de descarga y SHA-256 para identificar la fuente utilizada.

En `geocomercio_datos/` se guardan los archivos descargados, `geocomercio.sqlite`, `whatsapp.sqlite`, registros de servicio y el almacenamiento de Llama Stack. Los modelos también ocupan espacio en las cachés de la máquina de Colab.

Guardar el notebook no respalda automáticamente estas bases ni los modelos. Para conservar el estado, detén la actividad y respalda los datos y el almacenamiento de Stack antes de eliminar la máquina de Colab. El estado no queda guardado en una cuenta de Llama por el simple hecho de usar el modelo.

## Diagnóstico

```python
# Mensajes recibidos por WhatsApp y errores de envío.
ver_whatsapp_geo(limite=10)

# Usa el identificador del remitente recibido desde Meta.
USUARIO_WA = f"wa:{numero_recibido}"
ver_eventos_geo(USUARIO_WA, limite=10)
ver_historial_geo(USUARIO_WA)
```

Para comenzar otra conversación sin borrar análisis ni eventos:

```python
nueva_conversacion_geo(USUARIO_WA)
```

Para detener el túnel y la cola, después de definir la función del apartado 41:

```python
detener_whatsapp_geo()
```

## Alcance y límites

- Solo se analizan las tres clases cargadas de CDMX. Un radio que cruce su límite puede tener cobertura incompleta.
- Las distancias son geográficas, no recorridos ni tiempos de traslado.
- Los registros no confirman apertura actual ni horarios.
- Los conteos no permiten garantizar rentabilidad: faltan demanda, rentas, ventas y otros datos.
- El RAG recupera candidatos; una similitud alta no garantiza una respuesta sustentada.
- Los filtros pueden producir falsos positivos y falsos negativos. No verifican la exactitud de las respuestas.
- La cola procesa un mensaje a la vez. Cargar modelos o usar CPU puede aumentar la espera.
- Esta implementación es una demostración en Colab. El webhook verifica el token de alta, pero su receptor POST actual no valida la firma de Meta con App Secret. Un despliegue público permanente requiere completar esa autenticación y la gestión de credenciales y almacenamiento.

## Cuentas y costos

El notebook no utiliza una API comercial para inferencia de Llama, Google Maps, OpenAI ni una clave de la API DENUE: descarga archivos públicos y ejecuta los modelos en Colab. Hugging Face se utiliza para descargar modelos.

Colab, ngrok y el entorno de prueba de WhatsApp tienen condiciones y límites propios. La disponibilidad de GPU y la continuidad de una sesión no están garantizadas. Consulta los planes vigentes antes de pasar de la demostración a uso continuo; no se ofrece una garantía de operación ilimitada sin costo.

## Referencias

- [Configuración de WhatsApp Cloud API](https://developers.facebook.com/documentation/business-messaging/whatsapp/get-started).
- [Webhooks de WhatsApp](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/overview).
- [Ollama: Llama 3.1](https://ollama.com/library/llama3.1:8b) y [Llama Guard 3 8B](https://ollama.com/library/llama-guard3:8b).
- [Prompt Guard 2 86M](https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M).
- [Modelo de embeddings](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2).
- [Llama Stack](https://github.com/llamastack/llama-stack).
- [Documentación de ngrok](https://ngrok.com/docs/start) y [preguntas frecuentes de Colab](https://research.google.com/colaboratory/faq.html).

Los datos y modelos conservan las licencias y condiciones de sus proveedores. Antes de distribuir el proyecto, define la licencia del código propio y conserva las atribuciones correspondientes.
