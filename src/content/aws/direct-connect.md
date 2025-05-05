# AWS Direct Connect

## Introducción

Direct connect nos habilita un puerto para conectarnos desde nuestra red a la localización donde se encuentra AWS Direct Connect. Mediante un cable de fibra óptica conectamos nuestro router con el router de la parte AWS Direct Connect.

Hemos de imaginar 3 localizaciones:

- Nuestra red.
- Una localización de AWS Direct Connect, es como un data center. En ella está:
  - AWS Direct Connect Cage.
  - Customer Cage o Comms Partner Cage. Si el cliente no tiene un cage, lo alquila a un Partner que permite utilizar un puerto de su router desde la red del cliente.
- Una región de AWS, esta región está conectada con una o más Direct Connect locations.

El router de AWS Direct Connect Cage está conectado con el router del customer cage.

Debemos configurar interfaces virtuales sobre esta conexión física, llamadas vifs. Hay 3 tipos:

- Transit vifs.
- Public vifs. Para acceder a los servicios públicos de AWS.
- Vifs privadas. Para acceder a VPCs en AWS y así a servicios privados de AWS.

Gracias a estas interfaces, nos conectamos a AWS sin pasar por los ISP. No sirve para acceder a internet.

Ofrece baja resilencia y altas velocidades.

Puede tardar entre semanas o meses tener la conexión de nuestro router a AWS Direct Connect.

No ofrece resilencia, de querer evitar problemas con el cable de fibra óptica, debemos provisionar más conexiones.

Tenemos 3 velocidades disponibles: 1, 10 o 100 Gbps.

## Coste

Por hora que está habilitada la conexión y por tráfico de salida.
