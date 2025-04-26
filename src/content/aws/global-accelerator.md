# AWS Global Accelerator

## Introducción

Este producto mejora el performance de la conexión entre los usuarios y nuestra infraestructura de AWS.

Utiliza global accelerator edge locations, cada uno de ellos tienen varias direcciones IP anycast.

Una IP anycast significa que distintos dispositivos pueden tener la misma IP; esto es diferente a las IP unicast que usamos normalmente, donde cada dispositivo tiene una IP única. En las IP anycast, el tráfico va a la IP más cercana.

Cuando un usuario solicita algo en AWS, su tráfico inicialmente va en el internet público, llega al edge location global accelerator y desde aquí, viaja por el backbone network de AWS, el cual utiliza menos saltos para llegar al destino y ofrece mejor rendimiento. Al ser la red gestionada por AWS, en caso de no tener un buen rendimiento, podemos contactar con ellos para que lo solucionen.

Global accelerator también puede determinar qué parte de la infraestructura de AWS está mas cerca del global accelerator edge location y dirigir el tráfico ahí.

## Diferencias con CF

- CF acerca a los usuarios una caché del contenido HTTP.
- AWS global accelerator acerca la red de AWS a los usuarios, trabaja con los protocolos TCP y UDP, no realiza cacheo.
