# VPC

## Introducción

VPC significa Virtual Private Cloud.

Es un servicio para:

- Crear redes privadas en AWS. Nada entra o sale de la red a menos que se configure.
- Conectar servicios de AWS con servicios externos a AWS (ejemplo, con otras nubes o servicios on premise).

Una VPC es una red virtual en AWS y se crea a nivel de cuenta de AWS y de región.

## VPC CDR

Es el rango de IPs que se puede utilizar en la VPC.

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

## Subred

Por defecto, en un VPC hay tantas subredes como número de AZs en la región que se encuentre.

Tienen resilencia AZ.

Una subred de una VPC se crea en una AZ y no puede cambiarse de AZ.

Una subred solo puede estar en una AZ y una AZ puede tener 0 o más subredes.

El rango de IPs asignados debe estar dentro del de la VPC y no puede solaparse con rangos de otras subredes.

Por defecto usan IPv4 y pueden utilizar IPv6, en tal caso la VPC también debe tener IPv6. En la VPC el IPv6 tiene CIDR /56 y las subredes /64 (la VPC tiene espacio para 256 subredes).

Las subredes por defecto pueden comunicarse con otras subredes de la VPC.

Hay 5 IPs reservadas en una subred que no pueden utilizarse:

- Network Address: es la primera IP disponible.
- La siguiente IP disponible después del Network Address. La utiliza el VPC Router. Es un dispositivo lógico que enruta tráfico entre subredes y dentro y fuera e la VPC de estar configurada para ello.
- La segunda IP disponible después del Network Address: reservada para DNS. La asignación de estas IPs en una VPC tienen un comportamiento un poco especial ya que se asigna para la VPC y también en cada subred.
- La tercera IP disponible después del Network Address: reservada para usos futuros.
- Broadcast Address: última IP de la subred.
