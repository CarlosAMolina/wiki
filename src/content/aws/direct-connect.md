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

Debemos configurar interfaces virtuales sobre esta conexión física, llamadas vifs. Hay 3 tipos:

- Transit vifs.
- Public vifs. Para acceder a los servicios públicos de AWS.
- Vifs privadas. Para acceder a VPCs en AWS y así a servicios privados de AWS.

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
