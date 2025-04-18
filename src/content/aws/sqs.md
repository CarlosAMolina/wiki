# SQS

SQS. Simple Queue Service.

## Introducción

Se encuentra en la zona pública de AWS.

Proporciona colas de mensajes. Las colas ofrecen una comunicación uno a uno.

Al crear una cola SQS, son high availability y gestionada por AWS. Está disponible en una región.

Tamaño de los mensajes puede ser máximo 256 KB. Para tamaños mayores, podemos guardar los datos en S3 y en los mensajes SQS indicar la ruta de S3.

Como máximo los mensajes pueden estar en la cola hasta 14 días.

Soporta cifrado KMS en la cola y al enviarlo a los clientes.

Acceder a una cola se hace en base a identity policies o con queue policy (es un resource policy). Ambos controlan acceso desde una cuenta pero solo queue policy permite controlar acceso desde cuentas externas.

## Tipos

### FIFO

Garantiza que cada mensaje llegará en orden y una vez.

Máximo 3.000 mensajes por segundo de usar batching o 300 mensajes de no usarlo.

Las colas de este tipo deben tener el sufijo `.fifo`.

Usos: procesos que necesitan ordenar los datos, ajustar precios, etc.

### Standard

Garantiza que los mensajes llegarán por lo menos una vez, puede que lleguen más veces.

El orden de los mensajes es un best effort; intenta entregarlos como FIFO pero puede que no lo consiga siempre.

No tiene máximo de mensajes por segundo, puede escalar indefinidamente.

Usos: desacoplar pools de workers, almacenar datos para un procesamiento futuro, etc.

## Funcionamiento

- Clientes envían mensajes a la cola.
- Clientes revisan si hay mensajes en la cola.

### Visibility timeout

Cuando un cliente recibe un mensaje, este no se elimina de la cola, pero deja de estar visible por un periodo de tiempo llamado visibility time out.

Cuando un cliente termina de procesar un mensaje, para eliminarlo debe realizar la acción de eliminación de manera explícita.

Si no es eliminado, el mensaje aparece de nuevo en la cola tras el visibility time. Esto es útil en casos de error donde el mensaje no se procesó correctamente.

Durante el visibility timeout, el mensaje no está disponible en la cola.

Valor por defecto es de 30 segundos, los podemos configurar de 0 segundos a 12 horas. Puede configurarse para la cola o por mensaje.

## SQS Delay Queues

Permite indicar un tiempo inicial de espera hasta que los mensajes puedan ser visibles para los consumers.

Su valor por defecto es 0, el máximo son 15 minutos.

Puede hacerse una configuración por cola o por mensaje:

- Por cola. Usamos delay queues. El valor configurado se llama `DelaySeconds`.
- Por mensaje. Esta opción no está disponible en colas FIFO. La opción por mensaje sobreescribe la configuración de la cola. Se configura con los message timers.

Utilizado cuando queremos un delay en nuestra aplicación hasta que se inicia el proceso de los mensajes.

### Dead-Letter queues

Es la cola donde van mensajes erróneos que no han podido procesarse tras varios intentos. Ejemplo, tras 5 intentos.

Se configura una redrive policy, en esta se especifica:

- Source Queue que monitorizar.
- Dead-Letter Queue al que enviar los mensajes erróneos. Puede usarse una misma Dead-Letter para diferentes Source Queue.
- Configuramos el `maxReceiveCount`. Cuando un mensaje vuelve a la cola tras el Visibility Timeout, se incrementa el ReceiveCount; si este es mayor que el maxReceiveCount, el mensaje es movido a la Dead-Letter Queue.

Permite realizar distintas acciones en estos mensajes. Por ejemplo:

- Enviar una notificación informando del mensaje que llega a la cola.
- Crear y analizar los logs del mensaje erróneo o su contenido, para determinar la causa de que no se procese.
- Aplicar otro procesamiento a estos mensajes.

Hay que tener en cuenta que, cuando un mensaje llega a la cola original, se le asigna un timestamp de llegada, llamado `enqueue timestamp`. Al mover el mensaje a la Dead-Letter Queue, este valor no se actualiza y el `retention period` de la dead-letter queue debe tener en cuenta el `retention period` de la cola original; es decir, si en la cola original estuvo 1 día y el `retention period` del dead-letter queue es de 2 días, solo estará en el la cola dead-letter 1 día hasta ser eliminado automáticamente.

## Polling

Cada petición puede recibir de SQS entre 0 y 10 mensajes; como máximo 64 KB de información.

Hay 2 tipos de polling:

- Short. Hace una petición y puede recibir 0 o más mensajes. Aunque se devuelvan 0 mensajes, sigue contando como petición.
- Long. Especificas waitTimeSeconds; como máximo de 20 segundos. Al hacer la petición, en caso de haber mensajes, los devuelve, pero si no hay mensajes, espera hasta que haya 10 mensajes o 64 KB o haya pasado el waitTimeSeconds. Este es el método recomendado al ser más económico.

## Coste

En base a peticiones que hagamos a SQS, no en base a número de mensajes.
