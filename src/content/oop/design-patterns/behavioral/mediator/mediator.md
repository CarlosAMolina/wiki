## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Chat room](#chat-room)
  - [Mediator con eventos](#mediator-con-eventos)

## Motivación

Facilitar comunicación entre componentes sin que estos tengan que preocuparse unos de otros o tener una referencia directa entre sí.

Esto ayuda por ejemplo en casos en que objetos dejan de estar activos (ejemplo, un chat donde la gente se conecta y desconecta) ya que sería muy difícil mantener la relación.

Para utilizarlo:

- Creamos el mediator y cada objeto que lo necesite hará referencia a el, por ejemplo en un atributo.
- El mediator tiene funciones que los componentes pueden llamar.
- Los componentes tienen funciones que el mediator puede llamar.

## Ejemplo

### Chat room

Clase `ChatRoom` es el mediator, clases `Person` no tienen relación unas con otras.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/mediator/mediator.py>.

### Mediator con eventos

Es un ejemplo de mediator con eventos.

Creamos la clase `Game` que se le pasará a `Player` y `Coach`. Usaremos eventos para detectar que algo ocurre.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/mediator/mediator_with_events.py>.



