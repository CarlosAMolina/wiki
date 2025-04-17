# Serverless

## Caracterísiticas

- Las aplicaciones son colecciones de funciones pequeñas y especializadas en realizar una acción.
- No se gestiona ninguno o casi ningún servidor. Esto es responsabilidad del proveedor del servicio.
- Los entornos son stateless y efímeros; al ejecutarse obtienen la información que necesitan y para almacenarla lo hacen en algo externo.
- Utilizan arquitectura event-driven.
- Se intentan emplear productos de terceros cuando sea posible (ejemplo para el login, así evitamos su desarrollo, mantenimiento y coste) y FaaS cuando se necesite realizar operaciones de cómputo.

## Event-Driven Arquitecture

Las diferentes partes de la aplicación se comunican por eventos; están desacopladas unas de otras.

Las partes de la aplicación solo se ejecutan cuando es requerido.

La aplicación la forman:

- Producers: generan eventos.
- Consumers: reciben eventos y ejecutan un acción.
- Hay elementos que son producers y consumers.
- Buenas arquitecturas event-driven, en lugar de conectar todos los elementos entre sí, utilizan un event router para gestionar los eventos. Los eventos viajan por un event bus.

Si la arquitectura es de tipo serverless, solo se consumen recursos cuando se gestionan eventos.

## Alternativas open source

- [Apache Kafka](https://kafka.apache.org/)
- [Artículo sobre cambio de arquitectura monolítiva a AWS y finalmente a Celery y Kubernetes](https://itnext.io/back-to-the-future-the-future-might-not-just-be-serverless-after-all-b8165d6e84c2)

