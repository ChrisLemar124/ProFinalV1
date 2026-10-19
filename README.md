# Asistente Financiero Personal con n8n, Telegram, Google Gemini y Google Sheets

Este proyecto implementa un flujo de trabajo automatizado en **n8n** que funciona como un asistente financiero personal a través de **Telegram**. Utiliza un agente de inteligencia artificial impulsado por **Google Gemini** (`models/gemini-3.1-flash-lite`) para interpretar mensajes de texto sobre ingresos o gastos.

Principalmente, se configuró la integración con la API de Telegram, cuya función es la recepción y envío de mensajes hacia y desde el sistema automatizado. Dentro de esta plataforma se creó el chatbot, encargado de capturar la información que posteriormente será procesada por n8n.

*¿Cómo se implementa el desarrollo de este proyecto?*
Para llevar a cabo este proyecto, se realizó una investigación previa sobre las tecnologías, posibilidades y conceptos clave a integrar.

## Diseño General del Workflow

El flujo de trabajo está compuesto por una tubería principal de tres nodos (`Telegram Trigger`, `AI Agent` y `Send a text message`) y tres sub-nodos conectados al agente para el procesamiento del lenguaje natural y la lectura/escritura en la hoja de cálculo.


![alt text](<Archivos/WhatsApp Image 2026-10-19 at 12.38.30 AM.jpeg>)

## Requisitos Previos

1. **Instancia de n8n:** Versión compatible con nodos de LangChain (`@n8n/n8n-nodes-langchain.agent` v3.1).
2. **Bot de Telegram:** Creado mediante BotFather con su respectivo Token de API configurado en las credenciales de n8n.
3. **Google Gemini API:** Acceso configurado al modelo `models/gemini-3.1-flash-lite`.
4. **Google Sheets (OAuth2):** Credencial activa en n8n con permisos de lectura y escritura sobre Google Drive y Google Sheets.

## Existen  modelos divergentes
En este repositorio existen dos modelos presentados, siendo versiones que cumplen los mismos objetivos pero su implementación es diferente, se muestran ambos para su uso y recolección de feedback con el objetivo de aprender que modelo tiene mejor uso.


## Configuración de Nodos en n8n

### 1. Telegram Trigger
Escucha en tiempo real los mensajes entrantes enviados por el usuario al bot de Telegram.

* **Tipo de nodo:** `n8n-nodes-base.telegramTrigger`
* **Updates:** `message`
* **Additional Fields:** `Download: true`
* **Credencial:** `Telegram account`

![alt text](Archivos/image-7.png)
---



### 2. AI Agent
Es el núcleo del flujo de trabajo. Recibe el mensaje de Telegram y la fecha actual, consulta el historial en Google Sheets, aplica las reglas de negocio y escribe la nueva fila actualizada.

* **Tipo de nodo:** `@n8n/n8n-nodes-langchain.agent`
* **Prompt Type:** `Define below`
* **Expresiones dinámicas utilizadas:**
  * Fecha actual: `{{ $now.format('YYYY-MM-DD') }}`
  * Mensaje de entrada: `{{ $json.message.text }}`

#### Flujo de pasos definidos en el Prompt del Agente:
1. **Paso 1 (Leer Historial):** Ejecuta siempre la herramienta `Leer_Historial_Gastos` para obtener la última fila registrada (`Presupuesto_Inicial`, `Saldo_Fijos`, `Saldo_Basicos`, `Saldo_Libre` y `Presupuesto_Restante`). Si no hay filas previas, asume $0$ en todos los valores.
2. **Paso 2 (Calcular Movimiento):**
   * **Caso A (Reiniciar Presupuesto):** Registra `Tienda: "Nuevo Presupuesto Asignado"`, `Categoria: "Presupuesto"` y `Monto: 0`. Asigna el nuevo `Presupuesto_Inicial` y calcula los tres saldos multiplicando por $0.50$, $0.30$ y $0.20$ respectivamente.
   * **Caso B (Gasto por texto o ticket):** Identifica `Fecha`, `Tienda`, `Categoria` y `Monto`. Mantiene intacto el `Presupuesto_Inicial` de la fila anterior, resta el `Monto` únicamente de la cubeta correspondiente a la categoría, y actualiza `Presupuesto_Restante` sumando las tres cubetas.
3. **Paso 3 (Guardar en Google Sheets):** Ejecuta la herramienta `Modificar_Filas` enviando las 9 columnas actualizadas.
4. **Paso 4 (Responder):** Devuelve un resumen estructurado en texto plano, sin etiquetas de código ni formato Markdown complejo, con el detalle del movimiento y los saldos actuales.

![alt text](Archivos/image-2.png)

---

### 3. Google Gemini Chat Model
Proporciona el modelo de lenguaje encargado del razonamiento del agente y la invocación de herramientas.

* **Tipo de nodo:** `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`
* **Model Name:** `models/gemini-3.1-flash-lite`

![alt text](Archivos/image-3.png)

---

### 4. Leer_Historial_Gastos (Google Sheets Tool)
Herramienta conectada al agente para consultar los registros existentes antes de realizar cualquier cálculo.

* **Tipo de nodo:** `n8n-nodes-base.googleSheetsTool`
* **Operation:** `Get Rows` (lectura por defecto)
* **Document:** `Control de Gastos 2026`
* **Sheet:** `Hoja 1` (`gid=0`)

![alt text](Archivos/image-4.png)

---
### 5. Modificar_Filas (Google Sheets Tool)
Herramienta conectada al agente para insertar una nueva fila con los saldos actualizados mediante extracción automática de parámetros con `$fromAI()`.

