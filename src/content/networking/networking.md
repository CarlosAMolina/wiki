# Networking

## Firewall

### Stateful vs stateless firewalls

Cuando un cliente se conecta a un servidor, hay petición del cliente al servidor y respuesta del servidor al cliente. Estas peticiones se definen como petición outbound y petición inbound, qué petición es cada cual depende de dónde nos encontramos:

- Para el cliente, la petición inicial que envía al servidor es outbound y la respuesta del servidor es inbound.
- En cambio, para el servidor la petición inicial enviada por el cliente es inbound y la respuesta que le da es outbound.

Los firewalls de tipo sateless no pueden relacionar esta petición inicial con la respuesta, por lo que hay que definir dos reglas para cada conexión, una para peticiones inbound y otra para outbound. También hay que tener en cuenta que la petición del cliente al servidor es a un puerto conocido (ejemplo 443) por lo que en la regla sabemos qué puerto configurar; pero el puerto utilizado por el cliente y al que debe llegar la respuesta es un puerto efímero (su rango depende del sistema operativo, por ejemplo puede ir de 1024 a 65535), por lo que la regla que incluye los puertos efímeros debe especificar todo el rango.

En los firewalls de tipo stateful la configuración es más sencilla al tener que configurar solamente la petición (inbound u outbound), la regla para la respuesta (inbound u outbound) se genera automáticamente, ya que estos firewalls relacionan la petición con la respuesta. Tampoco necesitan configurar el rango entero de puertos efímeros porque el firewall sabe cuál se ha utilizado.

## Protocolos

### ARP

El protocolo ARP (Address Resolution Protocol) obtiene la dirección MAC de una dirección IP.

### DHCP

Dynamic Host Configuration Protocol.

Es para asignar automáticamente IPs a los hosts.

### NAT

El protocolo NAT (Network Address Translation) permite traducir una dirección IP Pública en una Privada.

Hay varios tipos:

- Static NAT: cada dirección IP privada tiene asignada una única dirección pública (fija). Así funciona Internet Gateway (IGW) de AWS.
- Dynamic NAT: traduce 1 dirección IP privada en la primera IP pública disponible.
- Port Address Translation (PAT): varias direcciones IP privadas se traducen en una IP pública, utiliza los puertos para diferenciar cada equipo de la red privada. Así funciona por ejemplo el router de nuestras casas, también es cómo en AWS funciona NATGW (NAT Gateway).
