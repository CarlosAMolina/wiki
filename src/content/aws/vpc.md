# VPC

## Introducción

VPC significa Virtual Private Cloud.

Es un servicio para:

- Crear redes privadas en AWS. Nada entra o sale de la red a menos que se configure.
- Conectar servicios de AWS con servicios externos a AWS (ejemplo, con otras nubes o servicios on premise).

Una VPC es una red virtual en AWS y se crea a nivel de cuenta de AWS y de región.

## VPC CDR

Es el rango de IPs que se puede utilizar en la VPC.

Limitaciones:

- Red mínima que puede tener: /28 (16 IPs).
- Red máxima que puede tener: /16 (65.536 IPs).

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
- Por defecto están aisladas de otras redes VPC, Internet y la zona pública de AWS. En la sección de servicios y zonas públicas y privadas se comenta cómo dar y tener acceso a VPCs.

## Consola de AWS

- Ver VPCs. Elegir VPC > your VPCs: la que tenga la columan `Default VPC` a `Yes` es la Default VPC.
- Ver subredes: Elegir VPC > subnets.
- Borrar VPC: seleccionas la VPC y click en Actions > Delete VPC. Al borrar el Default VPC también se borra el IGW, SG y NACL asociados
- Crear VPC:
  - Default VPC. Seleccionar VPC > your VPCs > Actions > Create defautl VPC.
  - Custom VPC. Seleccionar VPC > your VPCs > botón Create VPC.

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

En la consola de AWS, hay que configurar la rubred para habilitar que las IPs sean públicas.

## VPC Router

Cada VPC tiene uno.

En cada subred tiene una interfaz que posee la siguiente IP disponible después de la network address.

De no tener otra configuración, por defecto enruta tráfico entre subredes.

### Route tables

El VPC router puede configurarse con route tables; por defecto tiene una route table llamada main route table.

Cada subred puede tener una única route table y una route table puede estar asociada a distintas subredes. De no asociarle una route table a una subred, utiliza por defecto el main route table del VPC.

En caso de que un paquete matchee distintas rutas en el Route Table, el orden de prioridad a aplicar es primero el target local (tráfico va dentro de la VPC) y luego la ruta más específica (la de mayor CDR); de no matchear ninguna, se aplica el default route.

El default route hay que configurarlo en el IGW, no viene por defecto. Para IPv4 es `0.0.0.0/0` y para IPv6 `::/0`, estos rangos matchean todas las direcciones IP.

## Internet Gateway

IGW = Internet Gateway

Se utiliza en una VPC para que el tráfico pueda salir y entrar del VPC al AWS Public Zone (S3, SQS, SNS, etc.) o al Internet Público. Los recursos y servicios envían los paquetes al IGW antes de salir fuera de la VPC, por ejemplo el NAT Gateway envía las peticiones al IGW.

Tiene resilencia regional, esto implica que un solo IGW se aplica a todas las AZ de una región en que se use la VPC.

Una VPC puede tener 0 o 1 IGW y un IGW puede tener 0 o 1 VPC. Cuando no está asociado a ningún VPC, el Gateway aparece como `Detached`.

Aclaración sobre asignación de IPs. Al asignar una IPv4 pública, por ejemplo a una instancia EC2, el EC2 no recibe directamente esa IP pública, sino que en el IGW se guarda la relación entre la IP privada de la instancia y la IP pública; por eso en el sistema operativo de la instancia solo se ve la IP privada. En el caso de IPv6, se utiliza la misma IP para la parte privada y pública, por lo que el sistema operativo de la instancia conoce la IPv6 y el IGW no tiene que traducir entre IP pública y privada.

### Route table

Para que un recurso en una subred tenga acceso a Internet, hay que configurar la route table de la subred de modo que apunte al IGW. En caso de utilizar NATGW, ver explicación en su sección `Route table`.

### IGW e IPv6

De asignar al IGW una IPv6, ya tiene acceso público; no hace falta otras configuraciones como IPv4.

Al configurar la ruta ::/0 de IPv6 se da acceso bidireccional, de la VPC al exterior y viceversa.

### Egess-Only IGW

Es un tipo de gateway que funciona con IPv6 y da acceso solo al exterior de la VPC.

## Bastion Host o Jumpbox

Se trata de una instancia en una subred pública que permite acceder a recursos internos de una VPC desde fuera. Útil para VPC muy securizadas, inicialmente era el único modo de configurar instancias privadas.

Puede configurarse para que acepte conexiones filtrado por IP, o autenticación SSH u otra externa.

## NACL

NACL = Network Access Control List

Es como un firewall en un VPC; de tipo stateless (tiene reglas de inbound y outbound).

Se asocian a subredes, no pueden asociarse a otro recurso. No gestionan el tráfico dentro de la subred, solo el que sale o entra.

Cada subred puede tener una NACL, la que viene por defecto en una VPC o una personalizada:

- NACL por defecto: cuando se crea una VPC, se le asocia una NACL por defecto que permite todo el tráfico. Esto lo realiza AWS para facilitar su uso y porque prefiere que se utilicen security groups; a pesar de ello, para denegar tráfico creo que solo pueden hacerlo las NACL, por lo que se combinan con los security groups, estos para permitir tráfico y los NACL para denegarlo.
- Al crear NACLs personalizadas, se crean para un VPC e inicialmente no están asociadas a subredes. Por defecto tienen todo el tráfico denegado.

