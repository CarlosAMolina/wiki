# Kinesis

## Introducción

Es un servicio de streaming de datos escalable.

Los datos pueden leerlos diferentes consumers. Los leen cada cierto tiempo (segundos, horas, etc). Los consumers pueden ser ec2, lambdas, on permise services, etc.

Gestiona una gran cantidad de datos de diferentes aplicaciones (producers). Los producers pueden ser ec2, servicios on premise, IoT, etc.

Los producers envían datos a un `kinesis stream`. Por defecto los datos se mantienen en el stream 24 horas, puede ampliarse este tiempo hasta 365 días con un coste adicional. El almacenamiento de los datos está incluido en el servicio.

El kinesis stream está formado por:

- Canales, cada uno tiene 1 MB para añadir datos y 2 MB para consumir. Escala al necesitar más capacidad añadiendo más canales, lo que incrementa el coste.
- En los canales los datos se guardan en kinesis data records que tienen un tamaño de 1 MB máximo.

Se encuentra en la zona pública de AWS. Es highly available en la región.

Utilizado para data analytics de gran cantidad de datos en tiempo real o dashboards.

## Kinesis data firehose

Permite almacenar los datos de Kinesis de manera permanente en otro servicio, por ejemplo S3.

## Kinesis VS SQS

Es importante conocer si el objetivo es:

- Ingesta de datos. Usar Kinesis.
- Desacoplar procesos. Seguramente la mejor opción es SQS.
- Sincronización asíncrona. Seguramente la mejor opción es SQS.

SQS:

- Suele tener un origen o un grupo de origen de los datos, y 1 consumidor o 1 grupo consumidor. No cientos de ellos.
- No hay que utilizarlo para persistencia de datos. Tras procesarlos, se eliminan.
