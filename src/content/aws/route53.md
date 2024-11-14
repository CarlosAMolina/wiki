# Route53

## Introducción

Este servicio es para la gestión de DNS, permite:

- Registrar dominio.
- Hosting de zone files (un zonefile es una base de datos con toda la información del dominio) en name servers gestionados por AWS.

Es un servicio global, tiene resilencia global.

## Hosted Zones

Los zone files en AWS se llaman hosted zones porque están hosteadas en name servers gestionados por AWS.

Un hosted zone puede ser público o privado (asociados a unas VPC y solo accesibles desde ellas).
