# ELB

ELB = Elastic Load Balancer

## Introducción

Acepta conexiones de los clientes y las distribuye en las distintas partes de la infraestructura.

Permite que la infraestructura cambie (aumente, disminuya, repare recursos caídos) sin que el usuario lo note.

Nota. En la sección EC2, ver la parte de ASG para más información sobre ELB.

En la consola de AWS, los load balancers se configuran en la sección de EC2.

## Tipos

Hay 2 versiones de ELB:

- v1. Conviene evitarlo y utilizar v2, salvo en algunas excepciones. Los v1 no gestionan la layer OSI 7 de manera completa y tienen menos funcionalidad que v2, por ejemplo, 1 certificado SSL por Load Balancer lo que implica tener que utilizar múltiples Load Balancers; tampoco permite que las instancias conectadas a él escalen.
- v2. Mas rápidos y baratos. Ofrecen funcionalidades como target groups (es un conjunto de instancias; además, pueden escalar) y reglas para utilizar un mismo load balancer para distintas funcionalidades como cambiar el comportamiento según los clientes que lo usen, por ejemplo tener rules para distintos dominios, cada rule puede tener un certificado SSL y cada rule apunta a un target group.

Hay 3 tipos de ELB:

CLB (Classic Load Balancer):
- Es v1.

ALB (Application Load Balancer):

- Es v2.
- Soporta HTTP, HTTPS y WebSocket.
- Al trabajar con la layer 7 puede tatar con componentes de esta como content type, cookies, headers, health checks, etc.
- La conexión SSL termina en el ALB, luego se genera una nueva del ALB a la aplicación; es decir la conexión SSL se interrumpe entre el cliente y la aplicación.
- Debe tener certificados SSL para utilizar HTTPS.
- Más lentos que NLB al haber más niveles de red que gestionar.
- Utilizan reglas:
  - Las reglas son conexiones directas que llegan a un listener.
  - Las reglas se gestionan en orden de prioridad y se configuran con condiciones en los headers, paths solicitados, ip, etc.
  - Realizan acciones: redirigir petición, authenticación, etc.

NLB (Network Load Balancer):

- Es v2.
- Trabaja con la layer 4 por lo que soporta TCP, TLS y UDP. Por ello se utiliza en aplicaciones que no emplean HTTP(S), por ejemplo servidores de email, ssh game server o app financieras que no usen http(s).
- Son muy rápidos, pueden gestionar millones de peticiones.
- Pueden implementar health checks pero basados en ICMP / TCP.
- Pueden tener una IP estática. Esto es útil para añadir a lista blanca.
- No corta las conexiones SSL desde el cliente a la aplicación.
- Son muy utilizados con private link para dar servicios a otras VPC.

Si la aplicación no necesita una de las características que ofrece NLB, es mejor utilizar ALB porque ofrece más funcionalidades.

## Configuración

- Elegimos si utilizar solo IPv4 o IPv4 e IPv6.
- Que se ejecute en 2 o más AZ, en cada AZ elegimos 1 subred. Entonces se sitúa uno o más nodos de load balancer en una subred de cada AZ, los nodos escalan con la carga.
- Cada ELB se configura con un DNS A record que resuelve a los nodos del ELB.
- Modo:
  - Internet facing: los nodos tendrán IP pública y privada. Estos nodos pueden acceder a instancias EC2 públicas y privadas de AWS; por lo que no es necesario que las instancias EC2 sean públicas para acceder a ellas desde el exterior.
  - Internal: los nodos solo tendrán IPs privadas. Se utilizan para conectar distintos tiers de un aplicación, de modo que están desacopladas; no se conectan a algo específico sino que lo hacen al ELB y así la estructura de cada tier puede cambiar sin impacto.
- Sus nodos se configuran con listeners, que aceptan tráfico en un puerto y protocolo y se comunican con un puerto y protocolo específico.
- Necesitan que cada subred tenga mínimo 8 IPs libres, además hay que tener en cuenta las 5 IPs adicionales que se reservan en una subred. AWS recomienda utilizar mínimo una /27 para que el ELB pueda escalar (en un examen /28 también es respuesta válida si preguntan el mínimo).

## Distribución de las conexiones

A cada nodo le llega el mismo porcentaje de tráfico.

Con cross-zone LB se consigue que los nodos ELB puedan enviar tráfico a otras AZ.

De no tener cross-zone LB activado, los nodos solo pueden enviar tráfico en su AZ. Como todos los nodos reciben el mismo porcentaje de tráfico, si un nodo tiene menos instancias que otro, estas terminarán gestionando más carga; gracias a cross-zone LB evitamos esto.

## SSH

Modos en que ELB puede gestionar SSL:

- SSL Bridging
- SSL Pass Through
- SSL Offloading

### SSL Bridging

Modo por defecto.

Listener configurado para HTTPS.

La conexión termina en el ELB, por lo que el ELB debe tener un certificado SSL que matchee el dominio de la aplicación. Esto significa que AWS tiene acceso al certificado.

Tras el ELB, se inicia una nueva conexión SSL hacia las instancias backend. Las instancias necesitan tener también el certificado SSL del ELB y realizar operaciones criptográficas.

Por tanto, el ELB puede ver el tráfico HTTP, no está cifrado por SSL entre que se termina y se crea otra conexión; de este modo puede tomar decisiones en base a su contenido.

### SSL Pass Through

ELB no termina la conexión SSL; pasa la petición tal cual a las instancias.

Las instancias deben tener el certificado SSL requerido.

Al no estar el certificado en el LB, solo en las instancias que gestionamos nosotros, el certificado no es visible para AWS.

Este tipo de LB es NLB (Network Load Balancer).

Este LB utiliza TCP por lo que toma decisiones en base a las IPs.

### SSL Offloading

La conexión SSL termina en el ELB y, a diferencia de SSL Bridging, el ELB envía las conexiones a las instancias sin cifrar, envía por HTTP.

## Session/Connections stickiness

Se emplea en aplicaciones en las que la sesión del usuario se gestiona desde la instancia.

De no utilizar session stickiness, el LB envía las conexiones a todas las instancias disponibles, y si la sesión del usuario se gestiona en la instancia, cada vez que una nueva instancia se utilice, el usuario deberá iniciar sesión.

Los ELB permiten activar el session stickiness por el que el ELB genera al usuario una cookie llamada `AWSALB`. Esta cookie se encarga de las conexiones vayan a la misma instancia.

Podemos configurar la cookie para que expire entre 1 segundo y 7 días. Al expirar la cookie, el usuario debe logearse de nuevo.

Si la instancia falla, las conexiones irán a una nueva instancia.

El problema con este método es que siempre se usará una instancia aunque el usuario genera mucha carga. Por lo que se recomienda que la sesión del usuario no se gestione en la instancia sino en algo externo, por ejemplo DynamoDB; así las instancias serán serverless y puede distribuirse la carga.

La opción `stickiness` se configura en la consola de AWS desde EC2 > Target Groups; se activa para las instancias que pertenezcan al target group.
