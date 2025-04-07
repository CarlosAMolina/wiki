# Lambda

## Introducción

Lambda es un servicio FaaS (Function as a Service), lo que significa que está enfocado a procesos de corta duración.

Un lambda function:

- Es un código que lambda ejecuta.
- Utiliza un runtime, por ejemplo Python 3.11. Pueden crearse runtimes propios gracias a las layers.
- Se ejecuta en un runtime environment (también llamado execution context). Es como un contenedor al que definimos los recursos a utilizar:
  - Memoria. De 128 MB a 10.240 MB.
  - CPU. Según la cantidad de memoria, AWS asigna una cantidad de CPU.
  - Almacenamiento. Va de 512 MB a 10.240 MB.
- Tiene un tiempo de funcionamiento máximo de 15 minutos.

La lambda se despliega en AWS como un paquete.

El cobro es en base a los recursos asignados y el tiempo de ejecución.

## Networking

Tiene dos modos: público y VPC.

### Público

Por defecto se sitúan en al red pública de AWS.

Ofrece el mejor rendimiento al no necesitar configuraciones específicas de VPC.

No puede acceder a recursos en VPC a menos que el VPC tenga una IP pública y sus security controls permitan acceso externo.

### VPC

Al estar en una VPC se le aplican todas las reglas del VPC; es decir, puede acceder a lo que contenga la VPC pero no a recursos externos a menos que la VPC esté configurada para ello.

La lambda también necesita que el rol le otorgue ec2 network permissions para crear network interfaces en la VPC.

Las lambdas no se encuentran dentro de la VPC sino que están en una VPC propia del servicio Lambda y al ejecutarse deben conectarse al VPC.

Para conectarse al VPC, antes AWS lo gestionaba de modo que al ejecutarse crean un ENI para conectarse al VPC, esto consume tiempo. También, de ejecutar la lambda en paralelo o de manera concurrente, cada ejecución necesita una ENI, lo que no es una solución escalable.

Actualmente, este problema se soluciona porque en lugar de utilizar un ENI por ejecución, AWS analiza las lambdas en la región y en la cuenta del usuario para agruparlas en una combinación de subredes y security groups, por cada una de estas combinaciones se usa una ENI, por ejemplo:

- Si las lambdas usan distintas subredes y el mismo security group. Se necesita un ENI por subred.
- Si las lambdas usan la misma subred y el mismo security group. Se necesita un solo ENI.

En la arquitectura actual, la ENI se crea al configurar la lambda o cuando se modifica la configuración de red, por lo que no supone retardo cuando se invoca la lambda. Crear la ENI requiere 90 segundos.

### Seguridad

La lambda utiliza:

- IAM roles para interactuar con otros servicios.
- Resource policies para definir qué servicios y cuentas pueden invocar funciones lambda. Actualmente no puede modificarse con la consola de AWS, solo con la CLI o por API.

### Logs

Puede utilizar:

- CloudWatch. Guarda las métricas; por ejemplo la duración de la ejecución, invocaciones, fallos, latencia, etc.
- CloudWatch Logs. Recibe cualquier log generado en una lambda. Para esto, la lambda requiere permisos mediante un execution rol.
- X-Ray. Puede integrarse con X-Ray para obtener información de distributed tracing, por ejemplo sobre el usuario o la parte de una aplicación serverless que invoca la lambda.

### Tipos de invocación

#### Sincronización síncrona

Al ser síncrono, tras invocar a la lambda se espera una respuesta, que indica si el proceso fue correcto o tuvo fallos.

La gestión de errores o reprocesar lo realiza el cliente.

Ejemplo, cuando se invoca la lambda:

- Desde CLI o API.
- API Gateway.

#### Sincronización asíncrona

Cuando un servicio de AWS invoca una lambda.

Una vez invocada la lambda, no esperamos respuesta.

Por ejemplo, al subir una imagen a S3.

