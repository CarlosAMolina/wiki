# Networking

## Protocolos

### ARP

El protocolo ARP (Address Resolution Protocol) obtiene la dirección MAC de una dirección IP.

### NAT

El protocolo NAT (Network Address Translation) permite traducir una dirección IP Pública en una Privada.

Hay varios tipos:

- Static NAT: cada dirección IP privada tiene asignada una única dirección pública (fija). Así funciona Internet Gateway (IGW) de AWS.
- Dynamic NAT: traduce 1 dirección IP privada en la primera IP pública disponible.
- Port Address Translation (PAT): varias direcciones IP privadas se traducen en una IP pública, utiliza los puertos para diferenciar cada equipo de la red privada. Así funciona por ejemplo el router de nuestras casas, también es cómo en AWS funciona NATGW (NAT Gateway).