* **Tipo de nodo:** `n8n-nodes-base.googleSheetsTool`
* **Operation:** `Append Row` (`append`)
* **Document:** `Control de Gastos 2026`
* **Sheet:** `Hoja 1` (`gid=0`)
* **Mapping Column Mode:** `Map Each Column Manually` (`defineBelow`)
* **Valores mapeados por la IA:**
  * `Fecha`: `={{ $fromAI('Fecha', '', 'string') }}`
  * `Tienda`: `={{ $fromAI('Tienda', '', 'string') }}`
  * `Categoria`: `={{ $fromAI('Categoria', '', 'string') }}`
  * `Monto`: `={{ $fromAI('Monto', '', 'string') }}`
  * `Presupuesto_Inicial`: `={{ $fromAI('Presupuesto_Inicial', '', 'string') }}`
  * `Saldo_Fijos`: `={{ $fromAI('Saldo_Fijos', '', 'string') }}`
  * `Saldo_Basicos`: `={{ $fromAI('Saldo_Basicos', '', 'string') }}`
  * `Saldo_Libre`: `={{ $fromAI('Saldo_Libre', '', 'string') }}`
  * `Presupuesto_Restante`: `={{ $fromAI('Presupuesto_Restante', '', 'string') }}`

![alt text](Archivos/image-5.png)

---

### 6. Send a text message (Telegram)
Envía la respuesta final generada por el agente de vuelta al chat del usuario en Telegram.

* **Tipo de nodo:** `n8n-nodes-base.telegram`
* **Chat ID:** `={{ $('Telegram Trigger').item.json.message.from.id }}`
* **Text:** `={{ $json.output }}`
* **Credencial:** `Telegram account`

![alt text](Archivos/image-6.png)

---

## Instalación y Puesta en Marcha
1. Descarga el archivo `Proyecto.json` de este repositorio.
2. En tu panel de **n8n**, crea un nuevo flujo de trabajo y selecciona **Import from File** en el menú superior derecho.
3. Selecciona el archivo `Proyecto.json`.
4. Asigna tus propias credenciales en los nodos:
   * `Telegram Trigger` y `Send a text message` (Token de tu bot de Telegram).
   * `Google Gemini Chat Model` (API Key de Google AI Studio / Gemini).
   * `Leer_Historial_Gastos` y `Modificar_Filas` (Cuenta de Google Sheets OAuth2).
5. Crea una hoja de cálculo en Google Sheets con las 9 columnas especificadas y selecciónala en ambos nodos de Google Sheets.
6. Activa el flujo de trabajo cambiando el interruptor principal a **Active**.


## Objetivo
La prioridad de nuestro proyecto, DC-Organizador de Gastos, es ayudar a los usuarios a llevar un registro detallado de sus ingresos y egresos, automatizando este proceso a través de mensajería de texto para brindar una experiencia más ágil, amigable y eficiente.


## Estructura de Google Sheets para los modelos:: Versión:1
El flujo se conecta a un documento llamado **Control de Gastos 2026** dentro de la pestaña **Hoja 1**. La tabla debe contar exactamente con las siguientes 9 columnas en la primera fila:

El flujo se conecta a un documento llamado **Control de Gastos 2026** dentro de la pestaña **Hoja 1**. La tabla debe contar exactamente con las siguientes 9 columnas en la primera fila:

| Columna | Nombre Exacto del Campo | Descripción |
| :--- | :--- | :--- |
| **A** | `Fecha` | Fecha del movimiento en formato `YYYY-MM-DD`. |
| **B** | `Tienda` | Concepto o establecimiento donde se realizó el movimiento. |
| **C** | `Categoria` | Categoría asignada al movimiento o asignación de presupuesto. |
| **D** | `Monto` | Valor monetario del gasto registrado ($0$ cuando se reinicia el presupuesto). |
| **E** | `Presupuesto_Inicial` | Monto base asignado al presupuesto general. |
| **F** | `Saldo_Fijos` | Saldo disponible en la cubeta de gastos fijos ($50\%$). |
| **G** | `Saldo_Basicos` | Saldo disponible en la cubeta de gastos básicos ($30\%$). |
| **H** | `Saldo_Libre` | Saldo disponible en la cubeta de gastos libres o ahorro ($20\%$). |
| **I** | `Presupuesto_Restante` | Suma total disponible de las tres cubetas actuales. |

![Estructura de las columnas en Google Sheets](Archivos/image-1.png)

---

## Lógica Financiera: Sistema de Cubetas

El agente administra el dinero dividiendo el presupuesto inicial en tres sobres o cubetas con porcentajes fijos:

1. **Cubeta Fijos (`Saldo_Fijos`):** Recibe el $50\%$ del presupuesto inicial ($\text{Presupuesto\_Inicial} \times 0.50$).
   * **Categorías que descuentan de esta cubeta:** Renta, Luz, Agua, Gas, Internet, Servicios, Escuela.
2. **Cubeta Básicos (`Saldo_Basicos`):** Recibe el $30\%$ del presupuesto inicial ($\text{Presupuesto\_Inicial} \times 0.30$).
   * **Categorías que descuentan de esta cubeta:** Comida, Despensa, Limpieza, Cuidado personal.
3. **Cubeta Libre/Ahorro (`Saldo_Libre`):** Recibe el $20\%$ del presupuesto inicial ($\text{Presupuesto\_Inicial} \times 0.20$).
   * **Categorías que descuentan de esta cubeta:** Ocio, Regalos, Mascotas, Ahorro, Otros.
