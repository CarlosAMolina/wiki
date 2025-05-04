# AWS Site-to-Site VPN

## Introducción

Crea un IPSEC VPN entre AWS VPC y una red externa como on premise o de terceros.

## Características

- La conexión IPSEC está cifrada, utiliza el internet público a menos que creemos la VPN sobre direct connect.
- Es high availiable a menos que no esté bien configurada la parte del cliente (ver sección CGW).
- Puede provisionarse rápido, en 1 hora, al configurarse mediante software. A diferencia de Direct Connect que requiere más tiempo; ya que al ser una solución hardware, puede necesitar entre semanas y meses.
- Los routes table de AWS deben tener configuradas el rango de IPs privadas utilizadas en la red externa y viceversa.
- El coste es por hora y GB.

Tiene limitaciones que pueden hacer que usar otra alternativa, por ejemplo  AWS Direct Connect, sea una mejor opción:

- AWS tiene la limitación de velocidad de 1.25 Gbps como máximo; en la parte de cliente podría implicar una menor velocidad. Este máximo de 1.25 Gbps aplica al conjunto de todas las conexiones que usan el mismo VGW.
- Al utilizar el internet público, cada salto añade latencia.

Puede utilizarse con Direct Connect de diferentes modos:

- Como backup de Direct Connect.
- Provisionar primero VPN y luego reemplazarlo por Direct Connect cuando este último esté disponible.
- Trabajar como capa superior a Direct Connect para ofrecer cifrado.
- Trabajar junto a Direct Connect para ofrecer high availability.

En AWS podemos crear dos tipos de VPN:

- Estáticas:
  - Las rutas de la parte del cliente se guardan en las routes tables de AWS de manera estática; hay que configurarlas.
  - Redes del cliente están configuradas de manera estática en la conexión VPN.
  - No permite load balancing ni multi conexión en caso de fallo.
- Dinámicas:
  - Utiliza BGP para intercambiar información sobre el estado de la red en la parte de AWS y del cliente lo que permite seleccionar la mejor opción.
  - Ofrece high availability.
  - De activar `route propagation`, las rutas se añadirán al route table de la parte de AWS de manera automática, en caso contrario estas rutas estarán almacenadas como rutas estáticas.

## Partes

### CGW

CGW = Customer Gateway

Se utiliza para referirse a dos cosas:

- La parte de configuración lógica en AWS.
- El dispositivo físico que representa la configuración, el router al que la VPN se conecta en la red on premise.

Necesita una IP pública.

De utilizar un solo CGW, la VPN no será high available desde la parte del cliente, porque si su router falla, la VPN no funcionará.

Para solucionar esto, puede emplearse otro CGW con otra dirección IP. Habrá que crear otros VGW endpoints y que cada uno tenga una conexión a este nuevo CGW.

### Conexión VPN

Es la conexión entre VGW y CWG.

Se crea un túnel VPN entre cada endpoint del VGW y el CGW.

### VGW

VGW = Virtual Private Gateway

Es un tipo de Gateway lógico. Se crea y asocia con una VPC y es el target de una o más route tables.

Se encuentra en la AWS Public Zone.

#### Endpoints

Cada VGW tiene estos dispositivos físicos con IPv4 públicas.

Se encuentran en distintas availability zones, por lo que el VGW es high availiable, por lo menos en la parte de AWS, en la parte del customer puede que no sea así, ver sección CGW.
