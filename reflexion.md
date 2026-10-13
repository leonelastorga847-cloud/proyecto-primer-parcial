# Reflexión

## 1. ¿Qué casos fallaron y por qué? ¿Qué cambié para corregirlos?

Los 10 casos de `casos.csv` pasaron en la corrida final. Las fallas ocurrieron durante la construcción, antes de correr los casos:

- **Modelo de chat no disponible (404).** `gemini-2.5-flash` ya no está disponible para cuentas nuevas. Cambié a un modelo más reciente.
- **Servicio saturado (503).** El modelo más nuevo respondía con "high demand". Cambié a un modelo flash lite y activé *Retry On Fail* en el agente.
- **Tabla de vectores con el nombre equivocado.** Puse el nombre de la tool en el campo *Table Name* del PGVector Store. Lo corregí con el nombre real de la tabla y renombré el nodo para que el agente viera `buscar_politicas`.
- **Default Data Loader sin splitter.** Con *Text Splitting = Custom* la ingesta falló hasta conectar el Recursive Character Text Splitter.
- **`consultar_pedido` no encontraba el pedido 1002.** La tabla `pedidos` estaba vacía porque no se había importado el CSV. La cargué con un workflow temporal de una sola ejecución.
- **Caso 7 mal planteado.** La primera vez mandé los dos turnos en un solo mensaje, así que no probaba la memoria. Lo repetí con dos mensajes separados en la misma sesión.

## 2. Una limitación que no pude resolver

El agente **no verifica la identidad del cliente**: cualquiera que sepa un número de pedido puede consultar su estado y su guía. El prompt le prohíbe compartir nombre y correo, pero los números de pedido son consecutivos y fáciles de adivinar. Resolverlo bien requiere autenticación (por ejemplo, pedir el correo del pedido y validarlo contra la tabla) y no alcancé a hacerlo.

Otras limitaciones: la memoria Simple Memory se borra al reiniciar n8n, y el modelo lite depende de la disponibilidad del servicio de Google.

## 3. ¿Qué cambiaría para producción con usuarios reales?

- Autenticar al cliente antes de mostrar datos de un pedido.
- Memoria persistente (Postgres Chat Memory) y un canal real (Telegram o WhatsApp).
- Guardar los tickets en un sistema de soporte con notificación al equipo, no solo en una tabla.
- Un modelo de pago con mejor disponibilidad y límites claros, con un plan de respaldo si falla.
- Evaluaciones automáticas (n8n Evaluations) con muchos más casos, incluyendo intentos de manipulación del prompt.
- Revisión legal del manejo de datos personales; el plan gratuito de Gemini puede usar lo que se le envía para mejorar sus productos.
- Monitoreo de ejecuciones fallidas y de costos.

## 4. Qué hizo cada integrante

Carlos Leonel Astorga Trapero (24040168) hizo todo el proyecto, en un equipo de una persona:

- **Arquitectura:** diseño del agente, redacción del system prompt y definición de las tools y sus descripciones.
- **Datos e integraciones:** contenedor de pgvector, workflow de ingesta, credenciales, Data Tables `pedidos` y `tickets`, y los datos ficticios.
- **Calidad:** diseño y ejecución de los 10 casos de prueba, capturas de evidencia, README y esta reflexión.
