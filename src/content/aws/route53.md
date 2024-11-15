# Route53

## Introducción

Este servicio es para la gestión de DNS, permite:

- Registrar o transferir un dominio.
- Hosting de zone files (un zonefile o DNS zone es una base de datos con los DNS records del dominio) en name servers gestionados por AWS.

Es un servicio global, tiene resilencia global.

## Hosted Zones

Los zone files en AWS se llaman hosted zones porque están hosteadas en name servers gestionados por AWS.

Cuando creas un hosted zone, AWS lo guarda en 4 name servers.

Un hosted zone puede ser público o privado (asociados a unas VPC y solo accesibles desde ellas).

Los hosted zone tienen un coste mensual.

Si alguna vez se elimina un  hosted zone y vuelve a crearse, en la consola de AWS veremos en la sección `Hosted zones` que se han asignado unos name servers nuevos; deberemos copiarlos y actualizar la sección de `Registered domains` con estos nuevos valores.
