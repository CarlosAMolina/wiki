# Regions, Edge Locations y Availability Zones

## Introducción

La infraestructura de AWS está formada por un conjunto de infraestructuras individuales situadas a lo largo del mundo.

## Regions

Se trata de una zona seleccionada por AWS que contiene toda la infraestructura de AWS (storage, db, compute, etc.); no tiene porqué coincidir con una región geográfica, es decir, en un país puede haber varias regiones de AWS.

AWS va creando nuevas regiones.

A menos que un servicio de AWS sea global como IAM, cuando lo utilizas, lo estás empleando dentro de una región específica, por ejemplo EC2.

Beneficios de las regiones:

- Están aisladas unas de otras. Si una región falla, el resto funciona. También ofrecen resilencia por lo que si en una región B hay un espejo de una región A, la región B funcionará aunque falle A.
- A la región se le aplican las leyes del país en el que se encuentre; puedes seleccionar el que más te interese. Puedes configurar que la información no vaya a otras regiones.
- Puedes elegir la región más cercana a tus clientes para tener mejor velocidad. También se puede duplicar una región en otras.

Las regiones se identifican de 2 maneras:

- Region code: por ejemplo ap-southeast-2. Es como se muestra al utiliza la CLI o APIs de AWS.
- Region name: por ejemplo Asia Pacific (Sydney). Es como se muestra en la consola de AWS.

## Edge locations

Como puede que un país no tenga una región de AWS, Amazon crea Edge Locations para que los usuarios estén más cerca de la infraestructura de AWS y así obtener más velocidad.

Son más pequeños que una región, pueden ser por ejemplo 1 o más racks en un data center de terceros.

Están distribuidos globalmente, se encuentran en más lugares que las regiones.

Suelen tener solamente servicios de distribución de contenido. No pueden emplearse para desplegar instancias EC2.

A día de hoy hay más de 200 edge locations.

## Availability Zones

Su abreviatura es AZs.

En cada región, AWS ofrece múltiples AZs. Es una división lógica dentro las regiones, no se traba de diferentes data centers, una AZ puede ser parte de diferentes data centers.

Por ejemplo, la región ap-southeast-2 puede tener estas AZs:

- ap-southeast-2a
- ap-southeast-2b
- ap-southeast-2c

Se caracterizan porque:

- La infraestructura está aislada entre AZs. En caso de fallo el resto no se ve afectado.
- Pueden desplegarse los servicios en distintas AZs para tener resilencia para que si una falla no afecte a los otros servicios; también pueden conectarse unas AZs con otras con VPC.

Transmitir datos entre diferentes AZ (por ejemplo entre instancias EC2) tiene un pequeño coste; si las instancias están en la misma AZ y usan IP privadas, no tiene coste.

## Resilencia

En AWS hay distintos niveles de resilencia:

- Resilencia global: hay pocos servicios con este tipo de resilencia. Significa que el servicio opera globalmente en una misma base de datos y sus datos se replican en distintas regiones por lo que el fallo de una región no afecta al servicio, para que el servicio deje de funcionar deberían fallar todas las. Ejemplo de estos servicios: IAM y Route 53.
- Resilencia en una región: la tienen los servicios que operan en una región. Cada servicio en distintas regiones actúa como un servicio distinto. Los datos se replican en las AZs de la región. Esto significa que si un AZ falla, el servicio funcionará, para que deje de hacerlo, deberían de fallar todas las AZs.
- Resilencia AZ: son servicios que se ejecutan solo en una AZ, fallarán en caso de error de la AZ.
