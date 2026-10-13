# Agente de soporte para VoltaTech (tienda en línea ficticia)

## 1. Integrantes

| Nombre completo | Matrícula | Rol |
|---|---|---|
| Carlos Leonel Astorga Trapero | 24040168 | Arquitectura, Datos e integraciones y Calidad (equipo de una persona) |

## 2. Escenario y problema que resuelve

Escenario B: soporte de una tienda en línea. VoltaTech es una tienda ficticia de electrónica (audífonos, cargadores, bocinas, smartwatches y accesorios). El agente atiende a los clientes por chat: responde dudas sobre envíos, cancelaciones, devoluciones, garantías y pagos usando las políticas de la tienda, consulta el estado de un pedido y, cuando el caso no se puede resolver (producto dañado, garantía, cobro o reembolso, o petición de hablar con una persona), crea un ticket para que un agente humano responda por correo. Se niega con amabilidad a lo que está fuera de su alcance y no inventa datos.

## 3. Diagrama del workflow



```
Chat Trigger ──▶ AI Agent
                  ├─ Chat Model:  Google Gemini Chat Model
                  ├─ Memory:      Simple Memory (ventana de 10 mensajes)
                  ├─ Tool: buscar_politicas   (Postgres PGVector Store + Embeddings Google Gemini)
                  ├─ Tool: consultar_pedido   (Data Table Tool: pedidos)
                  └─ Tool: crear_ticket       (Data Table Tool: tickets)
```

Workflow auxiliar `02-ingesta`: Form Trigger → Extract From File → Postgres PGVector Store (con Default Data Loader, Recursive Character Text Splitter y Embeddings Google Gemini).

## 4. Tabla de tools

| Tool | Qué hace | Cuándo debería usarla el agente | Lee / Escribe |
|---|---|---|---|
| `buscar_politicas` | Búsqueda semántica (RAG) en las políticas de la tienda | Cualquier duda sobre reglas, plazos, costos o condiciones | Lee |
| `consultar_pedido` | Busca un pedido por su número y devuelve producto, estado, paquetería, guía y fecha estimada | Cuando el cliente pregunta por su pedido y da el número | Lee |
| `crear_ticket` | Inserta un ticket para un agente humano con pedido, motivo, descripción y correo | Reclamos (dañado, garantía, cobro, reembolso), petición de hablar con una persona o caso sin solución | Escribe |

## 5. Stack elegido

| Capa | Elección | Por qué |
|---|---|---|
| Modelo de chat | Google Gemini (modelo flash lite) | Recomendado por el proyecto; plan gratuito suficiente. Se usó un modelo lite porque los otros daban 404 o 503 |
| Embeddings | Google Gemini `gemini-embedding-001` | Única opción gratuita viable; vectores de 3072 dimensiones |
| Base vectorial | Postgres con pgvector en Docker | Ya se había usado en clase; sin cuentas externas. Sin índice HNSW/IVFFlat (límite de 2000 dimensiones) |
| Memoria | Simple Memory | Alcanza para recordar la sesión del chat |
| Datos del negocio | Data Tables de n8n (`pedidos`, `tickets`) | No requieren cuentas externas |
| Canal | Chat Trigger de n8n | Suficiente para probar y aprobar |

## 6. Cómo ponerlo a funcionar

1. Levanta n8n con Docker y un contenedor de pgvector:
   ```bash
   docker run -d --name pgvector -e POSTGRES_PASSWORD=<tu_contraseña> -e POSTGRES_DB=voltatech -p 5433:5432 pgvector/pgvector:pg16
   ```
2. En esa base ejecuta:
   ```sql
   CREATE EXTENSION IF NOT EXISTS vector;
   CREATE TABLE IF NOT EXISTS politicas_voltatech (
     id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
     text TEXT,
     metadata JSONB,
     embedding VECTOR(3072)
   );
   ```
3. Crea las credenciales en n8n (nunca dentro de un nodo):
   - **Google Gemini (PaLM) API**: tu API key de aistudio.google.com.
   - **Postgres**: host `host.docker.internal`, puerto `5433`, base `voltatech`, SSL en Disable.
4. Crea las Data Tables `pedidos` (pedido_id, cliente, email, producto, total, estado, paqueteria, guia, fecha_pedido, fecha_estimada_entrega) y `tickets` (pedido_id, motivo, descripcion, email_cliente, estado), todas de tipo String. Carga en `pedidos` los 12 pedidos ficticios.
5. Importa `workflows/02-ingesta.json`, asigna las credenciales, ejecútalo y sube `politicas.md` en el formulario.
6. Importa `workflows/01-agente.json`, asigna las credenciales y pega `system-prompt.md` como System Message (en modo Expression).
7. Abre el chat del workflow y prueba con los casos de `casos.csv`.

Todos los datos son ficticios.
