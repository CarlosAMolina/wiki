# Route53

## Introducción

Este servicio es para la gestión de DNS, permite:

- Registrar o transferir un dominio.
- Hosting de zone files en name servers gestionados por AWS.

Un zonefile o DNS zone es una base de datos con los DNS records del dominio. Almacena registros DNS (A, AAAA, MX, ...).

Es un servicio global, tiene resilencia global.

## Hosted Zones

Los zone files en AWS se llaman hosted zones porque están hosteadas en name servers gestionados por AWS.

Cuando creas un hosted zone, AWS crea 4 name servers desde los que se puede solicitar su información.

Un hosted zone puede ser público o privado (asociados a unas VPC y solo accesibles desde ellas).

Los hosted zone tienen un coste mensual.

Si alguna vez se elimina un  hosted zone y vuelve a crearse, en la consola de AWS veremos en la sección `Hosted zones` que se han asignado unos name servers nuevos; deberemos copiarlos y actualizar la sección de `Registered domains` con estos nuevos valores.

### Public Hosted Zones

Accesible desde:

- Internet: al hacer peticiones DNS y resolver el DNS tree, se acaba solicitando información a uno de los NS del hosted zone.
- VPCs: las instancias en la VPC pueden hacerle peticiones a través del Route53, el cual tiene la dirección de la VPC + 2.

Pueden utilizarlo dominios registrados fuera de AWS.

Se cobra el hosting del zone y un pequeño coste por las queries que recibe.

### Private Hosted Zones

Funciona como el público pero solo es accedido por las VPC con las que esté asociado.

Pueden asociarse VPC por la consola de AWS, con la CLI y por API. También puede asociarse con otras cuentas de AWS pero solo mediante CLI y API.

### Split-view o split-horizon DNS

Con la técnica split-view pueden solaparse el public y private hosted zone.

Con esto se consigue dar acceso público a los registros que queramos o poder tener una versión distinta para uso interno y externo; por ejemplo una página web diferente si se accede desde fuera de nuestra compañía.

Para ello los hosted zones tienen el mismo nombre y registros; no es necesario que todos los registros sean iguales, solo aquellos que queramos que sean accesibles públicamente.
