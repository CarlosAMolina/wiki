# Networking

## Firewall

### Stateful vs stateless firewalls

Cuando un cliente se conecta a un servidor, hay petición del cliente al servidor y respuesta del servidor al cliente. Estas peticiones se definen como petición outbound y petición inbound, qué petición es cada cual depende de dónde nos encontramos:

- Para el cliente, la petición inicial que envía al servidor es outbound y la respuesta del servidor es inbound.
- En cambio, para el servidor la petición inicial enviada por el cliente es inbound y la respuesta que le da es outbound.

Los firewalls de tipo sateless no pueden relacionar esta petición inicial con la respuesta, por lo que hay que definir dos reglas para cada conexión, una para peticiones inbound y otra para outbound. También hay que tener en cuenta que la petición del cliente al servidor es a un puerto conocido (ejemplo 443) por lo que en la regla sabemos qué puerto configurar; pero el puerto utilizado por el cliente y al que debe llegar la respuesta es un puerto efímero (su rango depende del sistema operativo, por ejemplo puede ir de 1024 a 65535), por lo que la regla que incluye los puertos efímeros debe especificar todo el rango.

En los firewalls de tipo stateful la configuración es más sencilla al tener que configurar solamente la petición (inbound u outbound), la regla para la respuesta (inbound u outbound) se genera automáticamente, ya que estos firewalls relacionan la petición con la respuesta. Tampoco necesitan configurar el rango entero de puertos efímeros porque el firewall sabe cuál se ha utilizado.

### Firewall y OSI layers

Un firewall que trabaje con la capa X, puede analizar los valores de las layers inferiores y de esa misma layer, pero no de las superiores.

Por ejemplo, un firewall level 7, puede analizar los protocolos de este nivel, de la layer 3, 5, etc. Por ejemplo, para HTTPS puede ver DNS, cabeceras, etc., y en el firewall se cortará la comunicación segura, analizará su contenido y vuelve a cifrar la comunicación; aunque desde el punto de vista del cliente, la conexión está cifrada hasta el servidor objetivo. Además de analizar el tráfico, puede bloquearlo, modificarlo, etc.

## Protocolos

### ARP

El protocolo ARP (Address Resolution Protocol) obtiene la dirección MAC de una dirección IP.

### BGP

BGP = Border Gateway Protocol

Es un path-vector protocol, comparte el path más corto entre diferentes destinos, este path se llama ASPATH.

Utiliza el protocolo TCP, puerto 179.

Corrige errores que se produzcan en la red.

Los destinos que gestiona se llaman AS (Autonomous System), esos pueden ser una gran red o un conjunto de routers, pero en cualquier caso, gestionados por una sola entidad. Para BGP, cada AS es una caja negra.

Cada AS recibe un número dado por IANA, llamado ASN. Su valor va de 0 a 65.535, siendo 64-12 al 16.534 privados.

El ASPATH está formado, por el ASN destino e `i` que indica la propia red. Ejemplo:

- ASPATH si el destino es la propia red: i.
- ASPATH si el destino otra red conectada directamente y con ASN 201: 201, i.
- ASPATH si el destino otra red no conectada directamente: 202, 201, i.

Como el path más corto que utiliza BGP no tiene por qué ser el que ofrece mejor velocidad, podemos emplear `AS Path Prepending` para dar preferencia a otros paths. Consiste en modificar artificialmente el ASPATH, por ejemplo convertir el `201, i` en `201, 201, 201, i`, así es más largo que otra ruta más rápida como puede ser `202, 201, i`

Hay dos tipos:

- iBGP: internal BGP. Conecta redes dentro de un AS.
- eBGP: external BGP. Conecta AS entre sí.

### DHCP

Dynamic Host Configuration Protocol.

Es para asignar automáticamente IPs a los hosts.

### HTTP

- [Servidor web para simular respuestas a peticiones - httpbin](https://httpbin.org/).

### IPSEC

Es un conjunto de protocolos.

Crea túneles seguros entre redes no seguras. Proporciona autenticación y cifrado.

Para establecer los túneles hay dos fases:

Fase 1:

- Genera en cada peer la misma clave simétrica mediante un proceso pesado que utiliza cifrado asimétrico.
- Se crea un túnel sobre el que se negociará el IPSEC SA (security association).

Fase 2:

- Es un proceso más rápido que emplea lo generado en la fase 1.
- Los peers negocian el IPSEC SA para autenticar el tráfico que irá por el túnel. Utiliza la clave simétrica generada en la fase anterior para crear otra key simétrica con la que cifrar lo enviado, IPSEC key.
- También genera otro túnel, este túnel es creado sobre el de la fase 1.
- Esta nueva clave y túnel funcionan cuando hay que transmitir tráfico de interés, es decir, tráfico que se detecta como que debe ser cifrado; en los momentos en que no hay tráfico de interés que enviar, se elimina este túnel y vuelve a crearse cuando sea necesario.

Hay dos tipos de VPN:

- Policy-based. Utiliza reglas para identificar el tráfico de interés. Pueden utilizarse diferentes matcheos y configuraciones de seguridad, lo que da más seguridad; por ejemplo un túnel de fase 2 para el tráfico del departamento financiero, otro para el tráfico de las cámaras de seguridad, etc. Hay varios SA pair, uno por túnel, y un solo IPSEC key.
- Route-based. Identifica qué cifrar mediante prefijos. Utiliza una sola SA Pair y un solo IPSEC key.

### NAT

El protocolo NAT (Network Address Translation) permite traducir una dirección IP Pública en una Privada. Son unos procesos que modifican la dirección IP origen o destino de los paquetes de red.

En IPv6 no se usa NAT para traducir IP privada a pública.

Hay varios tipos:

- Static NAT: cada dirección IP privada tiene asignada una única dirección pública (fija). Así funciona Internet Gateway (IGW) de AWS, cuando un paquete atraviesa el IGW, se cambia la IP privada por la IP pública y cuando vuelve realiza el proceso inverso.
- Dynamic NAT: traduce 1 dirección IP privada en la primera IP pública disponible.
- Port Address Translation (PAT): varias direcciones IP privadas se traducen en una IP pública, utiliza los puertos para diferenciar cada equipo de la red privada. También se llama `IP masquerading`. Así funciona por ejemplo el router de nuestras casas o en AWS el NATGW (NAT Gateway). Como direcciones IP privadas se esconden tras una IP pública, se pueden iniciar peticiones desde la red privada al exterior y recibir respuesta pero no se pueden iniciar peticiones de Internet a la red privada.
