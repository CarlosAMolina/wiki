# VPC

## Introducción

VPC significa Virtual Private Cloud.

Es un servicio para:

- Crear redes privadas en AWS.
- Conectar servicios de AWS con servicios externos a AWS.

Una VPC es una red virtual en AWS y se crea a nivel de cuenta de AWS y de región.

## VPC CDR

Es el rango de IPs que se puede utilizar en la VPC.

## Subred

Cada subred de una VPC está en una AZ.

## Tipos

## Default VPC

- Máximo 1 por región.
- Creadas por AWS.
- Usa el CIDR: 172.31.0.0/16
- Puede eliminarse y volver a crearse. Es mejor no eliminarla porque algunos servicios la usan por defecto. Cuando un servicio se ponga en producción, es mejor que no usen la Default VPC.

Algunas configuraciones que tiene por defecto:

- IGW: para tener comunicación in Internet.
- SG (Security Group) y NACL: para control del tráfico de entrada y de salida.
- Todo lo que se despliegue en esta VPC recibe una IP Pública.

## Custom VPC

- Puedes crear varias por región.
- Por defecto están aisladas de otras redes VPC, Internet y la zona de pública de AWS. En la sección de servicios y zonas públicas y privadas se comenta cómo dar y tener acceso a VPCs.

## Consola de AWS

- Ver VPCs. Elegir VPC > your VPCs: la que tenga la columan `Default VPC` a `Yes` es la Default VPC.
- Ver subredes: Elegir VPC > subnets.
- Borrar VPC: seleccionas la VPC y click en Actions > Delete VPC. Al borrar el Default VPC también se borra el IGW, SG y NACL asociados
- Crear VPC:
  - Default VPC. Seleccionar VPC > your VPCs > Actions > Create defautl VPC.
  - Custom VPC. Seleccionar VPC > your VPCs > botón Create VPC.
