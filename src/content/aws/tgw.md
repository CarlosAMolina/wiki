# TGW

## Introducción

TGW = Transit Gateway

Es un Network Transit Hub que conecta VPCs y redes on premise.

Es high available y escalable.

Con la creación de `attachments`, TGW se conecta a otras redes. Attachments válidos son:

- VPC.
- VPN.
- Direct connect.

Está enfocado a reducir la complejidad de redes en AWS. Por ejemplo, si tenemos 4 VPCs y queremos tener high availability:

- Cada una de las VPCs tendrá que tener una conexión a cada una de las otras VPCs. En este ejemplo harían falta 6 conexiones.
- Cada VPC tendrá una conexión a cada CGW (customer Gateway). Para tener high availability, será necesario más de un customer gateway. En el ejemplo harían falta 4 conexiones por cada CGW.

Con el TGW:

- Habrá una conexión entre el TGW y cada CGW.
- Habrá una conexión entre el TGW y cada VPC. En el ejemplo habría 4 conexiones. Esto es gracias a que TGW soporta transitive routing, lo que permite conectar todas las VPC entre sí sin necesidad de conectar cada VPC con el resto como en el ejemplo que vimos donde no se utilizaba TGW.

El TGW puede conectarse:

- A otros TGW en diferentes cuentas de AWS y distintas regiones. Esto permite crear redes globales.
- Puede integrarse con direct connect gateway utilizando un transit VIF.
