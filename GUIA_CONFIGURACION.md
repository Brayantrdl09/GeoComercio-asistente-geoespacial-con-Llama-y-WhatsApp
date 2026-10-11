# Configuración de GeoComercio

Guía para ejecutar [GeoComercio.ipynb](GeoComercio.ipynb) en Google Colab y conversar desde WhatsApp. Fecha de preparación: 9 de octubre de 2026.

Las secciones se refieren a los encabezados del notebook. Los nombres de menús externos pueden variar según el idioma y la cuenta.

## 1. Qué necesitas obtener

| Recurso | Dónde se obtiene | Dónde se usa |
| --- | --- | --- |
| Cuenta de Google | [Google Colab](https://colab.research.google.com/) | Ejecutar el notebook. |
| Acceso a Prompt Guard | [Ficha del modelo en Hugging Face](https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M) | Autorizar la descarga del modelo restringido. |
| Token de Hugging Face | [Access Tokens](https://huggingface.co/settings/tokens) | Secreto `huggin` en Colab. |
| Token de acceso de WhatsApp | [Meta for Developers: aplicaciones](https://developers.facebook.com/apps/) | Secreto `WHATSAPP_ACCESS_TOKEN`. |
| Phone Number ID | Panel de configuración de API de WhatsApp en tu aplicación de Meta | Secreto `WHATSAPP_PHONE_NUMBER_ID`. |
| Authtoken de ngrok | [Panel de ngrok](https://dashboard.ngrok.com/get-started/your-authtoken) | Secreto `NGROK_AUTHTOKEN`. |
| Número personal autorizado para pruebas | Panel de WhatsApp en Meta | Variable `NUMERO_META_GEO` entre los apartados 36 y 37. |
| Verify Token | Lo defines tú en `VERIFY_TOKEN_WA` | Mismo valor en el notebook y en el formulario del webhook de Meta. |
| Callback URL | La genera el apartado 39 | Formulario del webhook de Meta. |

**Se necesitan cuatro secretos de Colab.** El Phone Number ID es un identificador; no es una contraseña ni tu número personal. El token de ngrok debe ser un **Authtoken**, no una API Key.

## 2. Hugging Face: solicitar acceso al modelo

**Sí: cada persona que reproduzca el proyecto necesita acceso al repositorio de Prompt Guard y un token de su propia cuenta autorizada.** Compartir el notebook no comparte ese permiso.

1. Crea una cuenta o inicia sesión en [Hugging Face](https://huggingface.co/join).
2. Abre [meta-llama/Llama-Prompt-Guard-2-86M](https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M).
3. Completa el formulario de acceso y acepta las condiciones si estás de acuerdo. Usa datos reales. En afiliación, indica tu institución u organización real; si trabajas de forma independiente, indícalo según permita el formulario. La ficha solicita nombres completos y evitar siglas.
4. Envía la solicitud. Continúa cuando el repositorio indique que tienes acceso; el tiempo de aprobación no está garantizado.
5. En esa misma cuenta, abre [Settings → Access Tokens](https://huggingface.co/settings/tokens) y crea un token de lectura. Si usas permisos finos, habilita la lectura de este repositorio restringido al que ya tienes acceso. No se necesita permiso de escritura para descargarlo.
6. Copia el token en el secreto `huggin` de Colab.

Tener un token no reemplaza la aprobación del modelo. El notebook también admite el nombre `huggis` como alternativa, pero para una instalación nueva utiliza `huggin`.

Fuentes: [acceso a modelos restringidos](https://huggingface.co/docs/hub/en/models-gated) y [permisos de tokens](https://huggingface.co/docs/hub/en/security-tokens).

### Qué modelo se descarga de cada lugar

| Modelo | Descarga en esta implementación | Autenticación usada |
| --- | --- | --- |
| `llama3.1:8b` | Ollama, apartado 12 | No usa el token de Hugging Face. |
| `llama-guard3:8b` | Ollama, apartado 29 | No usa el token de Hugging Face. |
| `meta-llama/Llama-Prompt-Guard-2-86M` | Hugging Face, apartado 30 | Cuenta autorizada y token de lectura. |
| `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` | Hugging Face, apartado 22 | Repositorio público; sin solicitud de acceso en este flujo. |

El token permite descargar Prompt Guard. Las revisiones se ejecutan en la máquina de Colab; no se contrata un endpoint de inferencia de Hugging Face.

## 3. Meta: preparar WhatsApp de prueba

1. Entra a [Meta for Developers](https://developers.facebook.com/apps/) y completa el registro de desarrollador si se solicita.
2. Crea o selecciona una aplicación y habilita el caso de uso/producto de WhatsApp. Completa la vinculación del portafolio comercial que solicite el asistente de Meta.
3. Abre **WhatsApp → API Setup / Configuración de API**. Identifica el número empresarial de prueba asignado por Meta.
4. En destinatarios de prueba, agrega tu teléfono personal y completa su verificación.
5. Usa el botón de envío del panel para comprobar que recibes el mensaje de prueba.
6. Copia el **Access Token** del panel y el **Phone Number ID** correspondiente al número emisor de prueba.
7. Guárdalos en Colab con los nombres exactos de la tabla del apartado 5.

Referencia: [inicio de WhatsApp Cloud API](https://developers.facebook.com/documentation/business-messaging/whatsapp/get-started).

### Identificadores que debes distinguir

| Dato | Uso |
| --- | --- |
| Número de prueba de Meta | Es el teléfono del bot; a este número escribes desde tu WhatsApp personal. |
| Tu teléfono autorizado | Recibe las respuestas durante las pruebas. |
| Phone Number ID | Identifica al teléfono emisor en las llamadas HTTP. Se guarda en `WHATSAPP_PHONE_NUMBER_ID`. |
| WABA ID / identificador de cuenta empresarial | Identifica la cuenta de WhatsApp Business; no sustituye al Phone Number ID. |
| App ID | Identifica la aplicación; no sustituye al Phone Number ID. |

### Token de acceso, Verify Token y App Secret

| Valor | Quién lo proporciona | Situación en este notebook |
| --- | --- | --- |
| Access Token | Meta | Obligatorio para consultar el emisor y enviar mensajes. |
| Verify Token | Lo defines tú | Obligatorio para verificar la URL del webhook; debe coincidir en ambos lugares. |
| App Secret | Meta | No se utiliza en el receptor POST de esta demostración. |

El notebook define `VERIFY_TOKEN_WA = "geocomercio_prueba_2026"`. Copia su valor exacto al registrar el webhook. No lo sustituyas por el Access Token.

El token de acceso del panel es para pruebas y puede caducar. Si Meta rechaza su validez, actualiza `WHATSAPP_ACCESS_TOKEN` y repite el apartado 35 para cargarlo en memoria. No necesitas reconstruir los datos ni los embeddings por cambiarlo. Para operación continua, consulta la [guía de tokens de Meta](https://developers.facebook.com/documentation/business-messaging/whatsapp/access-tokens).

## 4. ngrok: obtener el Authtoken

1. Crea una cuenta o inicia sesión en [ngrok](https://dashboard.ngrok.com/signup).
2. Abre [Your Authtoken](https://dashboard.ngrok.com/get-started/your-authtoken).
3. Copia solamente el valor del token; no copies un comando completo.
4. Guárdalo en Colab como `NGROK_AUTHTOKEN`.

El notebook instala la dependencia y abre el túnel desde Colab. No necesitas iniciar otro túnel en tu computadora. El Authtoken autentica al agente; una API Key de administración de ngrok tiene otro uso.

Referencia: [agente de ngrok](https://ngrok.com/docs/gateway/agent).

## 5. Colab: cargar el notebook y guardar los secretos

1. Abre [Google Colab](https://colab.research.google.com/) y carga `GeoComercio.ipynb` mediante **Archivo → Subir notebook**.
2. En **Entorno de ejecución → Cambiar tipo de entorno de ejecución**, selecciona una GPU disponible. Hazlo antes de ejecutar las instalaciones.
3. Abre el panel **Secretos**, identificado por el icono de una llave.
4. Añade estos nombres, respetando mayúsculas y minúsculas, y activa el acceso de este notebook a cada secreto.

| Nombre exacto | Valor que debes pegar |
| --- | --- |
| `huggin` | Token de lectura de tu cuenta autorizada de Hugging Face. |
| `WHATSAPP_ACCESS_TOKEN` | Token de acceso de Meta para WhatsApp. |
| `WHATSAPP_PHONE_NUMBER_ID` | Phone Number ID del emisor de prueba. |
| `NGROK_AUTHTOKEN` | Authtoken de tu cuenta de ngrok. |

Pega los valores sin comillas, sin `Bearer` y sin espacios adicionales. No publiques estos valores en el código, salidas, capturas ni repositorios.

El apartado 0 establece `NOMBRE_SECRETO_HF = "huggin"`. Si cambias ese nombre, también debes cambiar el secreto correspondiente.

Colab puede limitar recursos o terminar una máquina inactiva. Revisa sus [condiciones de uso y disponibilidad](https://research.google.com/colaboratory/faq.html). Guardar el notebook conserva el código, no la máquina en ejecución.

## 6. Ejecutar datos, modelos y filtros

Ejecuta por apartados y espera a que termine cada uno. No uses «Ejecutar todo» sobre la celda de borrado de historial.

| Orden | Apartados | Comprobación |
| --- | --- | --- |
| 1 | 0 | Aparece `Entorno preparado`. Si solicita reiniciar la sesión, hazlo una vez y repite 0. |
| 2 | 1–10 | Se descargan los datos y funcionan las consultas geográficas. |
| 3 | 11–17 | Ollama y Llama Stack arrancan; se definen herramientas y memoria. |
| 4 | 18–20 | Puedes probar una conversación y consultar su diagnóstico. |
| 5 | 21–28 | Se descarga el PDF y se registra la herramienta documental. |
| 6 | 29–34 | Se descarga Llama Guard 8B y se carga Prompt Guard. |

Las primeras descargas pueden tardar y requieren varios GB de disco. Ejecutar en CPU puede producir tiempos de espera. No instales otra versión de NumPy o PyTorch en mitad del flujo; usa la preparación del apartado 0.

Las celdas de prueba dependen de `EJECUTAR_PRUEBAS_LLAMA`. Revisa su valor donde se asigna, incluido el apartado 18. Para una consulta local con los filtros cargados:

```python
print(conversar_geo_seguro(
    "Busca cafeterías a 700 metros de latitud 19.4983 y longitud -99.1449.",
    usuario="demo",
))
```

No hace falta solicitar una clave de OpenAI, Google Maps o de la API del INEGI para este flujo.

## 7. Configurar WhatsApp y tu número personal

Ejecuta el apartado **35**, que activa WhatsApp y carga las credenciales. Después ejecuta el **36**, que prepara la cola y la base de mensajes.

La celda de destinatario, situada entre **36 y 37**, debe configurar el número directamente. Usa este bloque y sustituye `52XXXXXXXXXX` por el número mexicano que autorizaste en Meta: **52 seguido de 10 dígitos**.

```python
NUMERO_META_GEO = "52XXXXXXXXXX"

if not (
    NUMERO_META_GEO.isdigit()
    and NUMERO_META_GEO.startswith("52")
    and len(NUMERO_META_GEO) == 12
):
    raise ValueError("Escribe tu número con 52 seguido de sus 10 dígitos.")

numero_recibido = "521" + NUMERO_META_GEO[2:]
numero_meta = NUMERO_META_GEO

DESTINATARIOS_PRUEBA_GEO = globals().get("DESTINATARIOS_PRUEBA_GEO", {})
DESTINATARIOS_PRUEBA_GEO[numero_recibido] = numero_meta
DESTINATARIOS_PRUEBA_GEO[numero_meta] = numero_meta

print("Destinatario configurado.")
```

Esta equivalencia corresponde al caso de prueba mexicano del proyecto: acepta la identificación entrante con `521` y envía al destinatario autorizado con `52`. No debe aplicarse como regla universal a teléfonos de otros países. Para el historial, usa el remitente exacto que aparece en los mensajes recibidos.

Continúa con **37** (procesamiento y envío), **38** (webhook) y **39** (servidor y túnel). La comprobación pública debe devolver HTTP 200. Conserva la URL y el Verify Token que imprime 39.

## 8. Registrar el webhook en Meta

Con el servidor y el túnel activos:

1. Abre la configuración de WhatsApp de tu aplicación en Meta y busca **Webhooks / Configuración del webhook**.
2. En **Callback URL**, pega la URL HTTPS que imprime 39, completa y terminada en `/webhook`.
3. En **Verify Token**, pega el valor exacto de `VERIFY_TOKEN_WA`.
4. Pulsa **Verificar y guardar**.
5. En los campos de suscripción de WhatsApp, activa **`messages`** para recibir mensajes.

No uses `localhost`, el puerto 11434 de Ollama ni la ruta `/salud` como Callback URL. `/salud` es solamente una comprobación de disponibilidad.

Referencia: [crear un endpoint de webhook](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/create-webhook-endpoint/).

Si vuelves a ejecutar 39, comprueba la URL resultante. Cuando cambie, actualízala en Meta. Aunque el dominio sea el mismo, el túnel y el servidor deben permanecer activos.

## 9. Comprobar el recorrido completo

1. Desde tu teléfono personal autorizado, escribe al número de prueba empresarial de Meta: `Hola`.
2. Comprueba la bienvenida de GeoComercio.
3. Envía: `Busca cafeterías a 700 metros de latitud 19.4983 y longitud -99.1449.`
4. Espera la respuesta y revisa el apartado 40.
5. Envía: `¿Y en el mismo punto a 500 metros?` para comprobar la memoria.

```python
ver_whatsapp_geo(limite=10)
ver_eventos_geo(f"wa:{numero_recibido}", limite=10)
```

`aceptado_por_meta` indica que la API aceptó el envío; no equivale por sí solo a confirmación de entrega o lectura. Un mensaje antiguo con estado `error` permanece en el registro: revisa la fecha del último mensaje.

## 10. Reinicios y conservación de datos

| Situación | Qué hacer |
| --- | --- |
| Cambiaste el Access Token | Actualiza el secreto y ejecuta 35. |
| Reiniciaste el túnel | Comprueba la URL de 39 y el webhook registrado. |
| Reiniciaste el kernel | Vuelve a ejecutar las definiciones en orden; comprueba también los servicios. Los archivos pueden seguir en la misma máquina. |
| Colab eliminó o reemplazó la máquina | Ejecuta la preparación y descargas de nuevo. Recupera las bases desde tu respaldo si tienes uno. |
| Quieres empezar una conversación sin contexto | Usa `nueva_conversacion_geo(usuario)`; conserva análisis y eventos. |
| Quieres eliminar conversaciones de Stack | Ejecuta deliberadamente la celda de borrado, con el usuario correcto y sin solicitudes en curso. |

La celda **“Borrar el historial de conversacion en llama stack”** elimina elementos de conversación y sus enlaces locales. No la ejecutes durante el arranque normal. Tampoco equivale a borrar los análisis, todos los diagnósticos locales o todos los registros independientes de respuestas de Stack.

## 11. Problemas habituales

| Síntoma | Revisión o acción |
| --- | --- |
| `GatedRepoError`, HTTP 401 o 403 en Hugging Face | Comprueba aprobación de Prompt Guard, cuenta del token, permisos de lectura y acceso del notebook al secreto. |
| `SecretNotFoundError` o acceso al secreto deshabilitado | Revisa el nombre exacto y el interruptor de acceso en Colab. |
| `Primero envía un mensaje al número de prueba` | Configura la celda de destinatario con el bloque del apartado 7; no debe depender de mensajes guardados. |
| Meta `131030: Recipient phone number not in allowed list` | Comprueba el destinatario verificado y la equivalencia configurada. Un Phone Number ID correcto no autoriza automáticamente a tu teléfono personal. |
| Meta rechaza el token | Actualiza el Access Token y ejecuta 35. |
| No aparecen mensajes nuevos en 40 | Revisa el túnel, Callback URL, suscripción `messages`, número de prueba y Phone Number ID. Envía un mensaje nuevo después de corregirlos. |
| `/salud` local responde y la dirección pública falla | Revisa el túnel con 39 y actualiza el webhook si la URL cambia. |
| `ConnectionRefusedError` o `APIConnectionError` | Comprueba Ollama y Stack; ejecuta `asegurar_servicios_geo()`. |
| `APITimeoutError` o mensaje largo en `procesando` | Comprueba carga de modelos y uso de GPU; la cola procesa un mensaje a la vez. |
| Consulta legítima bloqueada | Revisa si bloqueó Prompt Guard o Llama Guard en los eventos y comprueba `MODELO_LLAMA_GUARD = "llama-guard3:8b"`. |
| Error de NumPy después de instalar | Reinicia el kernel si 0 lo solicita y empieza por 0. Evita reinstalaciones intermedias. |
| `NameError` de una función | Ejecuta primero el apartado que la define; guardar el archivo no ejecuta sus celdas. |

## 12. Antes de compartir o desplegar

- Comparte código y documentación sin credenciales ni números personales.
- Elimina salidas con datos privados antes de publicar el notebook.
- La demostración no exige App Secret porque no valida firmas de los POST entrantes. Para un servicio público permanente, incorpora esa validación: el Verify Token solo participa en el alta del webhook.
- Revisa los límites y condiciones de [Colab](https://research.google.com/colaboratory/faq.html), [ngrok](https://ngrok.com/pricing) y [WhatsApp Business Platform](https://business.whatsapp.com/products/platform-pricing).

## Lista final de preparación

- [ ] Acceso aprobado al repositorio de Prompt Guard.
- [ ] Cuatro secretos de Colab creados y habilitados.
- [ ] Número personal agregado y verificado en Meta.
- [ ] Phone Number ID tomado del emisor de prueba correcto.
- [ ] Llama conversacional, embeddings y ambos filtros cargados.
- [ ] Destinatario configurado antes de abrir el webhook.
- [ ] Comprobación pública HTTP 200.
- [ ] Callback URL y Verify Token guardados en Meta.
- [ ] Campo `messages` suscrito.
- [ ] Mensaje nuevo recibido y respuesta comprobada en WhatsApp.
