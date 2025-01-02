# VPC

## Introducción

VPC significa Virtual Private Cloud.

Es un servicio para:

- Crear redes privadas en AWS. Nada entra o sale de la red a menos que se configure.
- Conectar servicios de AWS con servicios externos a AWS (ejemplo, con otras nubes o servicios on premise).

Una VPC es una red virtual en AWS y se crea a nivel de cuenta de AWS y de región.

## VPC CDR

Es el rango de IPs que se puede utilizar en la VPC.

## Subred

Por defecto, en un VPC hay tantas subredes como número de AZs en la región que se encuentre, cada subred está en una AZ.

## Tipos

### Default VPC

- Máximo 1 por región.
- Creadas por AWS.
- Usa el CIDR: 172.31.0.0/16
- Puede eliminarse y volver a crearse. Es mejor no eliminarla porque algunos servicios la usan por defecto. Cuando un servicio se ponga en producción, es mejor que no usen la Default VPC.

Algunas configuraciones que tiene por defecto:

- IGW: para tener comunicación in Internet.
- SG (Security Group) y NACL: para control del tráfico de entrada y de salida.
- Todo lo que se despliegue en esta VPC recibe una IP Pública.

### Custom VPC

- Puedes crear varias por región.
- Por defecto están aisladas de otras redes VPC, Internet y la zona de pública de AWS. En la sección de servicios y zonas públicas y privadas se comenta cómo dar y tener acceso a VPCs.

## Consola de AWS

- Ver VPCs. Elegir VPC > your VPCs: la que tenga la columan `Default VPC` a `Yes` es la Default VPC.
- Ver subredes: Elegir VPC > subnets.
- Borrar VPC: seleccionas la VPC y click en Actions > Delete VPC. Al borrar el Default VPC también se borra el IGW, SG y NACL asociados
- Crear VPC:
  - Default VPC. Seleccionar VPC > your VPCs > Actions > Create defautl VPC.
  - Custom VPC. Seleccionar VPC > your VPCs > botón Create VPC.

## Limitaciones VPC

- Red mínima que puede tener: /18 (16 IPs).
- Red máxima que puede tener: /16 (65.536 IPs).

## Diseño de una VPC

Hay que tener en cuenta las direcciones IP que ya se utilizan para evitar pisarlas, por ejemplo no debemos evitar utilizar el rango de red empleado por el default VPC.

## Tenancy

Hay dos opciones de configuración:

- Default: los recursos comparten hardware.
- Dedicated: no se comparte hardware en ningún recuro de la VPC. Cuesta más dinero.

## IPv4 e IPv6

En una VPC por defecto se usan IPs IPv4 privadas, las públicas se asignan al querer conexión fuera de la VPC.

Se puede utilizar IPv6 pero hay limitaciones en algunos servicios. En IPv6 no hay partes privadas pero aun así hay que configurar explícitamente comunicación con el exterior.

## DNS

Se trabaja con DNS mediante el servicio Route 53.

La IP del DNS es la IP base más dos. Ejemplo, si la base es 10.0.0.0, la IP del DNS es 10.0.0.2.

Opciones DNS en VPC:

- enableDNSHostnames: para que a instancias con IP públicas s3 les asigne DNS hostnames.
- enableDNSSupport: para habilitar resolución DNS en la VPC.
