# Ejemplos

## 13 servicios para construir prácticamente cualquier cosa

Fuente: newsletter de desplegando.cloud.

Ejemplo app, un ecommerce típico con frontend, backend, procesos asíncronos y todo lo que eso implica. Con los siguientes 13 servicios (+ el bonus) tennemos frontend, backend, datos, autenticación, comunicación asíncrona, orquestación, infraestructura como código y observabilidad. Una aplicación completa.

### El frontend

Route 53. Tu dominio. El punto de entrada. También maneja health checks, ruteo por geolocalización y failovers si escalas a múltiples regiones.

Amplify. Hosteas y despliegas tu frontend con CI/CD automático. Conectas tu repositorio de GitHub y cada push despliega solo. Soporta React, Next.js, Vue. HTTPS y CDN incluidos. La forma más rápida de tener un frontend en producción.

### El backend

API Gateway. Puerta de entrada al backend. Expone tus APIs, maneja autenticación, throttling y caché. Sin código, todo configuración.

Lambda — el corazón de serverless. Escribes una función, la subes, AWS escala solo. Pagas por ejecución. Usada para APIs, automatizaciones, procesamiento de eventos.

DynamoDB. Base de datos NoSQL, rápida, barata para aprender y extremadamente económica a escala. No es relacional, pero si la aprendes a usar, vuela.

S3. Almacenamiento de objetos. Imágenes de productos, archivos de usuarios, backups. Y el punto de partida de muchos flujos: subes un archivo y eso dispara un proceso.

Cognito. Autenticación y autorización listas para usar. Login, registro, MFA, social login con Google o Facebook. Se integra nativo con API Gateway y Amplify. No construyas tu propio sistema de auth.

### Los procesos asíncronos

EventBridge. Bus de eventos. Cuando algo pasa en tu aplicación (se creó una orden, un usuario se registró), publicas un evento y EventBridge lo rutea a donde tiene que ir.

SQS. Cola de mensajes. Absorbe cargas de trabajo, amortigua picos. Si llegan 10,000 órdenes de golpe, la cola las contiene y el sistema las procesa a su ritmo sin caerse.

Step Functions. Cuando necesitas un proceso con pasos en orden, con validaciones, reintentos y lógica clara. Mi newsletter entera se automatiza con Step Functions y me cuesta €0 por mes.

SNS. Notificaciones. Mandas un mensaje y SNS lo entrega como email, SMS, push notification o señal a otro sistema. Combinado con SQS es el patrón fan-out: un mensaje llega a múltiples destinos al mismo tiempo.

### La infraestructura y observabilidad

CloudFormation + CDK. Infraestructura como código desde el día uno. No configures cosas a mano. Define tu arquitectura en código, versiónala, revísala, hace rollbacks. CDK te deja escribirla en Python, TypeScript o Java en lugar de YAML.

CloudWatch. Logs, métricas, alarmas. Cuando algo falla, CloudWatch es lo primero que miras. Si no tienes observabilidad, estás volando a ciegas.

### El bonus

Bedrock. IA generativa en AWS. Acceso a modelos y herramientas para construir agentes. Ya está apareciendo en casi todas las aplicaciones nuevas.
