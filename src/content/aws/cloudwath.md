# CloudWatch

## Introducción

Es un servicio público, puede funcionar con software ajeno a AWS.

CloudWatch permite:

- Obtener métricas de otros servicios y software que use AWS. Ejemplo: uso CPU, peticiones REST, etc. Algunas métricas se generan por defecto (ejemplo, uso CPU en instancias EC2) pero otras requieren instalar el CloudWatch agent.
- CloudWatch Logs: permite monitorizar logs y realizar acciones en base a ellos. Hay situaciones en las que hay que instalar el CloudWatch agent.
- CloudWatch Events: pueden ejecutarse acciones cuando ocurran sucesos, por ejemplo que un EC2 se inicie, y también pueden lanzarse eventos programados para que se realicen otras acciones.

## Conceptos

### Namespace

Es un contenedor para monitorizar datos, para separar los distintos tipos de datos.

El nombre debe seguir unas reglas, hay algunos no válidos como los utilizados para los servicios de AWS que siguen la máscara `AWS/{service}` ejemplo `AWS/EC2`.

### Métrica

Una métrica es un conjunto de datos que tienen relación. Ejemplo, uso de la CPU. Hay que tener en cuenta que la métrica CPU no es específica de un servicio, sino que almacena el uso de la CPU de todos los servicios.

### Datapoint

Cada vez que un servicio envía su uso de CPU, la medida que está enviando es un Datapoint.

Está compuesto de un timestamp y de un valor.

### Dimensión

Sirve para que, en una métrica, saber a qué pertenece el Datapoint.

Por ejemplo, una máquina EC2, envía como información de la dimensión el nombre de la instancia y el tipo de instancia.

### Alarmas

Se asocian a una métrica. Identifican si la métrica indica un estado correcto o no, y en este caso pueden desencadenarse acciones.

Ejemplo, la alarma que notifica si el coste económico mensual de AWS sobrepasa un valor.
