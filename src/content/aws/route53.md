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

### CNAME vs alias

Un CNAME no puede trabajar con un naked/apex domain; esto en AWS es un problema porque hay servicios que usan un nombre DNS naked, por ejemplo los ELBs.

Los alias pueden trabajar con naked/apex domains (ejemplo cmoli.es) y con dominios normales (ejemplo www.cmoli.es).

El alias es un servicio de AWS; para poder utilizarlo es necesario Route53.

En AWS si un alias apunta a un registro de AWS, las peticiones al alias no tienen coste.

El alias es un subtipo, puedes tener un alias de un registro A y de un CNAME y para poder usarlo tiene que apuntar a algo del mismo tipo:

- IP alias para: API Gateway, CloudFront, ELB, S3, etc.

### Simple routing

Con simple routing puedes configurar únicamente un registro para un nombre de hosted zone; cada registro puede tener varios valores.

Ejemplo, para el hosted zone cmoli.es, creamos con simple routing el registro www, que tendrá las IPs 1.1.1.1, 2.2.2.2 y 3.3.3.3. Cuando un cliente solicita www.cmoli.es, recibe todos estos valores y utiliza uno.

Simple routing no es compatible con healthcheck (healthcheck chequea si el objetivo está disponible).

### Health Checks

Servicio para revisar el correcto funcionamiento y rendimiento de algo.

Puede hacer revisiones utilizando:

- TCP: revisa si hay conexión.
- HTTP y HTTPS: puede revisar el tiempo y código de respuesta y también la información recibida.

El resultado de la revisión es `healthy` o `unhealthy`. Para que una revisión sea healthy, el el 18% de los checkers deben devolver estado healthy.

Tipos de revisiones:

- Endpoint: revisamos un endpoint.
- CloudWatch Alarm: el servicio reacciona a alarmas de CloudWatch.
- Calculated checks: se trata de revisar logs realizados por otras revisiones.

Pueden revisar el estado de servicios dentro y fuera de AWS.

Los chequeos son cada 30 segundos. Puede ser cada 10 pero tiene coste adicional.

## Failover Routing

Permite redirigir el tráfico que va a recurso a otro cuando el primero falla.

El fallo se detecta con health check.

Con él creamos registros con el mismo nombre. Por ejemplo, creamos otro registro www, de esta manera redirigimos el tráfico a una página estática de S3 cuando el servicio no funcione.

Por tanto, se utiliza para tener un failover activo pasivo.

## Multi value routing

Es una mezcla entre simple y failover routing.

Permite aumentar la disponibilidad ya que podemos crear varios registros con el mismo nombre. Por ejemplo 3 registros www cada uno con su dirección IP

Cada record puede tener su health check, pero si hay más de 8 checks, recibiremos 8 checks de manera aleatoria.

El DNS responde con los registros que están healthy y el usuario elije cuál utilizar. No es un reemplazo para el load balancing.
