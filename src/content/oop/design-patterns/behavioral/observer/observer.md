## Motivación

Permite conocer el cambio en un sistema y de esta manera podremos notificar que ha producido un cambio, revisar si es un cambio permitido, etc.

Nos dará la oportunidad de escuchar ciertos tipos de eventos (un evento es algo que ha ocurrido) y también poder dejar de escucharlos.


En este patrón intervienen dos conceptos:

- Observable: la entidad que genera los eventos a los que subscribirnos.
- Observer: es un objeto que escucha, es informado de los eventos que ocurren en el sistema.

## Ejemplo

### Evento

Indicamos funciones que ejecutar cuando un evento ocurre.

Estas funciones las guardamos en una lista, a la que iremos añadiendo o quitando funciones según queramos suscribirnos o dejar de estarlo.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/observer/events.py>.

### Property Observer

Detecta cuando una propiedad cambia.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/observer/property_observers.py>.

### Property Dependencies

Como el ejemplo de Property Observer pero utilizado en el caso de que una propiedad dependa de otra.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/observer/property_dependencies.py>.


