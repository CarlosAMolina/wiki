# DNS

## TTL

La respuesta a una consulta DNS puede venir del servidor principal que contiene esa información, en este caso es una authoritative response. Pero otros servidores intermedios (resolvers) pueden cachear la información y su respuesta no es authoritative. El TTL es el tiempo en segundos que un resolver puede cachear los datos.

Es un valor importante porque si en el servidor principal se actualizan los valores, es el tiempo que los servidores resolvers tardarán en tomar el nuevo valor.

## DNS records

### NS

NS = Nameserver

Permite delegar a otro servidor la consulta de la información.

Tienen forma de URL, la cual indica el servidor al que hacer la consulta.

### A

Guarda el valor IPv4 de un host.

### AAAA

Guarda el valor IPv6 de un host.

### CNAME

CNAME = Canonical name.

Es una manera de crear alias. Por ejemplo, tenemos un registro A, y creamos los CNAMEs ftp, mail y www que apuntan al registro A, así cuando se soliciten esos CNAMEs se utilizará la IP del registro A; de este modo es más sencillo actualizar el valor IP cuando cambie, solo hay que hacerlo en un lugar.

Los CNAMEs no pueden apuntar directamente a una IP, deben hacerlo a otro nombre.

### MX

Utilizado en el envío de emails, guarda cuál es el servidor de emails de un dominio.

Está formado por dos partes:

- La prioridad, menor número significa mayor prioridad.
- Un valor que indica el servidor de emails del dominio. Puede especificarse de dos maneras:
  - Un nombre sin puntos: si nos encontramos en la zona `google.com` y el valor MX es `mail`, significa que hay que utilizar `mail.google.com`. El registro `A` tendrá un valor de `mail` que nos dirá la IP.
  - Un nombre con puntos: si nos encontramos en la zona `google.com` y el valor MX es `mail.foo.bar`, significa que hay que utilizar `mail.foo.bar`. Puede ser un valor dentro o fuera de la zona `google.com`.

### TXT

Utilizado paara dar a un servidor DNS un texto que otorga otras funcionalidades, por ejemplo para verificar la identidad del servidor.
