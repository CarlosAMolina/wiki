# AWS Direct Connect

## Introducción

DX = Direct Connect.

Direct connect nos habilita un puerto para conectarnos desde nuestra red a la localización donde se encuentra AWS Direct Connect. Mediante un cable de fibra óptica conectamos nuestro router con el router de la parte AWS Direct Connect.

Hemos de imaginar 3 localizaciones:

- Nuestra red.
- Una localización de AWS Direct Connect, es como un data center. En ella está:
  - AWS Direct Connect Cage. Aquí está el AWS DX Router al que puede conectarse un cliente, puede haber varios routers.
  - Customer Cage o Comms Partner Cage. Esta cage tiene uno o más routers. Si el cliente no tiene un cage, lo alquila a un Partner que permite conectar su router con el router en la red del cliente. Cada router on premise del cliente está conectado a un solo router de esta cage.
- Una región de AWS, esta región está conectada con una o más Direct Connect locations.

Cada router en AWS Direct Connect Cage está conectado con un router del customer cage mediante un link llamado cross-connect; es una conexión uno a uno, un router no puede estar conectado a más de un router.

Debemos configurar interfaces virtuales sobre esta conexión física, llamadas VIFs. Hay 3 tipos:

- Transit VIFs.
- Public VIFs. Para acceder a los servicios públicos de AWS.
- Vifs privadas. Para acceder a VPCs en AWS y así a servicios privados de AWS.

Ni las VIFs privadas ni las públicas ofrecen cifrado. Para conseguirlo, podemos utiliza VIFs + IPSec VPN, explicado en su propia sección.

Gracias a estas interfaces, nos conectamos a AWS sin pasar por los ISP. No sirve para acceder a internet.

Ofrece baja y constante latencia, y altas velocidades. Tenemos 3 velocidades disponibles: 1, 10 o 100 Gbps.

Puede tardar entre semanas o meses tener la conexión de nuestro router a AWS Direct Connect.

## Resilencia

La conexión entre la región AWS y el Direct Connect location es high available.

La parte entre el direct connect location y la del cliente no ofrece resilencia, para obtenerla, hay que emplear:

- Varias DX locations.
- Varios routers en cada DX location; varios en la parte de AWS y del customer.
- Varias redes de cliente on premise.
- Evitar utilizar la misma conexión física.

## Coste

Por hora que está habilitada la conexión y por tráfico de salida.

## VIFs + IPSec VPN

Como se dijo, ni las VIFs privadas ni las públicas ofrecen cifrado. Para conseguirlo, podemos utiliza VIFs + IPSec VPN.

Con esto conseguimos una conexión cifrada end to end desde el CGW (customer wateway) a VPCs (al AWS VGW, virtual private gateway, o TGW, transit gateway).

Se utiliza una interfaz VIF pública ya que las privadas solo dan acceso a IPs privadas, no a recursos públicos de AWS como VGW o TGW.

Las VPN por sí solas viajan por el internet público, lo que implica realizar saltos. La conexión es entre el router del cliente y la zona pública de AWS.

De utilizar VPN + DX, se accede a los mismos endpoints VPN públicos, y utiliza un VIF público, lo que da menor latencia y conexión directa. Pese a utilizar una VIF pública, crea una conexión privada a la VPC.

Usando el VIF + VPN podemos conectarnos a VPNs en otras regiones de AWS a través de la red global de AWS, lo que es muy útil para tener tráfico global cifrado entre las VPC y la red de cliente.

### VPN vs MACsec

Realizan funciones distintas, una no excluye a la otra.

VPN tiene más carga de cifrado que MACsec lo que afecta a la velocidad al verse más limitada por le hardware; MACsec es más rápido.

MACsec va desde el AWS DX router hasta donde termina el cross-connect.

Es más sencillo encontrar un router compatible con VPN e IPSec que un switch compatible con MACsec.