Una NACL puede esta asociada a varias subredes.

Se configura con reglas:

- Las reglas permiten o deniegan tráfico a una IP (o rango de IPs), puerto (o rango de puertos) y protocolo.
- Para determinar qué regla aplicar, se miran las reglas desde la de menor número hacia la de mayor, y se aplica la la primera que matchee. En lugar de un número, la regla puede tener un asterisco, que significa que se aplica de no matchear ninguna regla.

## Security Groups (SG)

Es como un firewall, de tipo stateful. Por lo que no hace falta configurar los puertos efímeros; al igual que los NACLs, sí que tiene reglas de inbound y outbound, pero no hay que configurar la respuesta.

Tienen la limitación de no poder denegar tráfico de manera explícita. Lo que hacen es permitir tráfico y, lo que no esté permitido queda denegado de manera implícita. Por eso no sirven para bloquear actores maliciosos, para eso se utilizan las NACLs.

Pueden configurarse indicado IP/CIDR y recursos lógicos de AWS, es decir, otros security groups o el mismo SG:

- Ejemplo a otros SG: si en una VPC tenemos una instancia de APP y otra de Web, imaginemos que la Web tiene acceso público gracias a su SG, pero la APP solo es accedida dentro de la subred por la instancia Web, para permitir que el SG del APP permita conexiones de la instancia Web, podemos configurar el SG con la IP interna de la Web o o un rango de IPs por si cambia; pero si el rango se amplía puede que no quede configurada, la solución es que el SG de la APP apunte en su configuración al SG de la Web. Así, todas las instancias con el SG de la Web asociado, tendrán acceso a las instancias que tengan asociado el SG de la App; este modo de configuración escala muy bien.
- Ejemplo referencia al mismo SG: para configurar la configuración dentro de la subred; ejemplo, si en una subred hay varias instancias y se les asocia el mismo SG, configurando este SG permite que, cuando aparezcan o remuevan instancias (ejemplo por auto scaling), la configuración es automática.

Los SG solo se asocian a Elastic Network Interfaces (ENI's); no se asocian ni a instancias ni a subredes, aunque la consola de Amazon pueda mostrarlo como si así fuera, en realidad quedan asociadas a network interfaces.

## NAT Gateway

NATGW = NAT Gateway

Con NAT damos a un recurso privado acceso de salida a Internet.

Tienen resilencia AZ. Para tener resilencia regional como el Internet Gateway, hay que desplegar un NATGW en cada AZ.

El NAT Gateway utiliza NAT de tipo estático e IP masquerading.

No funcionan con IPv6.

Para realizar su función, el Nat Gateway guarda en el translation table toda la información necesaria (IPs, puertos, etc).

AWS tiene dos maneras de proporcionar el servicio NAT:

- Históricamente se utilizaba un EC2 para esta función. Para ello, hay que desactivar en el EC2 la opción de Source/Destination checks porque por defecto, una instancia EC2 desecha cualquier paquete que no tenga como origen o destino la IP de la instancia, quitando este check se desactiva esta revisión.
- Con el servicio AWS Gateway configurado en una VPC.

Entre un EC2 y un NATGW hay estas diferencias:

- NATGW se recupera en caso de error.
- Al EC2 podemos conectarnos, ejemplo usarlo como bastion host, al NATGW no podemos conectarnos.
- En ambos podemos usar NACLs pero security groups solo se asocian al EC2; en el caso del NATGW se asocian a los recursos de la red.
- El NATGW de detectar mas tráfico escala automáticamente hasta 45 Gbps, el EC2 depende del ancho de banda del tipo de instancia.

Para tener una IP pública, debe estar en una subred pública (que asigne IP públicas). Las subredes privadas en la misma VPC que necesiten NAT pueden apuntar al NATGW de la red pública.


### NAT Gateway e IGW

El NAT Gateway envía las peticiones al IGW para salir fuera de la VPC. El Nat Gateway modifica las direcciones IP origen privadas, y asigna la suya privada, luego el IGW recibe estos paquetes y los envía a Internet, modificando en los paquetes la IP privada del NAT Gateway por la pública del NAT Gateway.

Si necesitas que una instancia tenga una IP pública usas IGW, si quieres que instancias privadas tengan acceso al AWS public zone e Internet, utilizas NAT Gateway e IGW.

### Route table

Creamos un route table para la VPC, por ejemplo, creamos un route table para cada AZ.

En cada route table añadimos un default route (0.0.0.0/0) que apunta al NATGW del AZ.

Cada route table debemos asociarlo a cada subred; porque si una subred no se le asigna una route table, por defecto usa el main route table y no utilizará el que hemos configurado.

### Elastic IP

Los NAT Gateway usan lo que se conoce como elastic IP.

- Es un tipo especial de IPv4 pública.
- Es una IP estática, no cambia.
- Está en una región de la cuenta AWS.
- Puede asociarse con lo que queramos.

### Coste

Se cobra por hora de uso y por cantidad de tráfico.
