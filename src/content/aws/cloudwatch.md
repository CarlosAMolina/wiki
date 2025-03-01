# CloudWatch

## Introducción

Es un servicio público y regional, puede funcionar con software ajeno a AWS.

CloudWatch permite:

- Almacenar y acceder a logs. Dentro de CloudWatch tenemos disponible para ello a `CloudWatch Logs`.
- Obtener métricas de otros servicios y software que use AWS. Ejemplo: uso CPU, peticiones REST, etc. Algunas métricas se generan por defecto (ejemplo, uso CPU en instancias EC2) pero otras requieren instalar el CloudWatch agent.
- CloudWatch Logs: permite monitorizar logs y realizar acciones en base a ellos. Hay situaciones en las que hay que instalar el CloudWatch agent.
- CloudWatch Events: pueden ejecutarse acciones cuando ocurran sucesos, por ejemplo que un EC2 se inicie, y también pueden lanzarse eventos programados para que se realicen otras acciones.

## CloudWatch Agent

Necesario para aplicaciones externas a AWS o para trabajar con logs propios de aplicaciones o de sistemas operativos.

Por ejemplo, para tratar con los logs dentro de una instancia EC2; el CloudWatch Agent es un programa que funciona en el sistema operativo y envía los logs a CloudWatch. También permite ver el uso de la CPU, lecturas de disco, etc.

El CloudWatch Agent debe tener permisos para tratar con CloudWatch y CloudWatch Logs. También hay que darle la configuración, por ejemplo mediante SSM Parameter Store.

Se utiliza:

- Log groups: un log group por cada archivo de la instancia que queramos guardar (ejemplo: `/var/log/https/error_log`, `/var/log/https/access_log`).
- Log stream: dentro del log group, hay un log stream por cada instancia que genera logs.

Otra opción en lugar del CloudWatch agent es utilizar los kits de desarrollo de AWS.

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

## Estructura

Las siguientes estructuras de definen desde la más pequeña a otras mayores que engloban las anteriores.

### Log events

Es cómo se guardan los logs generados.

Su formato es timestamp más el mensaje. El mensaje puede tener cualquier formato que luego puede interpretarse, como por ejemplo definir campos y columnas.

### Log streams

Es donde se almacenan los logs events.

Los log streams almacenan secuencias de logs del mismo origen; por ejemplo un log stream por cada instancia de EC2.

Ejemplos de nombres que podemos configurar que tengan:

- El ID de la instancia EC2.
- La fecha + un hash (puede haber varios para una misma fecha).

### Log group

Son contenedores que almacenan distintos log streams; pero logs streams del mismo tipo, ejemplo:

- Logs del sistema operativo.
- Logs de una lambda.
- Log de cada archivo de de las instancias EC2 que queramos guardar, ejemplo indicamos los siguientes paths: `/var/log/secure`, `/var/log/https/error_log`, `/var/log/https/access_log`. A cada log group hay que darle un nombre, puede usarse el mismo del path, ejemplo `/var/log/secure`.

También almacenan la configuración de los logs: permisos, retención, filtros de métricas que pueden tener alarmas configuradas, etc.

El log group solo aparece en CloudWatch en caso de que contenga algún log.

## Precio

[URL sobre los costes](https://aws.amazon.com/es/cloudwatch/pricing/).

## Logs de una instancia EC2

Por defecto CloudWatch no puede acceder a los logs internos de una instancia EC2; es necesario el CloudWatch Agent.
