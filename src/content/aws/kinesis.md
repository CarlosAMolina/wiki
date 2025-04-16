# Kinesis

## Introducción

Es un servicio de streaming de datos escalable.

Se encuentra en la zona pública de AWS. Es highly available en la región.

Utilizado para data analytics de gran cantidad de datos en tiempo real o dashboards.

Los datos pueden leerlos diferentes consumers. Los leen cada cierto tiempo (segundos, horas, etc). Los consumers pueden ser ec2, lambdas, on permise services, etc.

Gestiona una gran cantidad de datos de diferentes aplicaciones (producers). Los producers pueden ser ec2, servicios on premise, IoT, etc.

Los producers envían datos a un `kinesis stream`. Pueden ser diferentes servicios (de AWS o de terceros). Ejemplo: CloudWatch, IoT, etc.

Kinesis es un servicio en tiempo real. Pueden consumirse los datos del kinesis stream en tiempo real, sobre 200 mili segundos.

Por defecto los datos se mantienen en el stream 24 horas, puede ampliarse este tiempo hasta 365 días con un coste adicional. El almacenamiento de los datos está incluido en el servicio.

El kinesis stream está formado por:

- Canales, cada uno tiene 1 MB para añadir datos y 2 MB para consumir. Escala al necesitar más capacidad añadiendo más canales, lo que incrementa el coste.
- En los canales los datos se guardan en kinesis data records que tienen un tamaño de 1 MB máximo.

## Kinesis VS SQS

Es importante conocer si el objetivo es:

- Ingesta de datos. Usar Kinesis.
- Desacoplar procesos. Seguramente la mejor opción es SQS.
- Sincronización asíncrona. Seguramente la mejor opción es SQS.

SQS:

- Suele tener un origen o un grupo de origen de los datos, y 1 consumidor o 1 grupo consumidor. No cientos de ellos.
- No hay que utilizarlo para persistencia de datos. Tras procesarlos, se eliminan.

## Kinesis data firehose

Es un servicio diferente a Kinesis.

Kinesis no puede enviar los datos de manera nativa a los destinos pero Kinesis data firehose permite enviar los datos de Kinesis a:

- Data lakes.
- Data stores. Así pueden almacenarse los datos de Kinesis de manera permanente, por ejemplo en otro servicio de AWS como S3.
- Servicios de analytics.

Los destinos permitidos son:

- Servicios, de AWS o de terceros, a los que se les envían los datos por HTTP.
- Splunk.
- Redshift. En el caso de Redshift, se utiliza S3 de manera intermedia y luego se copian los datos a Redshift.
- ElasticSearch.
- S3.

Escala de manera automática, es serverless y tiene resilencia.

Puede tomar los datos de Kinesis o del producer directamente.

El envío de los datos al destino no es en tiempo real como en Kinesis, sí que llegan en tiempo real pero la entrega es `near realtime`, tarda cerca de 60 segundos. Almacena datos y los envía al llegar a 1 MB o pasar 60 segundos; esto puede configurarse.

Permite modificar datos utilizando lambdas, se envían a una lambda y vuelven a Kinesis data firehose para ser enviados al destino. Puede almacenarse un backup de datos sin modificar en S3.

Se cobra en base a la cantidad de datos gestionados.

## Kinesis Data Analytics

Permite el procesamiento de datos en tiempo real.

Puede obtener datos de:

- Kinesis Data Streams.
- Kinesis Firehose.
- Referencias a datos estáticos de S3.

Tras procesar los datos, puede enviarlos a:

- AWS Lambda.
- Kinesis Data Streams.
- Kinesis Firehose y este a otros servicios, por lo que a los servicios finales no sería un envío en tiempo real.

Entre las fuentes y los destinos de datos, montamos un Kinesis Analytics Application, en esta aplicación:

- Se generan unas tablas llamadas:
  - `In-application input stream` que tienen los datos de Kinesis stream o firehose.
  - Tablas de referencia que apuntan a datos estáticos de S3, puede usarse para alimentar el input stream.
  - Ejemplo. En un videojuego, los datos en tiempo real se guardan en el in-application input stream, y en la reference table hay metadatos de los jugadores como sus nombres.
- El código de la aplicación son sentencias SQL que procesan los inputs. Pueden ser sentencias SQL muy complejas.
- El output del procesamiento se almacena en:
  - `In-application output streams`. Estas tablas envían la información a servicios fuera del Kinesis Analytics Application.
  - `In-application error stream`. Estas tablas almacenan los errores en tiempo real.

Se cobra por los datos procesados por la aplicación; no es barato.

Conviene utilizarlo cuando:
- Streaming data que necesita procesamiento SQL en tiempo real.
- Time-series analytics. Ejemplo: elecciones o e-sports.
- Dahsboards en tiempo real, por ejemplo en juegos.
- Métricas en tiempo real para seguridad.


## Kinesis Video Streams

Se alimenta con vídeos (u otros datos que no son vídeo) en directo. Escala con la volumetría recibida.

Las fuentes que envían información pueden ser:

- Fuentes de vídeo. Ejemplo: cámaras de seguridad, smartphones, coches, drones, etc.
- Fuentes no de vídeo: audio, datos de temperatura, profundidad, radar, etc.

La información la envían las fuentes a Kinesis Video Streams. Por ejemplo, cada cámara de seguridad envía datos a un stream.

Se puede acceder a información de los datos frame-by-frame o como sea necesario. Pero es importante saber que no pueden accederse a los datos tal cual, ya que no son almacenados como son originalmente; se consumen mediante API.

Los datos pueden ser almacenados y cifrados (en tránsito o en tiempo real).

Suele usarse con otros servicios de AWS como Rekognition, para reconocimiento facial o Connect, para procesar audio.

Ejemplo. Cámaras de seguridad envían datos a kinesis video streams, estos los llevan a Rekognition Video y este da los datos del análisis a kinesis data stream el cual los lleva a una lambda para que tome decisiones en base a los resultados del análisis, por ejemplo según un nivel de matcheo de que las caras captadas por las cámaras corresponden a gente conocida o no.
