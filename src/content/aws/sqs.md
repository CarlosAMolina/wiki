# SQS

SQS. Simple Queue Service.

## Introducción

Se encuentra en la zona pública de AWS.

Proporciona colas de mensajes.

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

Es la cola donde van mensajes erróneos. Ejemplo, mensajes que no han podido procesarse tras 5 intentos.

Permite hacer distintos tipos de procesado en estos mensajes.

## Polling

Cada petición puede recibir de SQS entre 0 y 10 mensajes; como máximo 64 KB de información.

Hay 2 tipos de polling:

- Short. Hace una petición y puede recibir 0 o más mensajes. Aunque se devuelvan 0 mensajes, sigue contando como petición.
- Long. Especificas waitTimeSeconds; como máximo de 20 segundos. Al hacer la petición, en caso de haber mensajes, los devuelve, pero si no hay mensajes, espera hasta que haya 10 mensajes o 64 KB o haya pasado el waitTimeSeconds. Este es el método recomendado al ser más económico.

## Coste

En base a peticiones que hagamos a SQS, no en base a número de mensajes.
