# AppFlow

## Introducción

Permite:

- Intercambiar datos entre aplicaciones.
- Agregar datos de diferentes fuentes.
- Realizar modificaciones en los datos.

Las aplicaciones se comunican empleando conectores y flows.

Puede trabajar con:

- Endpoints públicos.
- Endpoints privados mediante PrivateLink.

Puede conectarse con aplicaciones Saas (Software as a Service) como Slack, Salesforce etc. También podemos crear nuestro propio conector.

Ejemplos de uso:

- Llevar contactos de Salesforce a AWS Redshift.
- Llevar tickets de Zendesk a S3.

## Flow

Un flow está formado por un conector origen y uno destino; puede haber otros componentes adicionales.

En el flow definimos:

- Mapeo de fuentes origen y destino.
- Opcionalmente pueden establecerse transformaciones, filtrado o validaciones de datos.

## Conectores

Los conectores almacenan la configuración y credenciales para acceder a las aplicaciones origen y destino.

Son definidos de manera independiente de los flows, pueden utilizarse en distintos flows.
