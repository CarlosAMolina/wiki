# Athena

## Introducción

Es un servicio serverless que permite hacer peticiones de lectura a información almacenada en S3.

Utilizado cuando queremos hacer queries a datos a los que previamente no hemos necesitado aplicar un proceso de transformación.

Podemos trabajar con archivos de terabytes de tamaño.

Puede trabajar con información estructurada, semi estructurada y sin estructura.

Para trabajar con la información, se usa la técnica schema on read:

- Se define un esquema, donde configuramos tablas, encargadas de traducir la estructura de los datos a consultar a un formato como el de una tabla relacional en db.
- Al solicitar datos, se ajustan a este esquema, lo que permite hacer peticiones parecidas a SQL.

Pueden modificarse los datos. Estos cambios se producen durante la lectura de la información, los datos originales en S3 no cambian.

El cobro es por la cantidad de datos escaneados; no se cobra por configuración o estructura. Puedes optimizar la query para escanear menos información y así reducir estos costes.

La información obtenida puede enviarse a otros servicios.

## Tipos de datos con los que trabaja

- XML.
- JSON.
- CSV/TSV.
- Parquet.
- Logs: Apache, CloudTrail, VPC Flowlogs, ELB logs.
- Glue Data Catalog.
- Web Server Logs.
- Etc.

## Athena Federated Query

Es una extensión a Athena. Permite hacer peticiones a otros orígenes que no sean S3. Ejemplo: CloudWatch, DynamoDB, DocumentDB, RDS, etc.

Para hacer las queries utiliza data source connectors que se ejecutan en funciones lambda y su función es traducir entre el origen de los datos y Athena.

## Cómo utilizarlo

Es necesario:

- Crear un bucket de S3 donde se guardarán los resultados de las peticiones.
- Dentro de Athena creamos una base de datos en su menú de opciones.
- En el menú de Athena, creamos una tabla para definir el esquema, no se almacena información en ella, es solo para definir el esquema. En esta tabla definimos:
  - Nombre de las columnas.
  - Tipo de dato de cada columna.
  - Formato del archivo origen.
  - Localización del archivo(s) origen.

## Referencias

- [Getting started - Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/getting-started.html).
- [Top 10 Performance Tuning Tips for Amazon Athena | AWS Big Data Blog](https://aws.amazon.com/es/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/).
- [CREATE TABLE types - Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/create-table.html).
