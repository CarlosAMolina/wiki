# Serverless

## Caracterísiticas

- Las aplicaciones son colecciones de funciones pequeñas y especializadas en realizar una acción.
- No se gestiona ninguno o casi ningún servidor. Esto es responsabilidad del proveedor del servicio.
- Los entornos son stateless y efímeros; al ejecutarse obtienen la información que necesitan y para almacenarla lo hacen en algo externo.
- Utilizan arquitectura event-driven.
- Se intentan emplear productos de terceros cuando sea posible (ejemplo para el login, así evitamos su desarrollo, mantenimiento y coste) y FaaS cuando se necesite realizar operaciones de cómputo.

## Event-Driven Arquitecture

Las diferentes partes de la aplicación se comunican por eventos; están desacopladas unas de otras.

Las partes de la aplicación solo se ejecutan cuando es requerido.

La aplicación la forman:

- Producers: generan eventos.
- Consumers: reciben eventos y ejecutan un acción.
- Hay elementos que son producers y consumers.
- Buenas arquitecturas event-driven, en lugar de conectar todos los elementos entre sí, utilizan un event router para gestionar los eventos. Los eventos viajan por un event bus.

Si la arquitectura es de tipo serverless, solo se consumen recursos cuando se gestionan eventos.

## Alternativas open source

- [Apache Kafka](https://kafka.apache.org/)
- [Artículo sobre cambio de arquitectura monolítiva a AWS y finalmente a Celery y Kubernetes](https://itnext.io/back-to-the-future-the-future-might-not-just-be-serverless-after-all-b8165d6e84c2)

## Patrones serverless en AWS

Nota. Información obtenida de [desplegando.cloud](https://desplegando.cloud/).

Ejemplos de problemas a solucionar con los patrones que veremos a continuación:

Ejemplo 1. Si llevamos nuestra aplicación a una función de Lambda gigante que hace:

- Request/Response (Patrón 1).
- Background processing (Patrón 2).
- Eventos (Patrón 3).
- Todo mezclado en una función.

Resultado:

- Usuario esperaba 10 segundos
- Si algo fallaba, fallaba todo
- Imposible escalar partes independientes

Al aplicar los patrones ganamos:

1. Respuestas rápidas: usuario ve confirmación en <1 segundo.
2. Costos optimizados: cada parte paga solo por su tiempo.
3. Resiliencia: si el email falla, el pedido ya está guardado. 
4. Visibilidad: ves exactamente dónde está cada proceso.
5. Escalamiento independiente: la API escala diferente que los reportes.

Ejemplo 2. WebApp de e-learning usando los siguientes 5 patrones desde el día uno:

- API síncrona: Consultar cursos (respuesta inmediata)
- Workflows: Procesar inscripciones, generar certificados (Step Functions)
- Eventos: Notificar cuando un usuario completa un curso (EventBridge)
- Scheduled: Enviar recordatorios semanales (EventBridge Schedule)
- Real-time: Chat entre estudiantes (WebSockets)

Resultado:

- Escaló y creció los usuarios sin cambiar la arquitectura
- Costo: $0 durante desarrollo, menos de $10/mes con 1,000 usuarios
- Cada parte funciona independiente- Fácil de mantener y extender

### Patrón 1: API Síncrona (Request/Response)

Cuándo: El usuario necesita una respuesta inmediata.

Ejemplo:

- Consultar datos
- Validar formulario

Arquitectura: Frontend -> API Gateway -> Lambda -> DynamoDB -> Response

Regla de oro: Si el usuario está esperando, debe ser síncrono y rápido (<1 segundo).

### Patrón 2: Workflow Asíncrono (Background Processing)

Cuándo: El proceso tiene múltiples pasos o tarda más de 3 segundos.

Ejemplo:

- Procesar pedido (validar -> pagar -> confirmar -> notificar)
- Generar reporte complejo
- Importar datos masivos

Arquitectura: API -> EventBridge/SQS -> Step Functions -> funciones específicas

Regla de oro: Si tiene pasos o puede fallar, usa orquestación (Step Functions).

Beneficio: Visibilidad completa del flujo, reintentos automáticos, cada paso independiente.

### Patrón 3: Eventos (Pub/Sub)

Cuándo: Múltiples sistemas necesitan reaccionar al mismo evento.

Ejemplo:

- Usuario se registra -> Enviar email + Crear perfil + Notificar admin + Analytics
- Pedido completado -> Actualizar inventario + Enviar factura + Notificar vendedor

Arquitectura: Evento -> EventBridge -> Múltiples funciones independientes

Regla de oro: Si otros sistemas necesitan saberlo, usa eventos. Cada listener es independiente.

Beneficio: Agregar nuevos listeners sin tocar código existente.

### Patrón 4: Scheduled Tasks (Cron Jobs)

Cuándo: Algo debe pasar en un horario específico.

Ejemplo:

- Enviar reportes diarios a las 8am
- Limpiar datos viejos cada domingo
- Sincronizar con sistemas externos cada hora

Arquitectura: EventBridge Schedule -> Lambda/Step Functions

Regla de oro: Si es periódico, usa scheduler. No polling, no servidores corriendo 24/7.

Beneficio: $0 cuando no se ejecuta, solo pagas por las ejecuciones.

### Patrón 5: Real-time Updates (WebSockets)

Cuándo: El frontend necesita actualizaciones en tiempo real sin refrescar.

Ejemplo:

- Chat en vivo
- Notificaciones instantáneas
- Dashboard con datos en tiempo real

Arquitectura:

API Gateway WebSocket / AWS AppSync Events -> Lambda -> DynamoDB Streams -> Lambda -> WebSocket

Regla de oro: Si el usuario necesita ver cambios sin refrescar, usa WebSockets.

Beneficio: Conexiones persistentes gestionadas por AWS, escalamiento automático.
