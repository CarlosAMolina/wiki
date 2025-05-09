# SNS

SNS = Simple Notification Service

## Características

- Opera en la zona pública de AWS. Por lo que puede usarse desde el internet público o desde una VPC con interfaz pública.
- Coordina el envío y entrega de mensajes, teniendo estos hasta 256 KB de payload.
- Ejemplo de uso, enviar notificaciones entre los servicios de AWS como CloudWath, CloudFormation, ASG, etc.
- Es un servicio con resilencia regional. Se replica los datos enviados dentro de la región, si falla una AZ funciona en otra; también escala en la región.
- Ofrece:
  - Conocer el estado del mensaje enviado. Creo que no todos los subscribers soportan esto.
  - Reintentos al enviar el mensaje.
  - SSE (Server Side Encryption).
  - Con un topic policy (es un resource policy) puede usarse SNS entre cuentas.

## Composición

### SNS Topic

Es en lo que se basa SNS.

En ellos se tratan los permisos y otras configuraciones.

Es por donde viajan los mensajes.

Presenta una comunicación one to many.

### Publisher

Es quien envía mensajes al topic.

### Subscribers

Reciben los mensajes enviados al topic; pueden recibir solo los mensajes que les interese.

Pueden ser: HTTP(s), email, SQS, SMS, Lambda, etc.

## Fanout arquitecture

Es cuando múltiples subscriptores SQS (Simple Queue Service) escuchan un topic; esto permite desencadenar múltiples procesos.

Por ejemplo, es útil si tenemos una aplicación que convierte los vídeos que llegan a S3 a diferentes calidades (480p, 720p, etc). Gracias a fanout podemos enviar el mensaje al topic SNS y a este topic tenemos subscritas una cola SQS por cada calidad; cada SQS iniciará el proceso que modifica la calidad del vídeo.