La lambda es responsable de volver a ejecutarse en caso de error. Podemos configurar que haya entre 0 y 2 intentos.

La lambda debe ser `idempotent`, es decir, que al volver a procesarse de el mismo resultado. Por ejemplo, si tengo 10€ en mi banco y quiero incrementarlo a 20€, si las lambda lo que hace es sumar 10€, en caso de error y volver a procesar puede que haya sumado 10€ más de una vez y acabe con más de 20€; pero si la lambda lo que hace es establecer la cantidad total de 20€, siempre se tendrá el mismo resultado aunque se ejecute varias veces.

La lambda puede enviar los eventos que no ha podido procesar tras los intentos a una DLQ (Dead Letter Queues), utilizado por ejemplo para diagnosis.

La lambda puede enviar el evento de salida (sea procesado correctamente o fallido) a destinos como:

- SQS.
- SNS.
- Lambda.
- EventBridge.

La lambda no necesita permisos sobre el origen para leer el evento enviado; pero si necesita solicitar información adicional entonces sí requiere más permisos.

#### Event source mappings

Utilizada en streams o colas que no soportan generación de eventos para invocar lambdas, ejemplo:

- Kinesis.
- DynamoDB streams.
- SQS.

En los productos de tipo stream, los consumidores pueden leer del stream, pero no se generan eventos al añadir datos. Como la lambda necesita ser notificada cuando hay nuevos datos, se utiliza el Event Source Mapping que va leyendo el stream o las colas, devuelve la información al stream o cola (a esta información se le llama source batch) y también la envía a la lambda (a esta información se la llama event batch). La lambda puede recibir cientos de event batch, según el tiempo que tarde en procesar cada evento, por lo que hay que tener en cuenta el tiempo máximo de ejecución de la lambda de 15 minutos.

Como no se envían eventos, sino que se leen del origen, el Event Source Mapping requiere permisos para interactuar con el origen. El Event Source Mapping utiliza los permisos del execution rol de la lambda al que envía el Event Batch.

Los eventos fallidos pueden ser enviados a una DLQ. Por ejemplo, cola SQS o topic SNS.

### Cold y Warm start

Al invocarse una lambda:

- Se crea y configura el execution context.
- Se descarga e instala el runtime, ejemplo Python 3.11.
- Se descarga e instala el deployment package de la lambda

Estos pasos se almacenan en el execution context.

Realizar estos pasos se llama cold start, lo que requiere cientos de milisegundos. Si una lambda se vuelve a ejecutar en poco tiempo, puede (no es algo seguro) que use el mismo execution context por lo que se ahorran los pasos anteriores, esto es un warm start y la lambda se ejecutaría en cuestión de milisegundos.

Los datos y los environments no se mantienen entre invocaciones. Hay situaciones en que se usa el mismo runtime environment, pero como norma general, se crea uno nuevo. Los context los va eliminando AWS.

Ejecuciones concurrentes utilizan distintos execution context y seguramente sean nuevos.

Puede activase el provisioned concurrency, con el que AWS crea y mantiene unos contexts listos para usar.

Hay otras opciones para mejorar el rendimiento, como usar el execution context entre distintas invocaciones de la lambda, pero no tenemos la certeza de que las invocaciones vayan a usar el mismo invocation context por lo que la lambda debe estar preparada para generar lo que sea necesario de nuevo. Ejemplo:

- Guardar en la ruta `/tmp/` datos y así no hay que descargarlos.
- Iniciar las conexiones a base de datos fuera del lambda handler.

## Versiones

Una lambda puede tener distintas versiones.

Una versión es el código más la configuración de la lambda.

Una vez publicada una versión, no puede modificarse, cada una tiene su propio ARN (Amazon Resource Name).

Existe la variable `$Latest` que apunta la última versión.

Puede crearse alias que apunten a distintas versiones de la lambda. Ejemplo de alias: dev, stagre, prod. Estos alias pueden modificarse.

