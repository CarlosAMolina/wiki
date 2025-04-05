# Serverless

## Event-Driven Arquitecture

Las diferentes partes de la aplicación se comunican por eventos; están desacopladas unas de otras.

La aplicación la forman:

- Producers: generan eventos.
- Consumers: reciben eventos y ejecutan un acción.
- Hay elementos que son producers y consumers.
- Buenas arquitecturas event-driven, en lugar de conectar todos los elementos entre sí, utilizan un event router para gestionar los eventos. Los eventos viajan por un event bus.

Si la arquitectura es de tipo serverless, solo se consumen recursos cuando se gestionan eventos.

## Alternativas open source

- [Apache Kafka](https://kafka.apache.org/)
- [Artículo sobre cambio de arquitectura monolítiva a AWS y finalmente a Celery y Kubernetes](https://itnext.io/back-to-the-future-the-future-might-not-just-be-serverless-after-all-b8165d6e84c2)

