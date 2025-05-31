# AWS Redshift

## Introducción

Es un data warehouse escalable que gestiona petabytes de datos. Un warehouse es donde distintas bases de datos guardan su información para luego poder analizarla.

Enfocado a analizar los datos, no a hacer las típicas peticiones sobre ellos que se hacen en una db.

Es una base de datos OLAP (Online Analitical Processing), la cual está basada en columnas, utilizada para queries complejas donde se analiza información. El otro tipo es OLTP (como RDS), guarda la información en filas y utilizada para aplicar operaciones en tiempo real.

Provisionarlo es como RDS, lo inicias y cuando no es necesario puedes pararlo. No es serverless, utiliza un servidor que hay que provisionar.

Se integra con otros servicios de AWS. Puede obtener los datos de S3, DynamoDb, migrarlos con DMS, o de otros productos de AWS.

Se puede interactuar con él con una interfaz de tipo SQL mediante conexiones JDBC/ODBC.

Funciona en una AZ en una VPC. No es high available por defecto, aunque los datos se replican en un nodo adicional y hay snapshots a S3 manuales o automáticos cada cierto tiempo o cada cierta cantidad de datos; gracias a S3, puede replicarse a otros lugares.

Formado por:

- Nodo líder. Los programas cliente interactúan con el, distribuye las peticiones y queries a ejecutar a los compute node.
- Nodo compute. Almacenan los datos y ejecuta las queries. Cada nodo está dividido en slices entre los que se reparte la memoria y almacenamiento del nodo y realizan las operaciones en paralelo.

## Redshift Sepectrum

Permite hacer queries a S3; necesitas tener un cluster de Redshfit pero no cargas la información en el.

## Federated query

Puedes solicitar fuentes en distintos orígenes remotos. Permite acceder a otras db a través de Redshfit.

## Enhanced VPC Routing

Se conecta a los servicios externos o públicos mediante conexiones públicas. Para tener más control de seguridad (por ejemplo aplica ACL), activamos esta opción y se hace uso de la red VPC.
