# CloudTrail

## Introducción

Es un producto que logea acciones realizadas en AWS.

## Limitaciones

No sirve para gestionar eventos en tiempo real. Crea los logs dentro de los 15 minutos en que ocurre la actividad.

## Términos

### CloudTrail event

Es el log de la acción realizada por un usuario, API, servicio, etc.

Tipos:

- Management events: guarda operaciones sobre recursos, como crear/terminar una instancia EC2, crear una VPC, bucket, etc., hacer login en la consola, etc. Por defecto están activados.
- Data events: operaciones realizadas sobre lo que contiene un recurso. Ejemplo: subir/leer archivo, invocar lambda, etc. Por defecto no se logean (generarían un gran volumen de logs) y tienen un coste adicional.
- Insights events: detecta errores o actividad inusual en la cuenta.

### Event history

Histórico de eventos. Por defecto tiene estas características que pueden cambiarse: está activado, retención de 90 días y no almacena en S3 (creo que esto se refiere al event history, los cloudtrail events sí usan s3 por defecto).

### Trail

Se utiliza para aplicar una configuración. Tipos de funcionamiento:

  - One region: los eventos guardados son los que ocurren en la misma región en que se creó el Trail.
  - All regions: puede interpretarse como Trails en todas las regiones gestionado como un solo Trail. Si AWS añade nuevas regiones, estas se incluyen automáticamente en el Trail.

Para los servicios globales (IAM, STS, CloudFront), es necesario que el Trail esté configurado para guardar logs de todas las regiones. Estos servicios generan Global Service Events y se guardan en la región us-east-1.

Por defecto algunos logs se almacenan en un bucket de S3 donde cobrarán el almacenamiento; la configuración permite indicar qué eventos guardar. Además de en S3, puede configurarse que se almacenen en CloudWatch, lo que permite utilizar las facilidades de este servicio.

Desde el management account de una AWS organization puedes crear un organizational trail, que incluirá todos los eventos de todas las cuentas de la organización.

## Precio

[URL sobre los costes](https://aws.amazon.com/es/cloudtrail/pricing/).
