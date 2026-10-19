La prioridad de nuestro proyecto, DC-Organizador de Gastos, es ayudar a los usuarios a llevar un registro detallado de sus ingresos y egresos, automatizando este proceso a través de mensajería de texto para brindar una experiencia más ágil, amigable y eficiente.

*¿Cómo se implementa el desarrollo de este proyecto?*
Para llevar a cabo este proyecto, se realizó una investigación previa sobre las tecnologías, posibilidades y conceptos clave a integrar.

Principalmente, se configuró la integración con la API de Telegram, cuya función es la recepción y envío de mensajes hacia y desde el sistema automatizado. Dentro de esta plataforma se creó el chatbot, encargado de capturar la información que posteriormente será procesada por n8n.

n8n es la plataforma de automatización encargada de conectar servicios, bases de datos, aplicaciones e Inteligencia Artificial. Gracias a ella, estas herramientas trabajan en conjunto de manera visual y versátil a través de flujos de trabajo (workflows), eliminando la necesidad de realizar las tareas manualmente.

*Flujo de trabajo*
Tal como se mencionó anteriormente, el flujo de trabajo inicia mediante el nodo de Telegram para posteriormente conectarse con el agente de IA. El proceso se activa cada vez que el usuario envía un mensaje y se reciben nuevos datos. Posteriormente, se evalúa la información para estructurarla y procesarla de forma precisa mediante la API del modelo de chat de Gemini AI.
