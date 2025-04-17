# Glue

## Introducción

Es un servicio serverless para realizar ETL.

Mueve y modifica datos de un origen a un destino.

Ejemplo de origen:

- Almacenamiento: S3, DB.
- Streams: Kinesis Data Stream, Apache Kafka.

Ejemplo de destino: S3, DB.

Otro servicio de AWS para gestionar ETL es datapipeline, pero este requiere crear clusters EMR (son servidores) y es menos económico que  Glue.

## AWS Glue Data Catalog

Un data catalog es una colección de metadatos sobre los que pueden realizarse gestión y búsquedas.

Almacena de manera persistente metadatos de los data sources.

Hay un catálogo por región y cuenta de AWS.

Evita silos, es decir, que los datos que gestiona un departamento de una organización dejen de estar visibles para otras partes de la organización.

## Combinación con otros servicios de AWS

Otros servicios enfocados a los datos pueden usar Glue, por ejemplo Athena.

Para ello se configuran unos crawlers a los que se les dan credenciales para trabajar con las fuentes de los datos.

Los crawlers determinan el esquema de las fuentes y crean metadatos en el data catalog; lo que permite que sea visible para toda la organización.

## Glue jobs

Son procesos ETL que toman datos de una fuente y los llevan a un destino, pudiendo aplicar transformaciones con scripts que creemos.

Pueden iniciarse manualmente o con EventBdridge.
