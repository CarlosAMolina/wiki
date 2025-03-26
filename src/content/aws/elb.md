# ELB

ELB = Elastic Load Balancer

## Introducción

Acepta conexiones de los clientes y las distribuye en las distintas partes de la infraestructura.

Permite que la infraestructura cambie (aumente, disminuya, repare recursos caídos) sin que el usuario lo note.

## Tipos

Hay 2 versiones de ELB:

- v1. Conviene evitarlo y utilizar v2, salvo en algunas excepciones. Los v1 no gestionan la layer OSI 7 de manera completa y tienen menos funcionalidad que v2, por ejemplo, 1 certificado SSL por Load Balancer lo que implica tener que utilizar múltiples Load Balancers.
- v2. Mas rápidos y baratos. Ofrecen funcionalidades como target groups y reglas para utilizar un mismo load balancer para distintas funcionalidades como cambiar el comportamiento según los clientes que lo usen.

Hay 3 tipos de ELB:

- CLB (Classic Load Balancer). Es v1.
- ALB (Application Load Balancer). Es v2. Soporta HTTP, HTTPS y WebSocket.
- NLB (Network Load Balancer). Es v2. Soporta TCP, TLS y UDP; por lo que se utiliza en aplicaciones que no emplean HTTP(S), por ejemplo servidores de email o ssh.

## Configuración

- Elegimos si utilizar solo IPv4 o IPv4 e IPv6.
- Que se ejecute en 2 o más AZ, en cada AZ elegimos 1 subred. Entonces se sitúa uno o más nodos de load balancer en una subred de cada AZ, los nodos escalan con la carga.
- Cada ELB se configura con un DNS A record que resuelve a los nodos del ELB.
- Modo:
  - Internet facing: los nodos tendrán IP pública y privada. Estos nodos pueden acceder a instancias EC2 públicas y privadas de AWS; por lo que no es necesario que las instancias EC2 sean públicas para acceder a ellas desde el exterior.
  - Internal: los nodos solo tendrán IPs privadas. Se utilizan para conectar distintos tiers de un aplicación, de modo que están desacopladas; no se conectan a algo específico sino que lo hacen al ELB y así la estructura de cada tier puede cambiar sin impacto.
- Sus nodos se configuran con listeners, que aceptan tráfico en un puerto y protocolo y se comunican con un puerto y protocolo específico.
- Necesitan que cada subred tenga mínimo 8 IPs libres, además hay que tener en cuenta las 5 IPs adicionales que se reservan en una subred. AWS recomienda utilizar mínimo una /27 para que el ELB pueda escalar (en un examen /28 también es respuesta válida si preguntan el mínimo).

## Distribución de las conexiones

A cada nodo le llega el mismo porcentaje de tráfico.

Con cross-zone LB se consigue que los nodos ELB puedan enviar tráfico a otras AZ.

De no tener cross-zone LB activado, los nodos solo pueden enviar tráfico en su AZ. Como todos los nodos reciben el mismo porcentaje de tráfico, si un nodo tiene menos instancias que otro, estas terminarán gestionando más carga; gracias a cross-zone LB evitamos esto.
