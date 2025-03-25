# ELB

ELB = Elastic Load Balancer

## Tipos

Hay 2 versiones de ELB:

- v1. Conviene evitarlo y utilizar v2, salvo en algunas excepciones. Los v1 no gestionan la layer OSI 7 de manera completa y tienen menos funcionalidad que v2, por ejemplo, 1 certificado SSL por Load Balancer lo que implica tener que utilizar múltiples Load Balancers.
- v2. Mas rápidos y baratos. Ofrecen funcionalidades como target groups y reglas para utilizar un mismo load balancer para distintas funcionalidades como cambiar el comportamiento según los clientes que lo usen.

Hay 3 tipos de ELB:

- CLB (Classic Load Balancer). Es v1.
- ALB (Application Load Balancer). Es v2. Soporta HTTP, HTTPS y WebSocket.
- NLB (Network Load Balancer). Es v2. Soporta TCP, TLS y UDP; por lo que se utiliza en aplicaciones que no emplean HTTP(S), por ejemplo servidores de email o ssh.
