# Zona pública y privada

## Servicios públicos y privados

Para acceder a un servicio, hay que tener en cuenta que afecta el punto de vista de networking y de permisos. Pero a la hora de hablar de servicios públicos y privados, solo nos fijamos en la parte de networking.

- Un servicio público es aquel que utiliza endpoints públicos, cualquier con conexión a Internet puede acceder a estos servicios. Ejemplo el servicio S3.
- Los servicios privados están en una VPC. Para acceder a ellos hay que tener acceso a la VPC.

### Diferentes network zones

#### Internet público

Es al que puede acceder todo el mundo con conexión a Internet, por ejemplo el servicio Gmail.

#### Zona AWS pública

Funciona entre la zona pública de Internet y la zona privada de AWS.

No es parte de la zona de Internet pública, sino que tiene conexión a Internet.

En esta zona se ejecutan los servicios públicos.

#### Zona AWS privada

Son las VPC.

Las VPC están aisladas unas de otras, de Internet y de la zona AWS pública a menos que se configure lo contrario:

- VPNs o AWS Direct Connect: para que recursos externos accedan a las VPC.
- Asignar IGW a una VPC: para que la VPC acceda a Internet y a servicios públicos de AWS. En este caso el tráfico de red no no viaja por el Internet público, sino por la zona de AWS pública.
- Asignar IGW y que este de una IP pública a una VPC: para acceder al VPC desde Internet público. Lo que hace es proyectar el servicio privado en la zona AWS pública.
