# Route53

## Introducción

Este servicio es para la gestión de DNS, permite (ver sección `Interoperability` para más detalle):

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

## CNAME vs alias

Un CNAME no puede trabajar con un naked/apex domain; esto en AWS es un problema porque hay servicios que usan un nombre DNS naked, por ejemplo los ELBs.

Los alias pueden trabajar con naked/apex domains (ejemplo cmoli.es) y con dominios normales (ejemplo www.cmoli.es).

El alias es un servicio de AWS; para poder utilizarlo es necesario Route53.

En AWS si un alias apunta a un registro de AWS, las peticiones al alias no tienen coste.

El alias es un subtipo, puedes tener un alias de un registro A y de un CNAME y para poder usarlo tiene que apuntar a algo del mismo tipo:

- IP alias para: API Gateway, CloudFront, ELB, S3, etc.

## Health Checks

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

## Route policies

### Simple routing

Con simple routing puedes configurar únicamente un registro para un nombre de hosted zone; cada registro puede tener varios valores.

Ejemplo, para el hosted zone cmoli.es, creamos con simple routing el registro www, que tendrá las IPs 1.1.1.1, 2.2.2.2 y 3.3.3.3. Cuando un cliente solicita www.cmoli.es, recibe todos estos valores y utiliza uno.

Simple routing no es compatible con healthcheck (healthcheck chequea si el objetivo está disponible).

### Failover Routing

Permite redirigir el tráfico que va a recurso a otro cuando el primero falla.

El fallo se detecta con health check.

Con él creamos registros con el mismo nombre. Por ejemplo, creamos otro registro www, de esta manera redirigimos el tráfico a una página estática de S3 cuando el servicio no funcione.

Por tanto, se utiliza para tener un failover activo pasivo.

### Multi value routing

Es una mezcla entre simple y failover routing.

Permite aumentar la disponibilidad ya que podemos crear varios registros con el mismo nombre. Por ejemplo 3 registros www cada uno con su dirección IP

Cada record puede tener su health check, pero si hay más de 8 checks, recibiremos 8 checks de manera aleatoria.

El DNS responde con los registros que están healthy y el usuario elije cuál utilizar. No es un reemplazo para el load balancing.

### Weighted routing

Se utiliza para implementar una load balancer simple o probar software.

Si tienes por ejemplo 3 registros www DNS, a cada uno le asocias un peso. Se calcula la suma de los pesos con el mismo nombre (www en este ejemplo) y cada registro se devolverá en un porcentaje igual a su peso dividido entre la suma de los pesos con su nombre (tanto healthy como unhealthy); de usar health checks, en caso de está unhealthy, se pasa al siguiente con mayor resultado en la división anterior.

Para no devolver un registro, se le asocia un peso de 0. Si todos los registros pesan 0, entonces se tienen todos en cuenta.

Esto permite probar software porque podemos asignar un peso pequeño a una nueva versión de un servicio para que no lo usen todos los usuarios.

### Latency-based routing

Permite aumentar el rendimiento y experiencia de usuario ya que Route53 devolverá los records con mejor performance.

Con el record guardamos una tag con la región de AWS en la que se encuentra y así AWS selecciona la que presenta menor latencia para el usuario. Para saberlo, guarda una base de datos con información de la latencia por región y lo compara con la localización de la IP de la solicitud.

De usar health checks, devolverá el caso con menor latencia y que esté healthy.

### Geolocation routing

Route53 devolverá el registro en base a la relevancia de la localización configurada en los registros y la localización del usuario (no en base a la cercanía ni a la latencia como vimos en latency-based policy).

Con el record guardamos una tag con su localización. La tag puede indicar (de mayor a menor relevancia):

- El estado dentro del país.
- El país.
- Continente.
- Usar la tag `default` utilizada si los registros en los grupos anteriores no matchean con la localización del usuario.

Se compara la localización del usuario con la de los registros en orden de preferencia anterior. Importante, no se devuelve el registro más cercano, sino la más relevante; también puede devolver la respuesta `NO ANSWER` en caso de que ningún registro esté configurado para tener relevancia, aunque esté cerca.

Puede utilizarse para:

- Restringir contenido por zonas.
- Devolver contenido en base al idioma.
- Load balancing según localización.

### Geoproximity routing

Ofrece a los usuarios los registros que se encuentren a menor distancia, para calcularla se tiene en cuenta el `bias`, lo vemos a continuación.

Indicas dónde se encuentra los recursos añadiendo una tag a los registros:

- Si es un servicio de AWS: indicas la región en que se encuentra.
- De no ser un servicio de AWS: indicas la latitud y la longitud.

Al calcular la distancia también se tiene en cuenta el `bias`, es un valor que añadimos a la configuración para modificar el cálculo de la distancia, puede ser positivo o negativo. Por ejemplo, si un usuario está mas cerca de un recurso situado en una zona A pero queremos que utilice un recurso que se encuentra en B; podemos aplicar un bias positivo a B para aumentar la zona a la que da servicio.

### Interoperability

Es cuando en Route53 no hacemos los siguientes puntos (parar entender el funcionamiento, ayuda ver estos puntos como funcionalidades diferentes dentro de Route53):

- Registrar el dominio.
- Hosting del dominio, a continuación indicamos qué acciones se realizan aquí.

Sino que uno de estos se realiza en Route53 y lo otro en un servicio externo a AWS.

#### Route53 como registrar y domain hosting

Al registrar un dominio en Route53:

- Actúa como el registrar aplicándonos un coste por el registro, se paga cada año o cada tres años. De estar el dominio disponible, se continúa con los siguientes pasos.
- Para el domain hosting (se realiza en la zona pública de AWS):
  - Se provisionan 4 name servers (NS).
  - Se crea un zone file, hosteado en los NS anteriores. Esto tiene un coste mensual (se le paga a Route53).
- Actúa como el domain registrar comunicando los cambios al registry del TLD que utilicemos para que añada una entrada para el dominio. En esta entrada dada al registry añade los NS que se utilicen.

#### Route53 solo como registrar

En caso de que Route53 solo realice la función de registrar, hay que aplicar otras configuraciones manuales. El dominio lo gestiona Route53 pero la provisión de los NS y el cobro mensual se hace con el proveedor externo que realice la función de hosting del dominio.

Esta arquitectura no se utiliza mucho porque dejaríamos de utiliza el hosting con Route53, y esta función ofrece más valor que la de registrar.

#### Route53 solo para domain hosting

Esta arquitectura es empleada más que la de utilzar Route53 solo como el registrar.

El tercero que haga de registrar realizará las funciones antes descritas que Route53 hace para ello.

Puede utilizarse esta arquitectura por ejemplo con un dominio ya registrado pero queremos usar el hosting de AWS.
