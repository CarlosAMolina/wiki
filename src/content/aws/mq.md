# MQ

## Introducción

Es una mezcla entre SNS y SQS. Es un servicio basado en Apache ActiveMQ, lo que lo hace compatible con JMS API y protocolos como AMQP, MQTT, OpenWire y STOMP.

No es un servicio público, funciona en una VPC.

La diferencia entre estos servicios es que SNS y SQS utiliza tecnología de AWS, pero si nuestra aplicación emplea sistemas de topic y colas basados en los protocolos y api anteriores, no será compatible con SNS y SQS y la aplicación debe modificarse para migrarla a AWS; gracias a MQ podemos migrar las partes de la aplicación que consumen los mensajes a AWS y mantener nuestro sistema on-premise que genera las colas.

MQ no tiene integración nativa con otros servicios de AWS (ejemplo logging, permisos, cifrado, etc.). Por lo que se recomienda usar SNS y SQS a menos que necesitemos migrar un sistema a AWS sin casi modificar la aplicación y que necesite usar los protocolos y api antes descritos.

## Arquitectura en AWS

Para acceder a él, nuestro software on premise que genera los mensajes debe usar una VPN o AWS Direct Connect.

Los datos los almacena en EFS y los Amazon MQ Broker pueden acceder a ellos.

Las aplicaciones migradas a AWS se comunican con los Amazon MQ Broker mediante JMS, AMQP, MQTT, etc.
